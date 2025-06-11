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

35. Solid: S: single reposnitbikity classes. O: open for expansion closed for modification, L: subclasses can take place of super classes. I: interfaces incldes methods that are needed. D: depdnecy inversion means that you create abstract classes that others implement to provide functionality rather than dependeing directly on an entity.
36. For post requests, you get file uploads from browser as: request.FILES['files'], this is an UploadFile object. You get bytes of it in chunks using `for chunk in .chunks()`. You can write this down in a file opened with `wb+` mode. 
37. To send the file you use `FileResponse` object. It expects the file object opened with `rb` mode. Now set the headers `Content-Type` and `Content-Disposition`. Content-Type is `application\octet-stream` for this type and content disposition is to set whether file should download directly as attachment or displayed inline. Content Dispoisiton is set as `attachment;filename=`.

38. `io` library provides varios kinds of streams. `io.BytesIO` you provide the `io.StringIO` uses streams to write things down. Use `BufferedReader` and `BufferedWriter` with file as `with` to write or read bytes. Buffered writer contrcutor accepts file object. 

# Django REST Framework (DRF) Serializers - Complete Guide

## What are Serializers?

39. **Serializers** convert complex Python objects (like Django model instances) into Python native data types (dict, list, etc.) that can then be easily rendered into JSON, XML or other content types. They also work in reverse - converting JSON/form data back into Python objects.

    ```python
    # Without serializers - manual and error-prone
    def user_to_json(user):
        return {
            'id': user.id,
            'username': user.username,
            'email': user.email,
            'date_joined': user.date_joined.isoformat()
        }
    
    # With serializers - automatic and robust
    class UserSerializer(serializers.ModelSerializer):
        class Meta:
            model = User
            fields = ['id', 'username', 'email', 'date_joined']
    ```

## Basic Serializer Types

40. **Serializer vs ModelSerializer** - Use `Serializer` for custom data structures, `ModelSerializer` for Django models. ModelSerializer automatically creates fields based on your model.

    ```python
    from rest_framework import serializers
    from .models import User, Post
    
    # Basic Serializer - define fields manually
    class CustomDataSerializer(serializers.Serializer):
        name = serializers.CharField(max_length=100)
        age = serializers.IntegerField()
        is_active = serializers.BooleanField(default=True)
        created_at = serializers.DateTimeField()
    
    # ModelSerializer - fields auto-generated from model
    class UserSerializer(serializers.ModelSerializer):
        class Meta:
            model = User                    # Which model to serialize
            fields = '__all__'              # Include all model fields
            # OR specify specific fields
            # fields = ['id', 'username', 'email', 'first_name']
            # OR exclude certain fields  
            # exclude = ['password', 'last_login']
    ```

## Using Serializers - Serialization (Object to JSON)

41. **Serializing single objects** - Pass the model instance to the serializer, then access `.data` to get the serialized dictionary.

    ```python
    # In your views.py
    from django.http import JsonResponse
    from .models import User
    from .serializers import UserSerializer
    
    def get_user(request, user_id):
        user = User.objects.get(id=user_id)         # Get the model instance
        serializer = UserSerializer(user)          # Create serializer with instance
        return JsonResponse(serializer.data)       # .data contains the serialized dict
        
    # serializer.data will be something like:
    # {
    #     'id': 1, 
    #     'username': 'john_doe', 
    #     'email': 'john@example.com',
    #     'date_joined': '2023-01-15T10:30:00Z'
    # }
    ```

42. **Serializing multiple objects (QuerySets)** - Use `many=True` parameter when serializing lists or QuerySets.

    ```python
    def get_all_users(request):
        users = User.objects.all()                  # QuerySet of multiple users
        serializer = UserSerializer(users, many=True)  # many=True for multiple objects
        return JsonResponse(serializer.data, safe=False)  # safe=False for lists
        
    # serializer.data will be a list:
    # [
    #     {'id': 1, 'username': 'john_doe', 'email': 'john@example.com'},
    #     {'id': 2, 'username': 'jane_doe', 'email': 'jane@example.com'},
    # ]
    ```

