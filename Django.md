Notes:
1. Settings.py path is determined first from environment variable then if you have defined in manage.py file the path of settings.py file. Even in manage.py also you use os.envrion only to set settings.py module path.
2. You can use different settings.py for different environements.
3. If any settings is missing from your settings.py file it will be fetched from the default settings.py that comes with django or if you are using an external library then the library might have provided the default value, it will take that in worst case.
4. Middlewares are packages which are executed in the order they are defined in settings.py. Requests are processesed from top to bottom and then it reaches its view. For response it is processes bottom to top of middlesware after sent by view.
5. Each middleware class has __init__ method and __call__ method. Init gets called on propject startup and call gets called on each request. In init method you are provided with the callback function that you will call to call either the next middleware or the view directly if there is no middleware next. this callback returns the response. this reponse is from the view itself although it may also get modified by other middlewares along the way.
6. Signals objects are objects that have `send()` function which is used to send these signals. There is a receiver that you can register that will be configured to receive these signals. Create a `Signal` object in sender app and then import it in receiver app apps.py and then in the read() function of the receiver object import it. Now in the receiver app, declare a function with `@receiver(signal_name)` this function will be triggered everytime a singal is sent by the sender app. Sender can pass any number of args in the `send()` call. First pass the class object of the sender class and then pass any other objects that need to be sent as keyword arg. The receiver function which got annoated will have access to those args as its own argumment.
7. Serializers in drf helps to serialize(convert complex python objects like class instances or model instances or queryset instances to python native datatypes like list,dict) or desrialize the complex python objects so that they can be easily converted to json or other format for storage or transportation. You do that with decalring a class with `serializers.Serializer` subclass and declaring fields directly inside the class as static fields. These fields are looked up when serialling or deserializing data.
8. For realtionssips, in django, define relationship in just one model with `OneToOneField` or `ManyToManyField` and `ForeignKey` fields with on_delete arg set to either models.CASCADE(to delete the connected entity on the deletion of this object) or models.PROTECT(to not delete). Use `related_name` to set set what name does the other entity refer to use this relationship.
9. Serializers check for individula field validations using the fiedls that you have decalred and you can use your own field validations using `validate_fieldName` and you can also implement your own validate(data) where you can do object level validations. This all happens with serializer.is_valid(). If true then you can either save it in case of model serilaaizer or use it around.
10. When creating a serilaizer use `MySerializer(data=request.data)` to serilaize request.data you can also use request.query_params.
11. For model serilaizer, to update provide both the model instance and the data= arg which includes the data.
12. `Model.objects.get()` returns a single model instance but `filter` returns a query set which can be used for loop like this: `for result in query_set:`
13. `django.db.transaction.atomic` is both a decorator and a function that can be used with `with` to perform atomic trnasactions which means if something fails everything will be rolled back. This decorator can be used with a function for bulk updates etc.
14. get(): get resources, post(): new resources, put(): update existing, patch(): partial updates of exisiting resources, delete(): to delete resources.
15. These above methods are http methods can be used as APiView functions.
16. redirect() is method from django.shortcuts that you import to redirect to url. render() is used to render templates with context= args to provide context variable.
17. ```{% for item in items %} <div>{% item %}</div>{% endfor %}```
18.  Model.objects.update() allows to update records. update() takes similar args as filter. Use Q() for complex logic like Q() | Q() or ~Q(), & Q(). fieldName__contains("") to check for string values. fieldName__icontains("some toher") for case insensiviity.fieldName__lte for less than equal to. Others are lte, gte, gt, lt,
19.  You can use `queryset.query()` to get the query which got this queryset.
20.  Permission classes can be implemetned by subclassing `BasePermission` class where you implement `has_pemission(self,request,view)` to check return true or false. request.user get the associated user object. request.user.is_authenticated check if user is authenticated or not and not anonymous.
21.  To create roles kind of things. Create sperate permission class for each role where you decalre the role as as string attribute and in the has_pemissions() method you just check if the user has your role string in his role or not.
22. `permission_classes=[]` is defined in the APIView class as class attribute.
23. Mostly user has all the features but for others and roles by extending `AbstractUser` and include the roles.
24. `IsAuthenticated()` permission class has blocks anonymous user.
25. Json web token based there is another middleware that you have which provides its own token generation and token refresh views which you can associate with any path.
26. in the urlpatterns if you have a path defined then in the path() you can provide name= arg to refer to it whenever and wherver you want with `reverse(name)` instead of using the full path.
27. Session middleware sets and retrives sessions from the session backedn and attaaches it to every request. The authnetication middleware associated the user id with session id that too in the session backend(session model table). There is _auth_user_id field to associate user id. The authenticatin middle ware then sets a user attribute in request object. request.user.is_authenticated is used to check if user is anonymous.
28. SECURTIY_KEY in django is used to sign the cookie that is stored in the browser. session id cookie store the session id in the clietn browser.
29. csrf works on unsafe methods(post,put,delete).
30. if frontend and backend are in different origin then enable cors and include those frontend domains that can access your backend api and use token based auth like jwt and disable csrf because you are NOT using sessions and will not be a problem.
31. To include cors, use cors headers library and include cors middleware and coresheaders app.
32. to disable csrf remove csrf middleware.
33. For https:
    set secure ssl direct to true so that even if browser sends http it will converted to https.
    set secure proxy ssl header to (HTTP_X_FORWWARDED_PROTO,https) which means proxy forwarded https to http to django so django can relax it is https request only.
    secure hsts second to 3600 meaning for 1 hr all requests should be sent as https only
    secure hsts include subdomain to true meaning all subdomains will be https only.

34. Famous http codes:
        200: ok succes
        201: created
        100: continue
        101: protocol changed
        301: moved permanently
        302: moved temporarily
        400: bad request
        401: unauthneticated
        403: unauthorized
        404: not found
        500: internal server error
        503: service unavailbale
        504: server timeout
