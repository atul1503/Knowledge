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

# Django Caching - Complete Guide

## Cache Configuration

59. **Cache backends in Django** - Django supports multiple cache backends. Configure them in your `settings.py` file. The most common are Redis, Memcached, Database, and Local Memory.

    ```python
    # settings.py - Different cache backend configurations
    
    # REDIS CACHE (most popular for production)
    CACHES = {
        'default': {
            'BACKEND': 'django_redis.cache.RedisCache',
            'LOCATION': 'redis://127.0.0.1:6379/1',        # Redis server location
            'OPTIONS': {
                'CLIENT_CLASS': 'django_redis.client.DefaultClient',
                'PASSWORD': 'your-redis-password',          # If Redis has auth
            },
            'TIMEOUT': 300,                                 # Default timeout in seconds (5 minutes)
            'KEY_PREFIX': 'myapp',                         # Prefix all cache keys
        }
    }
    
    # MEMCACHED CACHE
    CACHES = {
        'default': {
            'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
            'LOCATION': '127.0.0.1:11211',
        }
    }
    
    # DATABASE CACHE (slowest but always available)
    CACHES = {
        'default': {
            'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
            'LOCATION': 'my_cache_table',                   # Table name for cache
        }
    }
    
    # LOCAL MEMORY CACHE (for development only)
    CACHES = {
        'default': {
            'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
            'LOCATION': 'unique-snowflake',
        }
    }
    ```

60. **Installing Redis for Django** - To use Redis, you need to install Redis server and the Python client.

    ```bash
    # Install Redis server (Ubuntu/Debian)
    sudo apt-get install redis-server
    
    # Install Redis server (macOS with Homebrew)
    brew install redis
    
    # Install Python Redis client for Django
    pip install django-redis
    
    # Start Redis server
    redis-server
    
    # Test Redis connection
    redis-cli ping  # Should return "PONG"
    ```

61. **Multiple cache configurations** - You can configure multiple cache backends for different purposes.

    ```python
    # settings.py - Multiple cache backends
    CACHES = {
        'default': {
            'BACKEND': 'django_redis.cache.RedisCache',
            'LOCATION': 'redis://127.0.0.1:6379/1',
            'TIMEOUT': 300,
        },
        'sessions': {                                       # Separate cache for sessions
            'BACKEND': 'django_redis.cache.RedisCache',
            'LOCATION': 'redis://127.0.0.1:6379/2',        # Different Redis database
            'TIMEOUT': 3600,                                # 1 hour timeout
        },
        'long_term': {                                      # For data that changes rarely
            'BACKEND': 'django_redis.cache.RedisCache',
            'LOCATION': 'redis://127.0.0.1:6379/3',
            'TIMEOUT': 86400,                               # 24 hours timeout
        }
    }
    
    # Configure sessions to use specific cache
    SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
    SESSION_CACHE_ALIAS = 'sessions'
    ```

## Using Cache in Views

62. **Per-view caching** - Cache entire view responses using decorators. Best for views that don't change often.

    ```python
    from django.views.decorators.cache import cache_page
    from django.shortcuts import render
    from .models import Product
    
    @cache_page(60 * 15)  # Cache for 15 minutes
    def product_list(request):
        products = Product.objects.all()
        return render(request, 'products/list.html', {'products': products})
    
    # For class-based views
    from django.utils.decorators import method_decorator
    from django.views.generic import ListView
    
    @method_decorator(cache_page(60 * 15), name='get')
    class ProductListView(ListView):
        model = Product
        template_name = 'products/list.html'
    ```

63. **Cache with parameters** - Cache different versions based on URL parameters or user data.

    ```python
    from django.views.decorators.cache import cache_page
    from django.views.decorators.vary import vary_on_headers
    
    @cache_page(60 * 15)
    @vary_on_headers('User-Agent')  # Cache different versions for different browsers
    def browser_specific_view(request):
        return render(request, 'browser_specific.html')
    
    # Cache per user (be careful with memory usage)
    from django.views.decorators.cache import cache_page
    from django.contrib.auth.decorators import login_required
    
    @login_required
    @cache_page(60 * 5, key_prefix='user_dashboard')
    def user_dashboard(request):
        # This will cache different versions for each user
        user_data = request.user.get_dashboard_data()
        return render(request, 'dashboard.html', {'data': user_data})
    ```

## Low-Level Cache API

64. **Manual caching with cache API** - Use Django's cache API to cache specific data rather than entire views.

    ```python
    from django.core.cache import cache
    from django.shortcuts import render
    from .models import Product, Category
    
    def product_list(request):
        # Try to get data from cache first
        products = cache.get('all_products')
        
        if products is None:
            # Data not in cache, fetch from database
            products = list(Product.objects.select_related('category').all())
            
            # Store in cache for 15 minutes
            cache.set('all_products', products, 60 * 15)
            print("Fetched from database")  # Debug info
        else:
            print("Fetched from cache")     # Debug info
        
        return render(request, 'products/list.html', {'products': products})
    ```