## Using Serializers - Deserialization (JSON to Object)

43. **Deserializing data** - Pass data via `data=` parameter, check if valid with `is_valid()`, then access `validated_data` or `save()`.

    ```python
    from rest_framework.decorators import api_view
    from rest_framework.response import Response
    from rest_framework import status
    
    @api_view(['POST'])
    def create_user(request):
        # request.data contains the JSON data sent by client
        serializer = UserSerializer(data=request.data)  # Pass data to deserialize
        
        if serializer.is_valid():                        # Validate the data
            user = serializer.save()                     # Create and save the model instance
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        else:
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    ```

44. **Updating existing objects** - Pass both the instance and data to update an existing model.

    ```python
    @api_view(['PUT'])
    def update_user(request, user_id):
        user = User.objects.get(id=user_id)              # Get existing instance
        serializer = UserSerializer(user, data=request.data)  # Pass both instance and data
        
        if serializer.is_valid():
            serializer.save()                            # Updates the existing instance
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    ```

45. **Partial updates** - Use `partial=True` for PATCH requests where you only want to update some fields.

    ```python
    @api_view(['PATCH'])
    def partial_update_user(request, user_id):
        user = User.objects.get(id=user_id)
        serializer = UserSerializer(user, data=request.data, partial=True)  # partial=True allows incomplete data
        
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    ```

## Field Customization

46. **Read-only and write-only fields** - Control which fields are included during serialization vs deserialization.

    ```python
    class UserSerializer(serializers.ModelSerializer):
        password = serializers.CharField(write_only=True)    # Only used when creating/updating
        full_name = serializers.CharField(read_only=True)    # Only included in output
        
        class Meta:
            model = User
            fields = ['id', 'username', 'password', 'email', 'full_name']
            extra_kwargs = {
                'password': {'write_only': True},            # Alternative way to set write_only
            }
    ```

47. **Custom fields and SerializerMethodField** - Add computed fields that don't exist in your model.

    ```python
    class UserSerializer(serializers.ModelSerializer):
        # SerializerMethodField calls get_<field_name> method
        full_name = serializers.SerializerMethodField()
        posts_count = serializers.SerializerMethodField()
        
        class Meta:
            model = User
            fields = ['id', 'username', 'email', 'full_name', 'posts_count']
        
        def get_full_name(self, obj):                        # Method for full_name field
            return f"{obj.first_name} {obj.last_name}"
            
        def get_posts_count(self, obj):                      # Method for posts_count field
            return obj.posts.count()                         # Access related objects
    ```

## Validation

48. **Field-level validation** - Create `validate_<field_name>` methods to validate individual fields.

    ```python
    class UserSerializer(serializers.ModelSerializer):
        class Meta:
            model = User
            fields = ['username', 'email', 'age']
        
        def validate_username(self, value):                  # Validates username field
            if len(value) < 3:
                raise serializers.ValidationError("Username must be at least 3 characters")
            if User.objects.filter(username=value).exists():
                raise serializers.ValidationError("Username already exists")
            return value                                     # Must return the value if valid
        
        def validate_age(self, value):                       # Validates age field
            if value < 18:
                raise serializers.ValidationError("Must be 18 or older")
            return value
    ```

49. **Object-level validation** - Use `validate()` method to validate multiple fields together.

    ```python
    class EventSerializer(serializers.ModelSerializer):
        class Meta:
            model = Event
            fields = ['name', 'start_date', 'end_date', 'max_attendees']
        
        def validate(self, data):                            # Validates entire object
            start_date = data.get('start_date')
            end_date = data.get('end_date')
            
            if start_date and end_date and start_date >= end_date:
                raise serializers.ValidationError("End date must be after start date")
                
            max_attendees = data.get('max_attendees')
            if max_attendees and max_attendees < 1:
                raise serializers.ValidationError("Must allow at least 1 attendee")
                
            return data                                      # Must return the validated data
    ```

