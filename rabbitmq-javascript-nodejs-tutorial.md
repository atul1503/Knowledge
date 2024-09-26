## Installation on Mac

1. `brew install rabbitmq`
2. `brew services start rabbitmq`
3. Check the rabbitmq ui at `http://localhost:15672`, if its running, ui will be shown. **username and password both are `guest`**
4. `npm install amqplib`


## Sender code
```
const amqp = require('amqplib');

const args=process.argv.slice(2);

async function send() {
    const queue = 'hello';
    const msg = JSON.stringify({
        op: "add",
        a: args[0],
        b: args[1]
    });

    try {
        const connection = await amqp.connect('amqp://127.0.0.1:');
        const channel = await connection.createChannel();
        await channel.assertQueue(queue, { durable: false });
        channel.sendToQueue(queue, Buffer.from(msg));
        console.log(" [x] Sent %s", msg);
        await channel.close();
        await connection.close();
    } catch (error) {
        console.error(error+" is here");
    }
}

send();
```

Here,
1. We are first creating a connection using the amqp protocol url which `amqp:hostname:port`. But here host is 127.0.0.1 and port by default is 5672 so no need to specify port.
2. We are then creating a channel from the connection as its required and good practice.
3. We use this channel object to send messages.
4. using the `assertQueue` we are checking if queue already exists or not , if not then it gets created. Then we see an object with `durable` as `false` which means that messages from queue will be deleted if rabbitmq gets down. In production we have to keep this as `true`.
5. `sendToQueue` method is used to send message to queue. But the message that we send should be in binary. To convert it to binary we use `Buffer.from()` and pass our message as string to the `from()`. If you are sending an object, serialize it to a json string using `JSON.stringify(msg)` and then pass it to `Buffer.from()`.
6. After message is sent, the sender can do whatever it wants or can even exit. But before exiting close first the channel and then the connection.

**Here async await is used only because to set up a connection and channel to the amqp server, we don't want to use promise and form a complex nested code to send a message. Its just a preference thing and not a necessity. You can use promise also.**
   

## Consumer code
```
const amqp = require('amqplib');

async function receive() {
    const queue = 'hello';

    try {
        const connection = await amqp.connect('amqp://127.0.0.1');
        const channel = await connection.createChannel();
        await channel.assertQueue(queue, { durable: false });
        
        console.log(" [*] Waiting for messages in %s. To exit press CTRL+C", queue);
        
        channel.consume(queue, (msg) => {
            var msg=JSON.parse(msg.content.toString());
           if(msg.op==="add"){
            console.log("The sum of %s and %s is %s",msg.a,msg.b,Number(msg.a)+Number(msg.b));
           }
        }, { noAck: true });
    } catch (error) {
        console.error(error);
    }
}

receive();
```

Here, everything is same as sender code, but 
1. with `channel.consume()` we are passing the `queue` name  as string and the callback function that will be executed when a message is recieved from this queue.
2. the `consume()` call blocks the terminal as the consumer will continue to listen for messages from that queue. Here we have used async await only to wait for the connection and channel to be established with the rabbitmq server and not because we want the consumer to wait infinitely to process messages.

**the `consume()` blocks the main thread on which the consumer will be listening, so if you use ctrl+c then the whole process will stop and the consumer will stop listening and the connection and channel with the rabbitmq server will be destroyed.**

In a production env, you want the messages to persist on disk.
Use this paradigm only when the web server can't immediately process a request. The web server should then pass the request as a message to a queue in rabbitmq and any consumer who knows how to execute message from that queue will pick it up and execute. The web server should send the response saying that the request is recieved and is being processed. The user client code can then poll the status to check if completed or not.

If the `noAck` is `false` then the consumer has to manually acknowledge the broker that it processed the message like this `channel.ack(msg)`. There is a `Delivery Acknowledgment Timeout` which allows rabbitmq to wait for this much time for the consumer to which it has sent the message to process it. Within this time if the consumer fails to acknowledge the message then rabbitmq will not delete the message from queue and will send it to another consumer if it exists.