65. **Cache with complex keys** - Create dynamic cache keys based on multiple parameters.

    ```python
    from django.core.cache import cache
    from django.core.cache.utils import make_template_fragment_key
    
    def get_user_products(request, category_id=None):
        # Create a unique cache key based on user and category
        cache_key = f'user_products_{request.user.id}'
        if category_id:
            cache_key += f'_category_{category_id}'
        
        # Try cache first
        products = cache.get(cache_key)
        
        if products is None:
            # Build query based on parameters
            queryset = Product.objects.filter(user=request.user)
            if category_id:
                queryset = queryset.filter(category_id=category_id)
            
            products = list(queryset.select_related('category'))
            
            # Cache for 10 minutes
            cache.set(cache_key, products, 60 * 10)
        
        return render(request, 'products/user_list.html', {'products': products})
    ```

66. **Cache operations - get, set, delete, get_or_set** - Different ways to interact with cache.

    ```python
    from django.core.cache import cache
    
    def cache_operations_example():
        # Basic set and get
        cache.set('my_key', 'my_value', 300)    # Store for 5 minutes
        value = cache.get('my_key')             # Retrieve value
        
        # Get with default value
        value = cache.get('my_key', 'default_value')  # Returns default if key doesn't exist
        
        # Get or set - atomic operation
        def expensive_calculation():
            import time
            time.sleep(2)  # Simulate expensive operation
            return "calculated_result"
        
        # This will either get from cache or calculate and store
        result = cache.get_or_set('expensive_key', expensive_calculation, 300)
        
        # Set multiple values at once
        cache.set_many({
            'key1': 'value1',
            'key2': 'value2',
            'key3': 'value3'
        }, 300)
        
        # Get multiple values at once
        values = cache.get_many(['key1', 'key2', 'key3'])
        # Returns: {'key1': 'value1', 'key2': 'value2', 'key3': 'value3'}
        
        # Delete from cache
        cache.delete('my_key')
        
        # Delete multiple keys
        cache.delete_many(['key1', 'key2', 'key3'])
        
        # Clear entire cache (use with caution!)
        cache.clear()
    ```

## Template Fragment Caching

67. **Template fragment caching** - Cache parts of templates that are expensive to render.

    ```html
    <!-- products/list.html -->
    {% load cache %}
    
    <h1>Products</h1>
    
    <!-- Cache expensive product list for 15 minutes -->
    {% cache 900 product_list %}
        {% for product in products %}
            <div class="product">
                <h3>{{ product.name }}</h3>
                <p>{{ product.description }}</p>
                <p>Price: ${{ product.price }}</p>
            </div>
        {% endfor %}
    {% endcache %}
    
    <!-- Cache per category -->
    {% cache 600 product_list_by_category category.id %}
        <h2>{{ category.name }} Products</h2>
        {% for product in category.products.all %}
            <div class="product">{{ product.name }}</div>
        {% endfor %}
    {% endcache %}
    
    <!-- Cache per user (be careful with memory) -->
    {% cache 300 user_specific_content user.id %}
        <p>Welcome, {{ user.first_name }}!</p>
        <p>Your last login: {{ user.last_login }}</p>
    {% endcache %}
    ```

## Cache Invalidation

68. **Cache invalidation strategies** - Remove or update cached data when the underlying data changes.

    ```python
    from django.core.cache import cache
    from django.db.models.signals import post_save, post_delete
    from django.dispatch import receiver
    from .models import Product
    
    @receiver(post_save, sender=Product)
    def invalidate_product_cache(sender, instance, **kwargs):
        # Clear cache when product is created or updated
        cache.delete('all_products')
        cache.delete(f'product_{instance.id}')
        cache.delete(f'category_products_{instance.category_id}')
        
        # Clear user-specific caches if needed
        cache.delete(f'user_products_{instance.user_id}')
    
    @receiver(post_delete, sender=Product)
    def invalidate_product_cache_on_delete(sender, instance, **kwargs):
        # Clear cache when product is deleted
        cache.delete('all_products')
        cache.delete(f'category_products_{instance.category_id}')
        cache.delete(f'user_products_{instance.user_id}')
    
    # Manual cache invalidation in views
    from django.shortcuts import redirect
    from django.contrib import messages
    
    def update_product(request, product_id):
        product = Product.objects.get(id=product_id)
        
        if request.method == 'POST':
            # Update product logic here
            product.name = request.POST.get('name')
            product.save()
            
            # Manually invalidate related caches
            cache.delete('all_products')
            cache.delete(f'product_{product_id}')
            cache.delete(f'category_products_{product.category_id}')
            
            messages.success(request, 'Product updated successfully!')
            return redirect('product_detail', product_id)
        
        return render(request, 'products/edit.html', {'product': product})
    ```

## What to Cache - Best Practices