## Handling Relationships

50. **Foreign Key relationships** - By default, foreign keys are serialized as their primary key. You can customize this behavior.

    ```python
    # Models
    class Author(models.Model):
        name = models.CharField(max_length=100)
        email = models.CharField(max_length=100)
    
    class Book(models.Model):
        title = models.CharField(max_length=200)
        author = models.ForeignKey(Author, on_delete=models.CASCADE)
        published_date = models.DateField()
    
    # Default behavior - shows author ID only
    class BookSerializer(serializers.ModelSerializer):
        class Meta:
            model = Book
            fields = ['id', 'title', 'author', 'published_date']
        # Output: {'id': 1, 'title': 'Django Book', 'author': 5, 'published_date': '2023-01-01'}
    ```

51. **Nested serializers** - Include full related object data by using nested serializers.

    ```python
    class AuthorSerializer(serializers.ModelSerializer):
        class Meta:
            model = Author
            fields = ['id', 'name', 'email']
    
    class BookSerializer(serializers.ModelSerializer):
        author = AuthorSerializer(read_only=True)           # Nest the full author object
        
        class Meta:
            model = Book
            fields = ['id', 'title', 'author', 'published_date']
        
        # Output: {
        #     'id': 1, 
        #     'title': 'Django Book', 
        #     'author': {'id': 5, 'name': 'John Smith', 'email': 'john@example.com'},
        #     'published_date': '2023-01-01'
        # }
    ```

52. **Reverse relationships (One-to-Many)** - Access related objects from the "one" side of the relationship.

    ```python
    class AuthorSerializer(serializers.ModelSerializer):
        books = BookSerializer(many=True, read_only=True)   # 'books' is the related_name
        books_count = serializers.SerializerMethodField()
        
        class Meta:
            model = Author
            fields = ['id', 'name', 'email', 'books', 'books_count']
        
        def get_books_count(self, obj):
            return obj.books.count()
        
        # Output: {
        #     'id': 5,
        #     'name': 'John Smith',
        #     'email': 'john@example.com',
        #     'books': [
        #         {'id': 1, 'title': 'Django Book', 'published_date': '2023-01-01'},
        #         {'id': 2, 'title': 'Python Guide', 'published_date': '2023-06-01'}
        #     ],
        #     'books_count': 2
        # }
    ```

53. **Many-to-Many relationships** - Handle M2M relationships with proper serialization.

    ```python
    # Models
    class Tag(models.Model):
        name = models.CharField(max_length=50)
    
    class Article(models.Model):
        title = models.CharField(max_length=200)
        tags = models.ManyToManyField(Tag, blank=True)
    
    # Serializers
    class TagSerializer(serializers.ModelSerializer):
        class Meta:
            model = Tag
            fields = ['id', 'name']
    
    class ArticleSerializer(serializers.ModelSerializer):
        tags = TagSerializer(many=True, read_only=True)     # Full tag objects
        tag_names = serializers.SerializerMethodField()    # Just tag names
        
        class Meta:
            model = Article
            fields = ['id', 'title', 'tags', 'tag_names']
        
        def get_tag_names(self, obj):
            return [tag.name for tag in obj.tags.all()]
    ```

## Advanced Serializer Patterns

54. **Different serializers for different actions** - Use different serializers for list vs detail views, or create vs update.

    ```python
    # List view - minimal data
    class UserListSerializer(serializers.ModelSerializer):
        class Meta:
            model = User
            fields = ['id', 'username', 'email']
    
    # Detail view - full data  
    class UserDetailSerializer(serializers.ModelSerializer):
        posts = PostSerializer(many=True, read_only=True)
        full_name = serializers.SerializerMethodField()
        
        class Meta:
            model = User
            fields = ['id', 'username', 'email', 'first_name', 'last_name', 'full_name', 'posts']
        
        def get_full_name(self, obj):
            return f"{obj.first_name} {obj.last_name}"
    
    # In views
    class UserViewSet(viewsets.ModelViewSet):
        queryset = User.objects.all()
        
        def get_serializer_class(self):
            if self.action == 'list':
                return UserListSerializer
            return UserDetailSerializer
    ```

