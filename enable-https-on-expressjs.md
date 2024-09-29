**this will create self signed certificates and keys but in production you will have to get the certs and keys from CA**

1. `sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./selfsigned.key -out selfsigned.crt`. This will create key file and crt file with names selfsigned.key and selfsigned.cert
2. in your express js main js file, add this: 
```
const fs=require('fs');
var key=fs.readFileSync('selfsigned.key')
var cert=fs.readFileSync('selfsigned.crt')

var options = {
    key: key,
    cert: cert
  };

var server = https.createServer(options, app);

const port=8000

server.listen(port, () => {
    console.log("server starting on port : " + port)
  });
```

here the `app` object is the object used to create you express js app with `express()`.