69. **Good candidates for caching** - Cache data that is expensive to compute and doesn't change frequently.

    ```python
    # ✅ GOOD CACHING EXAMPLES
    
    # 1. Database queries that are expensive
    def get_popular_products():
        cache_key = 'popular_products'
        products = cache.get(cache_key)
        
        if products is None:
            # Complex query with joins and aggregations
            products = Product.objects.select_related('category') \
                .annotate(avg_rating=Avg('reviews__rating')) \
                .filter(avg_rating__gte=4.0) \
                .order_by('-avg_rating')[:10]
            
            products = list(products)  # Convert QuerySet to list for caching
            cache.set(cache_key, products, 60 * 30)  # Cache for 30 minutes
        
        return products
    
    # 2. API responses from external services
    def get_weather_data(city):
        cache_key = f'weather_{city}'
        weather = cache.get(cache_key)
        
        if weather is None:
            # Expensive API call
            response = requests.get(f'https://api.weather.com/v1/current?city={city}')
            weather = response.json()
            
            cache.set(cache_key, weather, 60 * 15)  # Cache for 15 minutes
        
        return weather
    
    # 3. Computed/aggregated data
    def get_site_statistics():
        cache_key = 'site_stats'
        stats = cache.get(cache_key)
        
        if stats is None:
            stats = {
                'total_users': User.objects.count(),
                'total_products': Product.objects.count(),
                'total_orders': Order.objects.count(),
                'revenue_this_month': Order.objects.filter(
                    created_at__month=datetime.now().month
                ).aggregate(total=Sum('total_amount'))['total']
            }
            
            cache.set(cache_key, stats, 60 * 60)  # Cache for 1 hour
        
        return stats
    
    # 4. Template fragments that are complex to render
    def get_navigation_menu():
        cache_key = 'navigation_menu'
        menu_html = cache.get(cache_key)
        
        if menu_html is None:
            categories = Category.objects.prefetch_related('subcategories').all()
            # Complex template rendering
            menu_html = render_to_string('includes/navigation.html', {'categories': categories})
            
            cache.set(cache_key, menu_html, 60 * 60 * 24)  # Cache for 24 hours
        
        return menu_html
    ```

70. **What NOT to cache** - Avoid caching data that changes frequently or is user-specific with high memory impact.

    ```python
    # ❌ BAD CACHING EXAMPLES
    
    # 1. DON'T cache frequently changing data
    def get_live_stock_prices():
        # Stock prices change every second - caching doesn't make sense
        cache_key = 'stock_prices'
        prices = cache.get(cache_key)
        if prices is None:
            prices = fetch_live_stock_prices()
            cache.set(cache_key, prices, 1)  # Even 1 second is too long for live data
        return prices
    
    # 2. DON'T cache user-specific data if you have many users
    def get_user_recent_activity(user_id):
        # This creates one cache entry per user - memory intensive
        cache_key = f'user_activity_{user_id}'
        activity = cache.get(cache_key)
        if activity is None:
            activity = UserActivity.objects.filter(user_id=user_id)[:10]
            cache.set(cache_key, activity, 300)  # Creates cache entry for each user
        return activity
    
    # 3. DON'T cache small, simple operations
    def get_user_full_name(user):
        # This operation is already fast - caching adds overhead
        cache_key = f'user_full_name_{user.id}'
        full_name = cache.get(cache_key)
        if full_name is None:
            full_name = f"{user.first_name} {user.last_name}"  # Simple string concatenation
            cache.set(cache_key, full_name, 300)
        return full_name
    
    # 4. DON'T cache sensitive data
    def get_user_payment_info(user_id):
        # Sensitive financial data shouldn't be cached
        cache_key = f'payment_info_{user_id}'
        payment_info = cache.get(cache_key)
        if payment_info is None:
            payment_info = PaymentMethod.objects.filter(user_id=user_id).first()
            cache.set(cache_key, payment_info, 1800)  # Security risk
        return payment_info
    ```

71. **Cache versioning and namespacing** - Use versions and prefixes to manage cache effectively.

    ```python
    from django.core.cache import cache
    from django.conf import settings
    
    # Cache with versioning
    def get_product_data(product_id, version='v1'):
        cache_key = f'product_{product_id}'
        product_data = cache.get(cache_key, version=version)
        
        if product_data is None:
            product = Product.objects.get(id=product_id)
            product_data = {
                'name': product.name,
                'price': str(product.price),
                'description': product.description
            }
            
            # Cache with version - allows easy invalidation of all v1 caches
            cache.set(cache_key, product_data, 1800, version=version)
        
        return product_data
    
    # Namespace caching by feature/app
    class CacheManager:
        def __init__(self, namespace):
            self.namespace = namespace
        
        def make_key(self, key):
            return f"{self.namespace}:{key}"
        
        def get(self, key, default=None):
            return cache.get(self.make_key(key), default)
        
        def set(self, key, value, timeout=300):
            return cache.set(self.make_key(key), value, timeout)
        
        def delete(self, key):
            return cache.delete(self.make_key(key))
    
    # Usage
    product_cache = CacheManager('products')
    user_cache = CacheManager('users')
    
    product_cache.set('bestsellers', bestseller_list, 1800)
    user_cache.set('online_count', online_users_count, 300)
    ```