55. **Dynamic fields** - Include/exclude fields based on query parameters or user permissions.

    ```python
    class DynamicFieldsModelSerializer(serializers.ModelSerializer):
        def __init__(self, *args, **kwargs):
            # Don't pass the 'fields' arg up to the superclass
            fields = kwargs.pop('fields', None)
            
            super().__init__(*args, **kwargs)
            
            if fields is not None:
                # Drop any fields that are not specified in the `fields` argument
                allowed = set(fields)
                existing = set(self.fields)
                for field_name in existing - allowed:
                    self.fields.pop(field_name)
    
    class UserSerializer(DynamicFieldsModelSerializer):
        class Meta:
            model = User
            fields = ['id', 'username', 'email', 'first_name', 'last_name']
    
    # Usage in views
    def get_user(request, user_id):
        user = User.objects.get(id=user_id)
        fields = request.GET.get('fields')                  # ?fields=id,username,email
        if fields:
            fields = fields.split(',')
            serializer = UserSerializer(user, fields=fields)
        else:
            serializer = UserSerializer(user)
        return Response(serializer.data)
    ```

56. **Custom create() and update() methods** - Override these methods for complex object creation/updating logic.

    ```python
    class UserSerializer(serializers.ModelSerializer):
        password = serializers.CharField(write_only=True)
        confirm_password = serializers.CharField(write_only=True)
        
        class Meta:
            model = User
            fields = ['username', 'email', 'password', 'confirm_password']
        
        def validate(self, data):
            if data['password'] != data['confirm_password']:
                raise serializers.ValidationError("Passwords don't match")
            return data
        
        def create(self, validated_data):
            # Remove confirm_password as it's not a model field
            validated_data.pop('confirm_password')
            
            # Hash the password before saving
            user = User(
                username=validated_data['username'],
                email=validated_data['email']
            )
            user.set_password(validated_data['password'])    # This hashes the password
            user.save()
            return user
        
        def update(self, instance, validated_data):
            # Update regular fields
            instance.username = validated_data.get('username', instance.username)
            instance.email = validated_data.get('email', instance.email)
            
            # Handle password separately if provided
            password = validated_data.get('password')
            if password:
                instance.set_password(password)
                
            instance.save()
            return instance
    ```

## Best Practices

57. **Optimize database queries** - Use `select_related()` and `prefetch_related()` to avoid N+1 query problems when using nested serializers.

    ```python
    # In your views - optimize queries before serializing
    def get_books(request):
        # Bad - causes N+1 queries (one query per book to get author)
        books = Book.objects.all()
        
        # Good - single query with JOIN to get authors too
        books = Book.objects.select_related('author').all()
        
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)
    
    def get_authors(request):
        # For reverse relationships, use prefetch_related
        authors = Author.objects.prefetch_related('books').all()
        serializer = AuthorSerializer(authors, many=True)
        return Response(serializer.data)
    ```

58. **Use appropriate field types** - Choose the right serializer field for your data type for proper validation and serialization.

    ```python
    class ProductSerializer(serializers.ModelSerializer):
        # Specific field types provide better validation
        price = serializers.DecimalField(max_digits=10, decimal_places=2)
        created_at = serializers.DateTimeField(format='%Y-%m-%d %H:%M:%S')
        is_featured = serializers.BooleanField()
        category_name = serializers.CharField(source='category.name', read_only=True)
        
        class Meta:
            model = Product
            fields = ['id', 'name', 'price', 'is_featured', 'created_at', 'category_name']
    ```
