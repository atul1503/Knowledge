##How to create a deep copy of a complex object

```
const complexObject={
    Home:{
        homeposts:[],
        load_prev: false,
        load_next: false
    },
    Comment_page:{
        reply_text:"",
        child_posts: [],
        post: {}
    },    
    username:"",
    login_page:{
        username:"",
        password:""
    }
}

const copy=JSON.parse(JSON.stringify(complexObject))
```
**But keep in mind that the object should not have functions or methods and should be serializable to json.**
This you can use almost always in redux reducers to change nested parts of the state store.


## How to stop parent events from getting triggered?

By default, parent events are also triggered if child events are triggered if they have registered the same event. For example if you added click event listeners on both parent and child then if you click on child, first child event will be triggered and then parent event. But, you can avoid parent event getting triggered if you put 
```
event.stopPropagation()
```
in child event handler.