72. **Cache monitoring and debugging** - Tools and techniques to monitor cache effectiveness.

    ```python
    from django.core.cache import cache
    import logging
    
    logger = logging.getLogger(__name__)
    
    def cache_with_logging(cache_key, fetch_function, timeout=300):
        """Cache wrapper with hit/miss logging"""
        data = cache.get(cache_key)
        
        if data is None:
            logger.info(f"Cache MISS for key: {cache_key}")
            data = fetch_function()
            cache.set(cache_key, data, timeout)
            logger.info(f"Cache SET for key: {cache_key}")
        else:
            logger.info(f"Cache HIT for key: {cache_key}")
        
        return data
    
    # Usage
    def get_expensive_data():
        def fetch_data():
            # Expensive operation
            return expensive_database_query()
        
        return cache_with_logging(
            'expensive_data',
            fetch_data,
            timeout=1800
        )
    
    # Cache statistics view (for debugging)
    from django.http import JsonResponse
    from django.contrib.admin.views.decorators import staff_member_required
    
    @staff_member_required
    def cache_stats(request):
        """View to check cache statistics (Redis only)"""
        try:
            from django_redis import get_redis_connection
            redis_conn = get_redis_connection("default")
            
            info = redis_conn.info()
            stats = {
                'redis_version': info.get('redis_version'),
                'used_memory_human': info.get('used_memory_human'),
                'keyspace_hits': info.get('keyspace_hits'),
                'keyspace_misses': info.get('keyspace_misses'),
                'connected_clients': info.get('connected_clients'),
            }
            
            # Calculate hit ratio
            hits = stats.get('keyspace_hits', 0)
            misses = stats.get('keyspace_misses', 0)
            total = hits + misses
            hit_ratio = (hits / total * 100) if total > 0 else 0
            stats['hit_ratio'] = f"{hit_ratio:.2f}%"
            
            return JsonResponse(stats)
        except Exception as e:
            return JsonResponse({'error': str(e)})
    ```

# Django + React Integration - Complete Guide

## Overview of Django + React Architecture

73. **Django + React integration patterns** - There are several ways to integrate React with Django. Choose based on your project needs.

    ```python
    # PATTERN 1: Django serves React build (most common)
    # - Django handles backend API + serves React static files
    # - React handles all frontend routing and UI
    # - Single domain deployment
    
    # PATTERN 2: Separate servers (development friendly)
    # - Django runs on localhost:8000 (API only)
    # - React runs on localhost:3000 (frontend only)
    # - CORS configured for cross-origin requests
    
    # PATTERN 3: Django templates + React components (hybrid)
    # - Django renders templates with React components embedded
    # - Good for gradually migrating from Django templates to React
    ```

## Setting Up React Build for Django

74. **React project structure** - Organize your React app within or alongside your Django project.

    ```bash
    # PROJECT STRUCTURE OPTION 1: React inside Django
    myproject/
    ├── myproject/          # Django project
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    ├── myapp/              # Django app
    │   ├── models.py
    │   ├── views.py
    │   └── urls.py
    ├── frontend/           # React app directory
    │   ├── public/
    │   ├── src/
    │   ├── package.json
    │   └── build/          # React production build
    ├── static/             # Django static files
    ├── templates/          # Django templates
    └── manage.py
    
    # PROJECT STRUCTURE OPTION 2: Separate directories
    my-fullstack-app/
    ├── backend/            # Django project
    │   ├── myproject/
    │   ├── myapp/
    │   └── manage.py
    └── frontend/           # React app
        ├── public/
        ├── src/
        └── package.json
    ```

75. **Building React for production** - Create production-ready React build that Django can serve.

    ```bash
    # Navigate to your React app directory
    cd frontend
    
    # Install dependencies
    npm install
    
    # Create production build
    npm run build
    
    # This creates a 'build' directory with:
    # build/
    # ├── static/
    # │   ├── css/
    # │   ├── js/
    # │   └── media/
    # ├── index.html
    # └── other static files
    ```

76. **Configure Django to serve React build** - Set up Django settings to serve React static files and handle routing.

    ```python
    # settings.py
    import os
    from pathlib import Path
    
    BASE_DIR = Path(__file__).resolve().parent.parent
    
    # Static files configuration
    STATIC_URL = '/static/'
    STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')  # For production
    
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'frontend/build/static'),  # React build static files
    ]
    
    # Template configuration to serve React's index.html
    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [
                os.path.join(BASE_DIR, 'frontend/build'),  # React build directory
                os.path.join(BASE_DIR, 'templates'),       # Django templates
            ],
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
    
    # For development - serve media files
    if DEBUG:
        STATICFILES_DIRS.append(os.path.join(BASE_DIR, 'static'))
    ```

## URL Configuration and Routing

77. **Django URLs for React routing** - Configure Django to handle API routes and let React handle frontend routing.

    ```python
    # myproject/urls.py (main URLs)
    from django.contrib import admin
    from django.urls import path, include, re_path
    from django.views.generic import TemplateView
    from django.conf import settings
    from django.conf.urls.static import static
    
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('api/', include('myapp.urls')),  # API routes
        path('auth/', include('dj_rest_auth.urls')),  # Authentication API
        
        # Serve React app for all other routes
        re_path(r'^.*$', TemplateView.as_view(template_name='index.html')),
    ]
    
    # Serve static files in development
    if settings.DEBUG:
        urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
        urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    ```

