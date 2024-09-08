# How to integrate react with react router to django. 

### In this we are using django as data provider for the react-app and react router does the routing instead of django.

#### First step: 
Build your react app with ```npm run build``` But for development stage, you can run your react app with its devlopment server. But once you are sure that all react development is done and you are going to deploy then only do the build. Surely, you must be refering to the backend server with its host and port, make sure that the host and port are imported from a sperate file when doing `fetch()` calls. This allows you to change url anytime you want.

## How to tell django  where to serve templates from?
You see this : 
```TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR,"build")],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
This configuration tells django where your index.html file will be. Django calls html file as templates.  All of this will already be there in django settings.py file. Just make sure that in the dirs field , specify the folder in which your index.html file will be. BASE_DIR is a directory which is by default set to the directory which contains manage.py file. `build` is the folder which is created by react on `npm run build`. So you just have to copy paste that build folder to the place where you see manage.py.

## How to tell django where and when to serve static content?

Now, in the settings.py , maybe just after the TEMPLATES variable, mention this also:

```STATIC_URL = 'static/'

STATICFILES_DIRS=[
    os.path.join(BASE_DIR, 'build/static'),
]
```

The static url variable tells django the url which will be used to directly access static things. If the user requests this path then only serve static content.
`REMEMBER: static files mean js, css not html. html is called template in django`

static file dirs variable tells django where to look for static files. Inside our build folder that react generates, it provides css and js, so that is the reason why we are including it here. You can include more paths also.


## How to access paths defined by react router in django?

By default what happens is that the django sends a not found response when the user requests a path that is not defined by our django project. But we can use this to our advantage to give chance to react router to serve those paths. So in the urls.py have this:

```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('hello/',HelloAPI.as_view()),
    re_path(r"^(?:.*)/?$",react_app)
]
```
urlpatterns will already be defined from the start. But we have to make sure that all django api calls that our react app has to do, gets to django rest, but to serve the frontend, we define an regex path at the very end in url patterns, which matches anything else. So the order of search here will be , django will first try to match admin/ , if not matched then it will match hello/ , if not matched it will send all requests to react_app view (here react_app is a view which is imported). But this also means that any path not defined by react router, will still reach react router. This regex pattern is very important, `r"^(?:.*)/?$"`, it matches everything.

## But you might ask how is react_app serving the react app contents?

in the views.py file , we define a view like this:

```
def react_app(request):
    return render(request,'index.html',{})
```
So here we are serving index.html as a template to all those paths that didnot match with admin/ or hello/. We know that this index.html is our gateway to the react world that we have setup in our django project. This index.html is the same index.html which was built by react on the build process. It also helps us to get to react router.
