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