78. **API URLs configuration** - Separate API endpoints from frontend routes.

    ```python
    # myapp/urls.py (API URLs)
    from django.urls import path
    from rest_framework.routers import DefaultRouter
    from . import views
    
    # API endpoints with 'api/' prefix
    urlpatterns = [
        path('products/', views.ProductListCreateView.as_view(), name='product-list'),
        path('products/<int:pk>/', views.ProductDetailView.as_view(), name='product-detail'),
        path('categories/', views.CategoryListView.as_view(), name='category-list'),
        path('users/profile/', views.UserProfileView.as_view(), name='user-profile'),
    ]
    
    # Or using DRF ViewSets and Router
    router = DefaultRouter()
    router.register(r'products', views.ProductViewSet)
    router.register(r'categories', views.CategoryViewSet)
    urlpatterns += router.urls
    ```

## React Configuration for Django Integration

79. **React package.json configuration** - Configure React build to work with Django static files.

    ```json
    {
      "name": "frontend",
      "version": "0.1.0",
      "private": true,
      "homepage": "/",
      "dependencies": {
        "react": "^18.2.0",
        "react-dom": "^18.2.0",
        "react-router-dom": "^6.8.0",
        "axios": "^1.3.0"
      },
      "scripts": {
        "start": "react-scripts start",
        "build": "react-scripts build",
        "build:production": "CI=false && react-scripts build",
        "eject": "react-scripts eject"
      },
      "proxy": "http://localhost:8000",
      "browserslist": {
        "production": [
          ">0.2%",
          "not dead",
          "not op_mini all"
        ],
        "development": [
          "last 1 chrome version",
          "last 1 firefox version",
          "last 1 safari version"
        ]
      }
    }
    ```

80. **React API configuration** - Set up Axios or fetch to communicate with Django API.

    ```javascript
    // src/api/config.js
    import axios from 'axios';
    
    // API base configuration
    const API_BASE_URL = process.env.NODE_ENV === 'production' 
        ? '/api'  // Same domain in production
        : 'http://localhost:8000/api';  // Django dev server
    
    // Create axios instance with default config
    const apiClient = axios.create({
        baseURL: API_BASE_URL,
        timeout: 10000,
        headers: {
            'Content-Type': 'application/json',
        }
    });
    
    // Add request interceptor for authentication
    apiClient.interceptors.request.use(
        (config) => {
            const token = localStorage.getItem('authToken');
            if (token) {
                config.headers.Authorization = `Bearer ${token}`;
            }
            return config;
        },
        (error) => Promise.reject(error)
    );
    
    // Add response interceptor for error handling
    apiClient.interceptors.response.use(
        (response) => response,
        (error) => {
            if (error.response?.status === 401) {
                // Handle unauthorized access
                localStorage.removeItem('authToken');
                window.location.href = '/login';
            }
            return Promise.reject(error);
        }
    );
    
    export default apiClient;
    ```

## React Components Integration

81. **React Router setup for Django** - Configure React Router to work with Django URL handling.

    ```javascript
    // src/App.js
    import React from 'react';
    import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
    import Header from './components/Header';
    import Home from './pages/Home';
    import Products from './pages/Products';
    import ProductDetail from './pages/ProductDetail';
    import Login from './pages/Login';
    import Dashboard from './pages/Dashboard';
    import NotFound from './pages/NotFound';
    import PrivateRoute from './components/PrivateRoute';
    
    function App() {
        return (
            <Router>
                <div className="App">
                    <Header />
                    <main>
                        <Routes>
                            <Route path="/" element={<Home />} />
                            <Route path="/products" element={<Products />} />
                            <Route path="/products/:id" element={<ProductDetail />} />
                            <Route path="/login" element={<Login />} />
                            
                            {/* Protected routes */}
                            <Route path="/dashboard" element={
                                <PrivateRoute>
                                    <Dashboard />
                                </PrivateRoute>
                            } />
                            
                            {/* 404 page */}
                            <Route path="*" element={<NotFound />} />
                        </Routes>
                    </main>
                </div>
            </Router>
        );
    }
    
    export default App;
    ```

82. **React component with Django API integration** - Example component that fetches data from Django.

    ```javascript
    // src/pages/Products.js
    import React, { useState, useEffect } from 'react';
    import { productService } from '../api/services';
    
    const Products = () => {
        const [products, setProducts] = useState([]);
        const [loading, setLoading] = useState(true);
        const [error, setError] = useState(null);
        const [page, setPage] = useState(1);
        const [hasNext, setHasNext] = useState(false);
    
        useEffect(() => {
            fetchProducts();
        }, [page]);
    
        const fetchProducts = async () => {
            try {
                setLoading(true);
                const data = await productService.getProducts({ page });
                
                if (page === 1) {
                    setProducts(data.results);
                } else {
                    setProducts(prev => [...prev, ...data.results]);
                }
                
                setHasNext(!!data.next);
                setError(null);
            } catch (err) {
                setError('Failed to fetch products');
                console.error('Error fetching products:', err);
            } finally {
                setLoading(false);
            }
        };
    
        const handleLoadMore = () => {
            if (hasNext && !loading) {
                setPage(prev => prev + 1);
            }
        };
    
        if (loading && page === 1) {
            return <div className="loading">Loading products...</div>;
        }
    
        if (error) {
            return <div className="error">Error: {error}</div>;
        }
    
        return (
            <div className="products-page">
                <h1>Products</h1>
                
                <div className="products-grid">
                    {products.map(product => (
                        <div key={product.id} className="product-card">
                            <h3>{product.name}</h3>
                            <p>{product.description}</p>
                            <p className="price">${product.price}</p>
                            <button onClick={() => window.location.href = `/products/${product.id}`}>
                                View Details
                            </button>
                        </div>
                    ))}
                </div>
                
                {hasNext && (
                    <button 
                        onClick={handleLoadMore} 
                        disabled={loading}
                        className="load-more-btn"
                    >
                        {loading ? 'Loading...' : 'Load More'}
                    </button>
                )}
            </div>
        );
    };
    
    export default Products;
    ```

## Development Workflow

83. **Development setup - separate servers** - Run Django and React development servers simultaneously.

    ```bash
    # Terminal 1: Start Django development server
    cd backend  # or your Django project root
    python manage.py runserver 8000
    
    # Terminal 2: Start React development server
    cd frontend
    npm start  # Runs on localhost:3000
    
    # React will proxy API requests to Django server
    # Frontend: http://localhost:3000
    # API: http://localhost:8000/api/
    ```

84. **Production deployment setup** - Build and deploy both Django and React together.

    ```bash
    # BUILD SCRIPT (build.sh)
    #!/bin/bash
    
    echo "Building React application..."
    cd frontend
    npm install
    npm run build
    
    echo "Collecting Django static files..."
    cd ..
    python manage.py collectstatic --noinput
    
    echo "Running Django migrations..."
    python manage.py migrate
    
    echo "Build complete!"
    ```

85. **Django settings for production** - Configure Django settings for serving React in production.

    ```python
    # settings/production.py
    from .base import *
    
    DEBUG = False
    ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']
    
    # Static files for production
    STATIC_URL = '/static/'
    STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
    
    # React build files
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'frontend/build/static'),
    ]
    
    # Whitenoise for serving static files (if not using nginx)
    MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        'whitenoise.middleware.WhiteNoiseMiddleware',  # Add this
        'django.contrib.sessions.middleware.SessionMiddleware',
        'corsheaders.middleware.CorsMiddleware',
        'django.middleware.common.CommonMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
        'django.middleware.clickjacking.XFrameOptionsMiddleware',
    ]
    
    # Whitenoise settings
    STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
    
    # CORS settings for production
    CORS_ALLOWED_ORIGINS = [
        "https://yourdomain.com",
        "https://www.yourdomain.com",
    ]
    ```

## CORS Configuration for Development

86. **CORS setup for development** - Configure Cross-Origin Resource Sharing when running separate servers.

    ```python
    # settings.py - Add CORS configuration
    
    # Install: pip install django-cors-headers
    
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'corsheaders',  # Add this
        'rest_framework',
        'myapp',
    ]
    
    MIDDLEWARE = [
        'corsheaders.middleware.CorsMiddleware',  # Add this first
        'django.middleware.security.SecurityMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.middleware.common.CommonMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
        'django.middleware.clickjacking.XFrameOptionsMiddleware',
    ]
    
    # CORS settings for development
    if DEBUG:
        CORS_ALLOWED_ORIGINS = [
            "http://localhost:3000",  # React development server
            "http://127.0.0.1:3000",
        ]
        CORS_ALLOW_CREDENTIALS = True
    ```

## Complete Example: Django View + React Component

87. **Complete integration example** - A working example showing Django API and React frontend communication.

    ```python
    # Django side - views.py
    from rest_framework import generics, status
    from rest_framework.response import Response
    from rest_framework.permissions import IsAuthenticated
    from django.contrib.auth.models import User
    from .models import Product
    from .serializers import ProductSerializer
    
    class ProductListCreateView(generics.ListCreateAPIView):
        queryset = Product.objects.all()
        serializer_class = ProductSerializer
        
        def get_queryset(self):
            queryset = Product.objects.all()
            category = self.request.query_params.get('category')
            search = self.request.query_params.get('search')
            
            if category:
                queryset = queryset.filter(category__name=category)
            if search:
                queryset = queryset.filter(name__icontains=search)
                
            return queryset.order_by('-created_at')
        
        def perform_create(self, serializer):
            serializer.save(created_by=self.request.user)
    ```

    ```javascript
    // React side - ProductForm.js
    import React, { useState } from 'react';
    import { productService } from '../api/services';
    
    const ProductForm = ({ onProductCreated }) => {
        const [formData, setFormData] = useState({
            name: '',
            description: '',
            price: '',
            category: ''
        });
        const [loading, setLoading] = useState(false);
        const [error, setError] = useState(null);
    
        const handleSubmit = async (e) => {
            e.preventDefault();
            setLoading(true);
            setError(null);
    
            try {
                const newProduct = await productService.createProduct(formData);
                setFormData({ name: '', description: '', price: '', category: '' });
                onProductCreated(newProduct);
                alert('Product created successfully!');
            } catch (err) {
                setError(err.response?.data?.message || 'Failed to create product');
            } finally {
                setLoading(false);
            }
        };
    
        const handleChange = (e) => {
            setFormData({
                ...formData,
                [e.target.name]: e.target.value
            });
        };
    
        return (
            <form onSubmit={handleSubmit} className="product-form">
                <h2>Create New Product</h2>
                
                {error && <div className="error">{error}</div>}
                
                <input
                    type="text"
                    name="name"
                    placeholder="Product Name"
                    value={formData.name}
                    onChange={handleChange}
                    required
                />
                
                <textarea
                    name="description"
                    placeholder="Description"
                    value={formData.description}
                    onChange={handleChange}
                    required
                />
                
                <input
                    type="number"
                    name="price"
                    placeholder="Price"
                    value={formData.price}
                    onChange={handleChange}
                    required
                />
                
                <select
                    name="category"
                    value={formData.category}
                    onChange={handleChange}
                    required
                >
                    <option value="">Select Category</option>
                    <option value="electronics">Electronics</option>
                    <option value="clothing">Clothing</option>
                    <option value="books">Books</option>
                </select>
                
                <button type="submit" disabled={loading}>
                    {loading ? 'Creating...' : 'Create Product'}
                </button>
            </form>
        );
    };
    
    export default ProductForm;
    ```

## Understanding Django Static Files and Templates Configuration

88. **Django static files vs templates - the difference** - Understanding what goes where and why.

    ```python
    # Django has TWO separate systems:
    # 1. STATIC FILES SYSTEM - handles CSS, JS, images, fonts
    # 2. TEMPLATE SYSTEM - handles HTML files that Django renders
    
    # React build creates:
    # build/
    # ├── static/          # CSS, JS files -> goes to STATICFILES_DIRS
    # │   ├── css/
    # │   ├── js/
    # │   └── media/
    # ├── index.html       # HTML template -> goes to TEMPLATES DIRS
    # └── other files
    ```

89. **STATIC_ROOT vs STATICFILES_DIRS** - Two different purposes in Django's static file handling.

    ```python
    # settings.py
    
    # STATIC_ROOT: WHERE Django collects all static files for production
    # - Used by 'collectstatic' command to gather all static files in one place
    # - Web server (nginx/Apache) serves files directly from this directory in production
    # - DON'T put files directly here - they get overwritten by collectstatic
    STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')  # Final destination
    
    # STATICFILES_DIRS: WHERE Django looks for static files during development
    # - Django searches these directories for static files
    # - Files from these directories get copied to STATIC_ROOT during collectstatic
    # - Put your source static files here
    STATICFILES_DIRS = [
        # Your custom static files (CSS, JS, images you create)
        os.path.join(BASE_DIR, 'static'),                
        
        # React's built static files (CSS, JS created by npm run build)
        os.path.join(BASE_DIR, 'frontend/build/static'), 
        
        # You can add more directories here
        # os.path.join(BASE_DIR, 'assets'),
    ]
    
    # STATIC_URL: The URL prefix for static files
    # - How browsers access static files
    # - In development: Django serves from STATICFILES_DIRS
    # - In production: Web server serves from STATIC_ROOT
    STATIC_URL = '/static/'
    ```

90. **TEMPLATES DIRS explained** - Where Django looks for HTML template files, not static files.

    ```python
    # TEMPLATES configuration
    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            
            # Where Django looks for HTML template files
            'DIRS': [
                # These are directories containing HTML template files
                os.path.join(BASE_DIR, 'templates'),         # Your Django templates
                os.path.join(BASE_DIR, 'frontend/build'),    # React's index.html
                # NOT frontend/build/static - that's for CSS/JS, not HTML
            ],
            
            # Also look in each app's templates/ directory
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

91. **Why React build/static is NOT in TEMPLATES DIRS** - Common confusion explained with examples.

    ```python
    # ❌ WRONG - DON'T do this
    TEMPLATES = [
        {
            'DIRS': [
                os.path.join(BASE_DIR, 'frontend/build/static'),  # WRONG!
            ],
        },
    ]
    
    # ✅ CORRECT - Here's why:
    
    # React build/static contains:
    # build/static/css/main.abc123.css     # CSS file - NOT an HTML template
    # build/static/js/main.def456.js      # JS file - NOT an HTML template
    # build/static/media/logo.png         # Image file - NOT an HTML template
    
    # These are STATIC FILES, so they belong in STATICFILES_DIRS:
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'frontend/build/static'),     # ✅ Correct place
    ]
    
    # React build root contains:
    # build/index.html                     # HTML file - this IS a template
    # build/manifest.json                  # Config file - could be a template
    
    # HTML files belong in TEMPLATES DIRS:
    TEMPLATES = [
        {
            'DIRS': [
                os.path.join(BASE_DIR, 'frontend/build'),    # ✅ Correct for HTML
            ],
        },
    ]
    ```

92. **Complete file flow example** - How files move from development to production.

    ```bash
    # DEVELOPMENT FLOW:
    # 1. You create static files in various places
    project/
    ├── static/                    # Your custom static files
    │   ├── css/custom.css
    │   └── js/custom.js
    ├── frontend/build/static/     # React's built static files
    │   ├── css/main.abc123.css
    │   └── js/main.def456.js
    └── myapp/static/myapp/        # App-specific static files
        └── style.css
    
    # 2. During development, Django finds static files from STATICFILES_DIRS:
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static'),                # Custom files
        os.path.join(BASE_DIR, 'frontend/build/static'), # React files
    ]
    # Plus Django automatically includes myapp/static/
    
    # PRODUCTION FLOW:
    # 1. Run collectstatic command
    python manage.py collectstatic
    
    # 2. Django copies ALL static files to STATIC_ROOT:
    staticfiles/                   # STATIC_ROOT directory
    ├── css/
    │   ├── custom.css            # From your static/
    │   └── main.abc123.css       # From React build/static/
    ├── js/
    │   ├── custom.js             # From your static/
    │   └── main.def456.js        # From React build/static/
    └── myapp/
        └── style.css             # From myapp/static/
    
    # 3. Web server serves files directly from staticfiles/
    ```

93. **Template file flow example** - How Django finds and serves HTML templates.

    ```bash
    # TEMPLATE SEARCH ORDER:
    # Django looks for templates in this order:
    
    # 1. TEMPLATES DIRS (in order):
    templates/                     # Your Django templates
    ├── base.html
    ├── home.html
    └── products/
        └── list.html
    
    frontend/build/                # React templates
    ├── index.html                # React's main template
    └── static/                   # (not searched for templates)
    
    # 2. APP_DIRS (if True):
    myapp/templates/myapp/         # App-specific templates
    ├── detail.html
    └── form.html
    
    # EXAMPLE: When you call render(request, 'index.html', context)
    # Django searches:
    # 1. templates/index.html        # ❌ Not found
    # 2. frontend/build/index.html   # ✅ Found! (React's index.html)
    # 3. myapp/templates/myapp/index.html  # (not checked, already found)
    ```

94. **Real-world configuration example** - Complete setup with explanations.

    ```python
    # settings.py - Complete static files and templates configuration
    import os
    from pathlib import Path
    
    BASE_DIR = Path(__file__).resolve().parent.parent
    
    # =========================
    # STATIC FILES CONFIGURATION
    # =========================
    
    # URL prefix for static files (how browsers access them)
    STATIC_URL = '/static/'
    
    # Production: where collectstatic puts all static files
    STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
    
    # Development: where Django looks for static files
    STATICFILES_DIRS = [
        # Your custom static files (CSS, JS, images you create)
        os.path.join(BASE_DIR, 'static'),                
        
        # React's built static files (CSS, JS created by npm run build)
        os.path.join(BASE_DIR, 'frontend/build/static'), 
        
        # You can add more directories here
        # os.path.join(BASE_DIR, 'assets'),
    ]
    
    # =========================
    # TEMPLATES CONFIGURATION  
    # =========================
    
    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            
            # Where Django looks for HTML template files
            'DIRS': [
                # Your Django HTML templates
                os.path.join(BASE_DIR, 'templates'),
                
                # React's index.html (the main React app entry point)
                os.path.join(BASE_DIR, 'frontend/build'),
                
                # NOTE: We DON'T include frontend/build/static here
                # because that contains CSS/JS files, not HTML templates
            ],
            
            # Also look in each app's templates/ directory
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
    
    # =========================
    # MEDIA FILES (user uploads)
    # =========================
    
    MEDIA_URL = '/media/'
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
    ```

95. **Debugging static files and templates** - Common issues and how to fix them.

    ```python
    # DEBUG STATIC FILES ISSUES
    
    # Problem: Static files not loading in development
    # Solution: Check these settings
    DEBUG = True  # Must be True for Django to serve static files
    
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'frontend/build/static'),  # Path must exist
    ]
    
    # Check if directory exists
    import os
    static_dir = os.path.join(BASE_DIR, 'frontend/build/static')
    print(f"Static directory exists: {os.path.exists(static_dir)}")
    print(f"Static directory contents: {os.listdir(static_dir) if os.path.exists(static_dir) else 'Directory not found'}")
    
    # Problem: Templates not loading
    # Solution: Check template paths
    TEMPLATES = [
        {
            'DIRS': [
                os.path.join(BASE_DIR, 'frontend/build'),  # Must contain index.html
            ],
        }
    ]
    
    # Problem: Static files not loading in production
    # Solution: Run collectstatic and check web server config
    # 1. python manage.py collectstatic
    # 2. Make sure web server serves STATIC_ROOT at STATIC_URL
    ```

96. **URL patterns for serving files** - How Django URLs work with static files and templates.

    ```python
    # urls.py - Complete URL configuration
    from django.contrib import admin
    from django.urls import path, include, re_path
    from django.views.generic import TemplateView
    from django.conf import settings
    from django.conf.urls.static import static
    
    urlpatterns = [
        # Admin interface
        path('admin/', admin.site.urls),
        
        # API endpoints
        path('api/', include('myapp.urls')),
        
        # Authentication APIs
        path('auth/', include('dj_rest_auth.urls')),
        
        # React app - catches ALL remaining URLs
        # Django will look for 'index.html' in TEMPLATES DIRS
        re_path(r'^.*$', TemplateView.as_view(template_name='index.html')),
    ]
    
    # Serve static files in development
    if settings.DEBUG:
        # Serve static files from STATICFILES_DIRS
        urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
        
        # Serve user-uploaded files
        urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
        
    # Production: Web server (nginx/Apache) handles static files directly
    # Django never sees requests to /static/* or /media/*
    ```
