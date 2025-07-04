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

# Django Models - Complete Guide

## Model Definition and Fields

39. **Django Models basics** - Models define the structure of your database tables. Each model class represents a database table, and each attribute represents a database field.

    ```python
    # models.py
    from django.db import models
    from django.contrib.auth.models import User
    from django.utils import timezone
    
    class Category(models.Model):
        name = models.CharField(max_length=100, unique=True)
        description = models.TextField(blank=True)
        created_at = models.DateTimeField(auto_now_add=True)
        updated_at = models.DateTimeField(auto_now=True)
        is_active = models.BooleanField(default=True)
        
        class Meta:
            verbose_name_plural = "Categories"
            ordering = ['name']
            
        def __str__(self):
            return self.name
    
    class Product(models.Model):
        name = models.CharField(max_length=200)
        description = models.TextField()
        price = models.DecimalField(max_digits=10, decimal_places=2)
        category = models.ForeignKey(Category, on_delete=models.CASCADE, related_name='products')
        created_by = models.ForeignKey(User, on_delete=models.CASCADE)
        created_at = models.DateTimeField(auto_now_add=True)
        updated_at = models.DateTimeField(auto_now=True)
        
        class Meta:
            ordering = ['-created_at']
            
        def __str__(self):
            return self.name
    ```

40. **Common field types** - Choose the right field type for your data to ensure proper validation and database optimization.

    ```python
    class ExampleModel(models.Model):
        # Text fields
        char_field = models.CharField(max_length=100)           # Short text, required max_length
        text_field = models.TextField()                         # Long text, no length limit
        slug_field = models.SlugField(max_length=100)          # URL-friendly text
        email_field = models.EmailField()                      # Email validation
        url_field = models.URLField()                          # URL validation
        
        # Number fields
        integer_field = models.IntegerField()                  # Integer numbers
        decimal_field = models.DecimalField(max_digits=10, decimal_places=2)  # Precise decimals
        float_field = models.FloatField()                      # Floating point numbers
        
        # Date/time fields
        date_field = models.DateField()                        # Date only
        datetime_field = models.DateTimeField()               # Date and time
        time_field = models.TimeField()                        # Time only
        auto_now_add = models.DateTimeField(auto_now_add=True) # Set once on creation
        auto_now = models.DateTimeField(auto_now=True)         # Update every save
        
        # Boolean fields
        boolean_field = models.BooleanField()                  # True/False, required
        null_boolean = models.BooleanField(null=True, blank=True)  # True/False/None
        
        # File fields
        file_field = models.FileField(upload_to='uploads/')    # Any file
        image_field = models.ImageField(upload_to='images/')   # Image files only
        
        # Choice fields
        STATUS_CHOICES = [
            ('draft', 'Draft'),
            ('published', 'Published'),
            ('archived', 'Archived'),
        ]
        status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='draft')
        
        # JSON field (PostgreSQL, MySQL 5.7+)
        json_field = models.JSONField(default=dict, blank=True)
    ```

41. **Model relationships** - Define how models relate to each other using ForeignKey, OneToOneField, and ManyToManyField.

    ```python
    # One-to-Many relationship (ForeignKey)
    class Author(models.Model):
        name = models.CharField(max_length=100)
        email = models.EmailField()
        
        def __str__(self):
            return self.name
    
    class Book(models.Model):
        title = models.CharField(max_length=200)
        author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name='books')
        published_date = models.DateField()
        
        def __str__(self):
            return self.title
    
    # One-to-One relationship
    class UserProfile(models.Model):
        user = models.OneToOneField(User, on_delete=models.CASCADE)
        bio = models.TextField(max_length=500, blank=True)
        location = models.CharField(max_length=30, blank=True)
        birth_date = models.DateField(null=True, blank=True)
        
        def __str__(self):
            return f"{self.user.username}'s profile"
    
    # Many-to-Many relationship
    class Tag(models.Model):
        name = models.CharField(max_length=50, unique=True)
        
        def __str__(self):
            return self.name
    
    class Article(models.Model):
        title = models.CharField(max_length=200)
        content = models.TextField()
        tags = models.ManyToManyField(Tag, related_name='articles', blank=True)
        author = models.ForeignKey(User, on_delete=models.CASCADE)
        
        def __str__(self):
            return self.title
    ```

42. **Model methods and properties** - Add custom methods and properties to your models for business logic.

    ```python
    class Product(models.Model):
        name = models.CharField(max_length=200)
        price = models.DecimalField(max_digits=10, decimal_places=2)
        cost = models.DecimalField(max_digits=10, decimal_places=2)
        created_at = models.DateTimeField(auto_now_add=True)
        
        def __str__(self):
            return self.name
        
        # Custom methods
        def get_profit_margin(self):
            """Calculate profit margin as percentage"""
            if self.cost > 0:
                return ((self.price - self.cost) / self.cost) * 100
            return 0
        
        def is_recently_created(self):
            """Check if product was created in the last 30 days"""
            from datetime import datetime, timedelta
            return self.created_at >= timezone.now() - timedelta(days=30)
        
        # Properties
        @property
        def profit(self):
            """Calculate absolute profit"""
            return self.price - self.cost
        
        @property
        def formatted_price(self):
            """Return price formatted as currency"""
            return f"${self.price:.2f}"
        
        # Class methods
        @classmethod
        def get_expensive_products(cls, min_price=100):
            """Get products above a certain price"""
            return cls.objects.filter(price__gte=min_price)
        
        # Save method override
        def save(self, *args, **kwargs):
            # Custom logic before saving
            if not self.slug:
                self.slug = self.name.lower().replace(' ', '-')
            super().save(*args, **kwargs)
    ```

## Django ORM - Querying Data

43. **Basic queries** - Essential ORM operations for retrieving data from the database.

    ```python
    # Get all objects
    products = Product.objects.all()
    
    # Get single object
    product = Product.objects.get(id=1)                    # Raises DoesNotExist if not found
    product = Product.objects.filter(id=1).first()        # Returns None if not found
    
    # Filter objects
    active_products = Product.objects.filter(is_active=True)
    expensive_products = Product.objects.filter(price__gte=100)
    recent_products = Product.objects.filter(created_at__gte=timezone.now() - timedelta(days=7))
    
    # Exclude objects
    not_electronics = Product.objects.exclude(category__name='Electronics')
    
    # Order objects
    products_by_price = Product.objects.order_by('price')           # Ascending
    products_by_price_desc = Product.objects.order_by('-price')     # Descending
    products_multi_order = Product.objects.order_by('category', '-price')  # Multiple fields
    
    # Limit results
    first_10_products = Product.objects.all()[:10]
    products_5_to_15 = Product.objects.all()[5:15]
    
    # Check if objects exist
    has_products = Product.objects.exists()
    has_expensive_products = Product.objects.filter(price__gte=1000).exists()
    
    # Count objects
    total_products = Product.objects.count()
    expensive_count = Product.objects.filter(price__gte=100).count()
    ```

44. **Advanced filtering with field lookups** - Use Django's powerful field lookup system for complex queries.

    ```python
    # Text field lookups
    Product.objects.filter(name__contains='laptop')           # Case-sensitive contains
    Product.objects.filter(name__icontains='laptop')          # Case-insensitive contains
    Product.objects.filter(name__startswith='Apple')          # Starts with
    Product.objects.filter(name__endswith='Pro')              # Ends with
    Product.objects.filter(name__exact='MacBook Pro')         # Exact match
    Product.objects.filter(name__iexact='macbook pro')        # Case-insensitive exact
    Product.objects.filter(name__regex=r'^Apple.*Pro$')       # Regular expression
    
    # Number field lookups
    Product.objects.filter(price__lt=100)                     # Less than
    Product.objects.filter(price__lte=100)                    # Less than or equal
    Product.objects.filter(price__gt=100)                     # Greater than
    Product.objects.filter(price__gte=100)                    # Greater than or equal
    Product.objects.filter(price__range=(50, 200))            # Between values
    
    # Date field lookups
    Product.objects.filter(created_at__year=2023)             # Specific year
    Product.objects.filter(created_at__month=12)              # Specific month
    Product.objects.filter(created_at__day=25)                # Specific day
    Product.objects.filter(created_at__week_day=1)            # Day of week (1=Sunday)
    Product.objects.filter(created_at__date=timezone.now().date())  # Specific date
    
    # Null/empty lookups
    Product.objects.filter(description__isnull=True)          # NULL values
    Product.objects.filter(description__isnull=False)         # NOT NULL values
    Product.objects.filter(name__in=['iPhone', 'iPad'])       # In list
    Product.objects.exclude(name__in=['iPhone', 'iPad'])      # Not in list
    ```

45. **Complex queries with Q objects** - Combine multiple conditions with AND, OR, and NOT logic.

    ```python
    from django.db.models import Q
    
    # OR conditions
    expensive_or_popular = Product.objects.filter(
        Q(price__gte=1000) | Q(views__gte=10000)
    )
    
    # AND conditions (default behavior, but explicit)
    expensive_and_popular = Product.objects.filter(
        Q(price__gte=1000) & Q(views__gte=10000)
    )
    
    # NOT conditions
    not_expensive = Product.objects.filter(~Q(price__gte=1000))
    
    # Complex combinations
    complex_query = Product.objects.filter(
        Q(category__name='Electronics') & 
        (Q(price__lt=100) | Q(price__gt=1000)) &
        ~Q(name__contains='Refurbished')
    )
    
    # Dynamic query building
    def search_products(name=None, min_price=None, max_price=None, category=None):
        query = Q()
        
        if name:
            query &= Q(name__icontains=name)
        if min_price:
            query &= Q(price__gte=min_price)
        if max_price:
            query &= Q(price__lte=max_price)
        if category:
            query &= Q(category__name=category)
        
        return Product.objects.filter(query)
    ```

46. **Relationship queries** - Query across model relationships efficiently.

    ```python
    # Forward relationship queries (following ForeignKey)
    # Get products from a specific category
    electronics = Product.objects.filter(category__name='Electronics')
    
    # Get products created by users with specific email domain
    gmail_users_products = Product.objects.filter(created_by__email__endswith='@gmail.com')
    
    # Chain relationships
    products_by_active_users = Product.objects.filter(
        created_by__is_active=True,
        created_by__profile__location='New York'
    )
    
    # Reverse relationship queries (following related_name)
    # Get categories that have products
    categories_with_products = Category.objects.filter(products__isnull=False).distinct()
    
    # Get categories with expensive products
    categories_with_expensive = Category.objects.filter(products__price__gte=1000).distinct()
    
    # Get users who have created products
    authors = User.objects.filter(product__isnull=False).distinct()
    
    # Count related objects
    categories_with_count = Category.objects.filter(products__count__gt=5)
    ```

47. **Aggregation and annotation** - Perform calculations and add computed fields to your queries.

    ```python
    from django.db.models import Count, Sum, Avg, Max, Min, F
    
    # Aggregation - single values for entire queryset
    total_products = Product.objects.count()
    average_price = Product.objects.aggregate(Avg('price'))['price__avg']
    max_price = Product.objects.aggregate(Max('price'))['price__max']
    price_stats = Product.objects.aggregate(
        avg_price=Avg('price'),
        max_price=Max('price'),
        min_price=Min('price'),
        total_value=Sum('price')
    )
    
    # Annotation - add calculated fields to each object
    # Add product count to each category
    categories = Category.objects.annotate(
        product_count=Count('products')
    )
    
    # Add average price to each category
    categories = Category.objects.annotate(
        avg_price=Avg('products__price'),
        max_price=Max('products__price')
    )
    
    # Filter by annotated fields
    popular_categories = Category.objects.annotate(
        product_count=Count('products')
    ).filter(product_count__gte=10)
    
    # F expressions - reference model fields
    # Update all prices by 10%
    Product.objects.update(price=F('price') * 1.1)
    
    # Find products where price is higher than cost
    profitable_products = Product.objects.filter(price__gt=F('cost'))
    
    # Annotate with calculated fields
    products_with_profit = Product.objects.annotate(
        profit=F('price') - F('cost'),
        profit_margin=F('price') / F('cost') * 100
    )
    ```

48. **Query optimization** - Techniques to make your queries faster and more efficient.

    ```python
    # Use select_related() for forward relationships (ForeignKey, OneToOneField)
    # Instead of N+1 queries, use JOINs
    
    # BAD - causes N+1 queries
    products = Product.objects.all()
    for product in products:
        print(product.category.name)  # Each iteration hits the database
    
    # GOOD - single query with JOIN
    products = Product.objects.select_related('category', 'created_by').all()
    for product in products:
        print(product.category.name)  # No additional database hits
    
    # Use prefetch_related() for reverse relationships (ManyToMany, reverse ForeignKey)
    # BAD - causes N+1 queries
    categories = Category.objects.all()
    for category in categories:
        print(category.products.count())  # Each iteration hits the database
    
    # GOOD - separate optimized queries
    categories = Category.objects.prefetch_related('products').all()
    for category in categories:
        print(category.products.count())  # Uses prefetched data
    
    # Combine select_related and prefetch_related
    products = Product.objects.select_related('category').prefetch_related('tags').all()
    
    # Use only() to fetch specific fields only
    products = Product.objects.only('name', 'price').all()  # Only fetch name and price
    
    # Use defer() to exclude specific fields
    products = Product.objects.defer('description').all()  # Fetch everything except description
    
    # Use values() for dictionary results (lighter than model instances)
    product_data = Product.objects.values('name', 'price', 'category__name')
    
    # Use values_list() for tuple results
    product_names = Product.objects.values_list('name', flat=True)  # Returns list of names
    price_pairs = Product.objects.values_list('name', 'price')      # Returns list of tuples
    ```

# Django Forms - Complete Guide

## Form Definition and Field Types

49. **Django Forms basics** - Forms handle user input validation and HTML generation automatically.

    ```python
    # forms.py
    from django import forms
    from django.contrib.auth.models import User
    from .models import Product, Category
    
    class ProductForm(forms.Form):
        name = forms.CharField(max_length=200)
        description = forms.CharField(widget=forms.Textarea)
        price = forms.DecimalField(max_digits=10, decimal_places=2)
        category = forms.ModelChoiceField(queryset=Category.objects.all())
        is_active = forms.BooleanField(required=False)
        
        def clean_price(self):
            price = self.cleaned_data.get('price')
            if price and price <= 0:
                raise forms.ValidationError("Price must be positive")
            return price
    
    # ModelForm - automatically generates form from model
    class ProductModelForm(forms.ModelForm):
        class Meta:
            model = Product
            fields = ['name', 'description', 'price', 'category']
            widgets = {
                'description': forms.Textarea(attrs={'rows': 4}),
                'price': forms.NumberInput(attrs={'step': '0.01'}),
            }
            labels = {
                'name': 'Product Name',
                'description': 'Product Description',
            }
            help_texts = {
                'price': 'Enter price in USD',
            }
    ```

50. **Form field types and widgets** - Choose appropriate field types and customize their appearance.

    ```python
    class ExampleForm(forms.Form):
        # Text inputs
        char_field = forms.CharField(max_length=100)
        text_field = forms.CharField(widget=forms.Textarea)
        email_field = forms.EmailField()
        url_field = forms.URLField()
        slug_field = forms.SlugField()
        
        # Number inputs
        integer_field = forms.IntegerField()
        float_field = forms.FloatField()
        decimal_field = forms.DecimalField(max_digits=10, decimal_places=2)
        
        # Choice fields
        CHOICES = [
            ('option1', 'Option 1'),
            ('option2', 'Option 2'),
            ('option3', 'Option 3'),
        ]
        choice_field = forms.ChoiceField(choices=CHOICES)
        multiple_choice = forms.MultipleChoiceField(choices=CHOICES)
        
        # Boolean fields
        boolean_field = forms.BooleanField()
        null_boolean = forms.NullBooleanField()
        
        # Date/time fields
        date_field = forms.DateField()
        datetime_field = forms.DateTimeField()
        time_field = forms.TimeField()
        
        # File fields
        file_field = forms.FileField()
        image_field = forms.ImageField()
        
        # Model fields
        category = forms.ModelChoiceField(queryset=Category.objects.all())
        tags = forms.ModelMultipleChoiceField(queryset=Tag.objects.all())
        
        # Custom widgets
        password_field = forms.CharField(widget=forms.PasswordInput)
        hidden_field = forms.CharField(widget=forms.HiddenInput)
        custom_textarea = forms.CharField(
            widget=forms.Textarea(attrs={
                'rows': 5,
                'cols': 40,
                'placeholder': 'Enter your message here...'
            })
        )
    ```

51. **Form validation** - Implement field-level and form-level validation for user input.

    ```python
    class ProductForm(forms.ModelForm):
        class Meta:
            model = Product
            fields = ['name', 'description', 'price', 'category']
        
        def clean_name(self):
            """Field-level validation for name"""
            name = self.cleaned_data.get('name')
            if name and len(name) < 3:
                raise forms.ValidationError("Name must be at least 3 characters long")
            
            # Check for duplicate names (excluding current instance if editing)
            queryset = Product.objects.filter(name=name)
            if self.instance and self.instance.pk:
                queryset = queryset.exclude(pk=self.instance.pk)
            if queryset.exists():
                raise forms.ValidationError("A product with this name already exists")
            
            return name
        
        def clean_price(self):
            """Field-level validation for price"""
            price = self.cleaned_data.get('price')
            if price and price <= 0:
                raise forms.ValidationError("Price must be positive")
            if price and price > 10000:
                raise forms.ValidationError("Price cannot exceed $10,000")
            return price
        
        def clean(self):
            """Form-level validation"""
            cleaned_data = super().clean()
            name = cleaned_data.get('name')
            description = cleaned_data.get('description')
            
            # Check if name appears in description
            if name and description and name.lower() in description.lower():
                raise forms.ValidationError(
                    "Product name should not appear in the description"
                )
            
            return cleaned_data
    ```

52. **Using forms in views** - Handle form display, validation, and processing in your views.

    ```python
    # views.py
    from django.shortcuts import render, redirect, get_object_or_404
    from django.contrib import messages
    from .forms import ProductForm
    from .models import Product
    
    def create_product(request):
        if request.method == 'POST':
            form = ProductForm(request.POST)
            if form.is_valid():
                product = form.save(commit=False)
                product.created_by = request.user
                product.save()
                messages.success(request, 'Product created successfully!')
                return redirect('product_detail', pk=product.pk)
        else:
            form = ProductForm()
        
        return render(request, 'products/create.html', {'form': form})
    
    def update_product(request, pk):
        product = get_object_or_404(Product, pk=pk)
        
        if request.method == 'POST':
            form = ProductForm(request.POST, instance=product)
            if form.is_valid():
                form.save()
                messages.success(request, 'Product updated successfully!')
                return redirect('product_detail', pk=product.pk)
        else:
            form = ProductForm(instance=product)
        
        return render(request, 'products/update.html', {'form': form, 'product': product})
    
    # Function-based view with manual form handling
    def manual_form_handling(request):
        if request.method == 'POST':
            form = ProductForm(request.POST)
            if form.is_valid():
                # Access cleaned data
                name = form.cleaned_data['name']
                price = form.cleaned_data['price']
                category = form.cleaned_data['category']
                
                # Manual object creation
                product = Product.objects.create(
                    name=name,
                    price=price,
                    category=category,
                    created_by=request.user
                )
                return redirect('product_detail', pk=product.pk)
        else:
            form = ProductForm()
        
        return render(request, 'products/manual_form.html', {'form': form})
    ```

# Django Views - Complete Guide

## Function-Based Views (FBVs)

53. **Function-based views basics** - Simple functions that take a request and return a response.

    ```python
    # views.py
    from django.shortcuts import render, get_object_or_404, redirect
    from django.http import HttpResponse, JsonResponse
    from django.contrib.auth.decorators import login_required
    from django.contrib import messages
    from .models import Product, Category
    
    def product_list(request):
        products = Product.objects.all()
        return render(request, 'products/list.html', {'products': products})
    
    def product_detail(request, pk):
        product = get_object_or_404(Product, pk=pk)
        return render(request, 'products/detail.html', {'product': product})
    
    @login_required
    def product_create(request):
        if request.method == 'POST':
            # Handle form submission
            name = request.POST.get('name')
            price = request.POST.get('price')
            category_id = request.POST.get('category')
            
            product = Product.objects.create(
                name=name,
                price=price,
                category_id=category_id,
                created_by=request.user
            )
            
            messages.success(request, 'Product created successfully!')
            return redirect('product_detail', pk=product.pk)
        
        # GET request - show form
        categories = Category.objects.all()
        return render(request, 'products/create.html', {'categories': categories})
    
    def product_search(request):
        query = request.GET.get('q', '')
        products = Product.objects.filter(name__icontains=query) if query else []
        
        # Return JSON for AJAX requests
        if request.headers.get('X-Requested-With') == 'XMLHttpRequest':
            data = [{'id': p.id, 'name': p.name, 'price': str(p.price)} for p in products]
            return JsonResponse({'products': data})
        
        return render(request, 'products/search.html', {'products': products, 'query': query})
    ```

54. **Class-Based Views (CBVs)** - Reusable view classes with built-in functionality.

    ```python
    from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
    from django.contrib.auth.mixins import LoginRequiredMixin
    from django.urls import reverse_lazy
    from .models import Product
    from .forms import ProductForm
    
    class ProductListView(ListView):
        model = Product
        template_name = 'products/list.html'
        context_object_name = 'products'
        paginate_by = 10
        
        def get_queryset(self):
            queryset = super().get_queryset()
            category = self.request.GET.get('category')
            if category:
                queryset = queryset.filter(category__name=category)
            return queryset.order_by('-created_at')
        
        def get_context_data(self, **kwargs):
            context = super().get_context_data(**kwargs)
            context['categories'] = Category.objects.all()
            return context
    
    class ProductDetailView(DetailView):
        model = Product
        template_name = 'products/detail.html'
        context_object_name = 'product'
        
        def get_context_data(self, **kwargs):
            context = super().get_context_data(**kwargs)
            context['related_products'] = Product.objects.filter(
                category=self.object.category
            ).exclude(id=self.object.id)[:5]
            return context
    
    class ProductCreateView(LoginRequiredMixin, CreateView):
        model = Product
        form_class = ProductForm
        template_name = 'products/create.html'
        success_url = reverse_lazy('product_list')
        
        def form_valid(self, form):
            form.instance.created_by = self.request.user
            messages.success(self.request, 'Product created successfully!')
            return super().form_valid(form)
    
    class ProductUpdateView(LoginRequiredMixin, UpdateView):
        model = Product
        form_class = ProductForm
        template_name = 'products/update.html'
        
        def get_success_url(self):
            return reverse_lazy('product_detail', kwargs={'pk': self.object.pk})
        
        def form_valid(self, form):
            messages.success(self.request, 'Product updated successfully!')
            return super().form_valid(form)
    
    class ProductDeleteView(LoginRequiredMixin, DeleteView):
        model = Product
        template_name = 'products/delete.html'
        success_url = reverse_lazy('product_list')
        
        def delete(self, request, *args, **kwargs):
            messages.success(self.request, 'Product deleted successfully!')
            return super().delete(request, *args, **kwargs)
    ```

55. **Custom mixins and view utilities** - Create reusable view components.

    ```python
    from django.contrib.auth.mixins import UserPassesTestMixin
    from django.core.exceptions import PermissionDenied
    
    class OwnerRequiredMixin(UserPassesTestMixin):
        """Ensure only the owner can access/modify the object"""
        def test_func(self):
            obj = self.get_object()
            return obj.created_by == self.request.user
    
    class CategoryFilterMixin:
        """Add category filtering to list views"""
        def get_queryset(self):
            queryset = super().get_queryset()
            category = self.request.GET.get('category')
            if category:
                queryset = queryset.filter(category__slug=category)
            return queryset
        
        def get_context_data(self, **kwargs):
            context = super().get_context_data(**kwargs)
            context['categories'] = Category.objects.all()
            context['current_category'] = self.request.GET.get('category')
            return context
    
    # Using custom mixins
    class UserProductListView(LoginRequiredMixin, CategoryFilterMixin, ListView):
        model = Product
        template_name = 'products/user_products.html'
        
        def get_queryset(self):
            queryset = super().get_queryset()
            return queryset.filter(created_by=self.request.user)
    
    class ProductUpdateView(LoginRequiredMixin, OwnerRequiredMixin, UpdateView):
        model = Product
        form_class = ProductForm
        template_name = 'products/update.html'
    ```

# Django Admin - Complete Guide

## Basic Admin Configuration

56. **Django Admin setup** - Configure your models to appear in the Django admin interface.

    ```python
    # admin.py
    from django.contrib import admin
    from .models import Category, Product, Tag
    
    # Simple admin registration
    admin.site.register(Category)
    
    # Custom admin configuration
    @admin.register(Product)
    class ProductAdmin(admin.ModelAdmin):
        list_display = ['name', 'price', 'category', 'created_at', 'is_active']
        list_filter = ['category', 'is_active', 'created_at']
        search_fields = ['name', 'description']
        list_editable = ['price', 'is_active']
        list_per_page = 25
        date_hierarchy = 'created_at'
        
        # Field organization
        fieldsets = (
            ('Basic Information', {
                'fields': ('name', 'description', 'category')
            }),
            ('Pricing', {
                'fields': ('price', 'cost')
            }),
            ('Status', {
                'fields': ('is_active',),
                'classes': ('collapse',)
            }),
        )
        
        # Read-only fields
        readonly_fields = ['created_at', 'updated_at']
        
        # Custom actions
        actions = ['make_inactive', 'make_active']
        
        def make_inactive(self, request, queryset):
            queryset.update(is_active=False)
            self.message_user(request, f'{queryset.count()} products were marked as inactive.')
        make_inactive.short_description = "Mark selected products as inactive"
        
        def make_active(self, request, queryset):
            queryset.update(is_active=True)
            self.message_user(request, f'{queryset.count()} products were marked as active.')
        make_active.short_description = "Mark selected products as active"
    ```

57. **Advanced admin customization** - Add computed fields, custom methods, and complex functionality.

    ```python
    @admin.register(Product)
    class ProductAdmin(admin.ModelAdmin):
        list_display = ['name', 'price', 'category', 'profit_margin', 'created_at']
        list_filter = ['category', 'created_at', ProfitMarginFilter]
        search_fields = ['name', 'description', 'category__name']
        
        # Custom methods for list_display
        def profit_margin(self, obj):
            if obj.cost > 0:
                margin = ((obj.price - obj.cost) / obj.cost) * 100
                return f"{margin:.1f}%"
            return "N/A"
        profit_margin.short_description = "Profit Margin"
        profit_margin.admin_order_field = 'price'  # Allow sorting
        
        # Custom filter
        def get_queryset(self, request):
            queryset = super().get_queryset(request)
            if not request.user.is_superuser:
                queryset = queryset.filter(created_by=request.user)
            return queryset
        
        # Custom form validation
        def save_model(self, request, obj, form, change):
            if not change:  # Creating new object
                obj.created_by = request.user
            super().save_model(request, obj, form, change)
    
    # Custom filter class
    class ProfitMarginFilter(admin.SimpleListFilter):
        title = 'profit margin'
        parameter_name = 'profit_margin'
        
        def lookups(self, request, model_admin):
            return (
                ('high', 'High (>50%)'),
                ('medium', 'Medium (20-50%)'),
                ('low', 'Low (<20%)'),
            )
        
        def queryset(self, request, queryset):
            if self.value() == 'high':
                return queryset.filter(price__gt=F('cost') * 1.5)
            elif self.value() == 'medium':
                return queryset.filter(
                    price__gt=F('cost') * 1.2,
                    price__lte=F('cost') * 1.5
                )
            elif self.value() == 'low':
                return queryset.filter(price__lte=F('cost') * 1.2)
    ```

58. **Inline admin models** - Edit related models on the same page.

    ```python
    class ProductImageInline(admin.TabularInline):
        model = ProductImage
        extra = 1
        fields = ['image', 'alt_text', 'is_primary']
        readonly_fields = ['image_preview']
        
        def image_preview(self, obj):
            if obj.image:
                return format_html(
                    '<img src="{}" width="50" height="50" />',
                    obj.image.url
                )
            return "No image"
        image_preview.short_description = "Preview"
    
    class ProductReviewInline(admin.StackedInline):
        model = ProductReview
        extra = 0
        fields = ['reviewer', 'rating', 'comment']
        readonly_fields = ['created_at']
    
    @admin.register(Product)
    class ProductAdmin(admin.ModelAdmin):
        inlines = [ProductImageInline, ProductReviewInline]
        list_display = ['name', 'price', 'category', 'image_count', 'avg_rating']
        
        def image_count(self, obj):
            return obj.images.count()
        image_count.short_description = "Images"
        
        def avg_rating(self, obj):
            avg = obj.reviews.aggregate(Avg('rating'))['rating__avg']
            return f"{avg:.1f}" if avg else "No reviews"
        avg_rating.short_description = "Average Rating"
    ```

# Django Authentication & Authorization - Complete Guide

## User Authentication

59. **Django's built-in authentication** - Use Django's authentication system for login, logout, and user management.

    ```python
    # views.py
    from django.contrib.auth import authenticate, login, logout
    from django.contrib.auth.decorators import login_required
    from django.contrib.auth.forms import UserCreationForm
    from django.shortcuts import render, redirect
    from django.contrib import messages
    
    def user_login(request):
        if request.method == 'POST':
            username = request.POST.get('username')
            password = request.POST.get('password')
            
            user = authenticate(request, username=username, password=password)
            if user is not None:
                login(request, user)
                messages.success(request, 'Logged in successfully!')
                return redirect('dashboard')
            else:
                messages.error(request, 'Invalid username or password.')
        
        return render(request, 'accounts/login.html')
    
    @login_required
    def user_logout(request):
        logout(request)
        messages.success(request, 'Logged out successfully!')
        return redirect('home')
    
    def user_register(request):
        if request.method == 'POST':
            form = UserCreationForm(request.POST)
            if form.is_valid():
                user = form.save()
                login(request, user)
                messages.success(request, 'Account created successfully!')
                return redirect('dashboard')
        else:
            form = UserCreationForm()
        
        return render(request, 'accounts/register.html', {'form': form})
    
    @login_required
    def dashboard(request):
        return render(request, 'accounts/dashboard.html')
    ```

60. **Custom user model** - Extend Django's default User model with additional fields.

    ```python
    # models.py
    from django.contrib.auth.models import AbstractUser
    from django.db import models
    
    class CustomUser(AbstractUser):
        email = models.EmailField(unique=True)
        phone = models.CharField(max_length=20, blank=True)
        date_of_birth = models.DateField(null=True, blank=True)
        profile_picture = models.ImageField(upload_to='profiles/', blank=True)
        is_verified = models.BooleanField(default=False)
        
        # Use email as username
        USERNAME_FIELD = 'email'
        REQUIRED_FIELDS = ['username']
        
        def __str__(self):
            return self.email
    
    # settings.py
    AUTH_USER_MODEL = 'myapp.CustomUser'
    
    # Custom user creation form
    from django.contrib.auth.forms import UserCreationForm
    
    class CustomUserCreationForm(UserCreationForm):
        email = forms.EmailField(required=True)
        phone = forms.CharField(max_length=20, required=False)
        
        class Meta:
            model = CustomUser
            fields = ('username', 'email', 'phone', 'password1', 'password2')
        
        def save(self, commit=True):
            user = super().save(commit=False)
            user.email = self.cleaned_data['email']
            user.phone = self.cleaned_data['phone']
            if commit:
                user.save()
            return user
    ```

61. **Permission system** - Use Django's permission system for fine-grained access control.

    ```python
    # models.py
    from django.contrib.auth.models import Permission
    from django.contrib.contenttypes.models import ContentType
    
    class Product(models.Model):
        name = models.CharField(max_length=200)
        price = models.DecimalField(max_digits=10, decimal_places=2)
        created_by = models.ForeignKey(User, on_delete=models.CASCADE)
        
        class Meta:
            permissions = [
                ('can_approve_product', 'Can approve product'),
                ('can_feature_product', 'Can feature product'),
                ('can_bulk_edit', 'Can bulk edit products'),
            ]
    
    # views.py
    from django.contrib.auth.decorators import permission_required
    from django.contrib.auth.mixins import PermissionRequiredMixin
    
    @permission_required('myapp.can_approve_product')
    def approve_product(request, pk):
        product = get_object_or_404(Product, pk=pk)
        product.is_approved = True
        product.save()
        messages.success(request, 'Product approved successfully!')
        return redirect('product_detail', pk=pk)
    
    class ProductApprovalView(PermissionRequiredMixin, UpdateView):
        model = Product
        permission_required = 'myapp.can_approve_product'
        fields = ['is_approved']
        
        def handle_no_permission(self):
            messages.error(self.request, 'You do not have permission to approve products.')
            return redirect('product_list')
    
    # Programmatic permission handling
    def check_product_permissions(request, product):
        user = request.user
        
        # Check if user can edit this product
        if user.has_perm('myapp.change_product') or product.created_by == user:
            return True
        
        # Check custom permissions
        if user.has_perm('myapp.can_approve_product'):
            return True
        
        return False
    ```

62. **Groups and roles** - Organize users into groups with specific permissions.

    ```python
    # utils.py - Group management utilities
    from django.contrib.auth.models import Group, Permission
    from django.contrib.contenttypes.models import ContentType
    
    def create_user_groups():
        """Create default user groups with permissions"""
        
        # Create groups
        admin_group, _ = Group.objects.get_or_create(name='Product Managers')
        editor_group, _ = Group.objects.get_or_create(name='Product Editors')
        viewer_group, _ = Group.objects.get_or_create(name='Product Viewers')
        
        # Get content type
        product_ct = ContentType.objects.get_for_model(Product)
        
        # Product Manager permissions
        admin_perms = Permission.objects.filter(
            content_type=product_ct,
            codename__in=['add_product', 'change_product', 'delete_product', 'can_approve_product']
        )
        admin_group.permissions.set(admin_perms)
        
        # Product Editor permissions
        editor_perms = Permission.objects.filter(
            content_type=product_ct,
            codename__in=['add_product', 'change_product']
        )
        editor_group.permissions.set(editor_perms)
        
        # Product Viewer permissions
        viewer_perms = Permission.objects.filter(
            content_type=product_ct,
            codename='view_product'
        )
        viewer_group.permissions.set(viewer_perms)
    
    # views.py - Group-based access control
    from django.contrib.auth.decorators import user_passes_test
    
    def is_product_manager(user):
        return user.groups.filter(name='Product Managers').exists()
    
    @user_passes_test(is_product_manager)
    def product_manager_dashboard(request):
        products = Product.objects.all()
        return render(request, 'products/manager_dashboard.html', {'products': products})
    
    # Class-based view with group checking
    class ProductManagerView(UserPassesTestMixin, ListView):
        model = Product
        template_name = 'products/manager_list.html'
        
        def test_func(self):
            return self.request.user.groups.filter(name='Product Managers').exists()
        
        def handle_no_permission(self):
            messages.error(self.request, 'You must be a Product Manager to access this page.')
            return redirect('product_list')
    ```

# Django Testing - Complete Guide

## Test Types and Setup

63. **Django testing basics** - Write tests to ensure your application works correctly and catch bugs early.

    ```python
    # tests.py
    from django.test import TestCase, Client
    from django.contrib.auth.models import User
    from django.urls import reverse
    from .models import Product, Category
    
    class ProductModelTest(TestCase):
        def setUp(self):
            """Set up test data"""
            self.user = User.objects.create_user(
                username='testuser',
                password='testpass123'
            )
            self.category = Category.objects.create(
                name='Electronics',
                description='Electronic products'
            )
            self.product = Product.objects.create(
                name='Test Product',
                description='A test product',
                price=99.99,
                category=self.category,
                created_by=self.user
            )
        
        def test_product_creation(self):
            """Test product is created correctly"""
            self.assertEqual(self.product.name, 'Test Product')
            self.assertEqual(self.product.price, 99.99)
            self.assertEqual(self.product.category, self.category)
            self.assertEqual(self.product.created_by, self.user)
        
        def test_product_str_method(self):
            """Test product string representation"""
            self.assertEqual(str(self.product), 'Test Product')
        
        def test_product_profit_calculation(self):
            """Test custom product methods"""
            self.product.cost = 50.00
            self.product.save()
            
            expected_profit = self.product.price - self.product.cost
            self.assertEqual(self.product.profit, expected_profit)
        
        def tearDown(self):
            """Clean up after tests"""
            # Django automatically handles database cleanup
            pass
    ```

64. **View testing** - Test your views to ensure they respond correctly and handle different scenarios.

    ```python
    from django.test import TestCase, Client
    from django.contrib.auth.models import User
    from django.urls import reverse
    from django.http import Http404
    
    class ProductViewTest(TestCase):
        def setUp(self):
            self.client = Client()
            self.user = User.objects.create_user(
                username='testuser',
                password='testpass123'
            )
            self.category = Category.objects.create(name='Electronics')
            self.product = Product.objects.create(
                name='Test Product',
                price=99.99,
                category=self.category,
                created_by=self.user
            )
        
        def test_product_list_view(self):
            """Test product list view"""
            response = self.client.get(reverse('product_list'))
            
            self.assertEqual(response.status_code, 200)
            self.assertContains(response, 'Test Product')
            self.assertContains(response, '99.99')
        
        def test_product_detail_view(self):
            """Test product detail view"""
            response = self.client.get(
                reverse('product_detail', kwargs={'pk': self.product.pk})
            )
            
            self.assertEqual(response.status_code, 200)
            self.assertEqual(response.context['product'], self.product)
        
        def test_product_create_view_anonymous(self):
            """Test product creation requires authentication"""
            response = self.client.post(reverse('product_create'), {
                'name': 'New Product',
                'price': 149.99,
                'category': self.category.pk
            })
            
            # Should redirect to login
            self.assertEqual(response.status_code, 302)
        
        def test_product_create_view_authenticated(self):
            """Test product creation when authenticated"""
            self.client.login(username='testuser', password='testpass123')
            
            response = self.client.post(reverse('product_create'), {
                'name': 'New Product',
                'price': 149.99,
                'category': self.category.pk
            })
            
            self.assertEqual(response.status_code, 302)  # Redirect after creation
            self.assertTrue(Product.objects.filter(name='New Product').exists())
        
        def test_product_update_view_owner(self):
            """Test product update by owner"""
            self.client.login(username='testuser', password='testpass123')
            
            response = self.client.post(
                reverse('product_update', kwargs={'pk': self.product.pk}),
                {
                    'name': 'Updated Product',
                    'price': 199.99,
                    'category': self.category.pk
                }
            )
            
            self.assertEqual(response.status_code, 302)
            self.product.refresh_from_db()
            self.assertEqual(self.product.name, 'Updated Product')
    ```

65. **Form testing** - Test form validation and processing.

    ```python
    from django.test import TestCase
    from .forms import ProductForm
    
    class ProductFormTest(TestCase):
        def setUp(self):
            self.category = Category.objects.create(name='Electronics')
        
        def test_product_form_valid_data(self):
            """Test form with valid data"""
            form_data = {
                'name': 'Test Product',
                'description': 'A test product',
                'price': 99.99,
                'category': self.category.pk
            }
            
            form = ProductForm(data=form_data)
            self.assertTrue(form.is_valid())
        
        def test_product_form_invalid_data(self):
            """Test form with invalid data"""
            form_data = {
                'name': '',  # Required field
                'description': 'A test product',
                'price': -10,  # Invalid price
                'category': self.category.pk
            }
            
            form = ProductForm(data=form_data)
            self.assertFalse(form.is_valid())
            self.assertIn('name', form.errors)
            self.assertIn('price', form.errors)
        
        def test_product_form_clean_name(self):
            """Test custom name validation"""
            form_data = {
                'name': 'ab',  # Too short
                'description': 'A test product',
                'price': 99.99,
                'category': self.category.pk
            }
            
            form = ProductForm(data=form_data)
            self.assertFalse(form.is_valid())
            self.assertIn('Name must be at least 3 characters', str(form.errors))
        
        def test_product_form_save(self):
            """Test form save method"""
            user = User.objects.create_user(username='testuser', password='testpass')
            
            form_data = {
                'name': 'Form Test Product',
                'description': 'Created via form',
                'price': 149.99,
                'category': self.category.pk
            }
            
            form = ProductForm(data=form_data)
            if form.is_valid():
                product = form.save(commit=False)
                product.created_by = user
                product.save()
                
                self.assertEqual(product.name, 'Form Test Product')
                self.assertEqual(product.created_by, user)
    ```

66. **API testing with DRF** - Test your API endpoints and serializers.

    ```python
    from rest_framework.test import APITestCase
    from rest_framework import status
    from django.contrib.auth.models import User
    from django.urls import reverse
    from .models import Product, Category
    
    class ProductAPITest(APITestCase):
        def setUp(self):
            self.user = User.objects.create_user(
                username='testuser',
                password='testpass123'
            )
            self.category = Category.objects.create(name='Electronics')
            self.product = Product.objects.create(
                name='Test Product',
                price=99.99,
                category=self.category,
                created_by=self.user
            )
        
        def test_get_product_list(self):
            """Test GET /api/products/"""
            url = reverse('product-list')
            response = self.client.get(url)
            
            self.assertEqual(response.status_code, status.HTTP_200_OK)
            self.assertEqual(len(response.data), 1)
            self.assertEqual(response.data[0]['name'], 'Test Product')
        
        def test_get_product_detail(self):
            """Test GET /api/products/{id}/"""
            url = reverse('product-detail', kwargs={'pk': self.product.pk})
            response = self.client.get(url)
            
            self.assertEqual(response.status_code, status.HTTP_200_OK)
            self.assertEqual(response.data['name'], 'Test Product')
        
        def test_create_product_authenticated(self):
            """Test POST /api/products/ with authentication"""
            self.client.force_authenticate(user=self.user)
            
            data = {
                'name': 'New API Product',
                'description': 'Created via API',
                'price': 199.99,
                'category': self.category.pk
            }
            
            url = reverse('product-list')
            response = self.client.post(url, data)
            
            self.assertEqual(response.status_code, status.HTTP_201_CREATED)
            self.assertEqual(response.data['name'], 'New API Product')
            self.assertTrue(Product.objects.filter(name='New API Product').exists())
        
        def test_create_product_unauthenticated(self):
            """Test POST /api/products/ without authentication"""
            data = {
                'name': 'New API Product',
                'price': 199.99,
                'category': self.category.pk
            }
            
            url = reverse('product-list')
            response = self.client.post(url, data)
            
            self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
        
        def test_update_product_owner(self):
            """Test PUT /api/products/{id}/ by owner"""
            self.client.force_authenticate(user=self.user)
            
            data = {
                'name': 'Updated API Product',
                'description': 'Updated via API',
                'price': 299.99,
                'category': self.category.pk
            }
            
            url = reverse('product-detail', kwargs={'pk': self.product.pk})
            response = self.client.put(url, data)
            
            self.assertEqual(response.status_code, status.HTTP_200_OK)
            self.product.refresh_from_db()
            self.assertEqual(self.product.name, 'Updated API Product')
    ```

67. **Advanced testing techniques** - Mocking, fixtures, and test utilities.

    ```python
    from django.test import TestCase, override_settings
    from django.test.utils import override_settings
    from unittest.mock import patch, Mock
    import tempfile
    import os
    
    class AdvancedTestingExample(TestCase):
        def setUp(self):
            self.temp_dir = tempfile.mkdtemp()
        
        def tearDown(self):
            # Clean up temp files
            import shutil
            shutil.rmtree(self.temp_dir)
        
        @override_settings(MEDIA_ROOT=tempfile.mkdtemp())
        def test_file_upload(self):
            """Test file upload functionality"""
            from django.core.files.uploadedfile import SimpleUploadedFile
            
            # Create a fake file
            fake_file = SimpleUploadedFile(
                "test.txt",
                b"file_content",
                content_type="text/plain"
            )
            
            # Test file upload
            response = self.client.post('/upload/', {'file': fake_file})
            self.assertEqual(response.status_code, 200)
        
        @patch('requests.get')
        def test_external_api_call(self, mock_get):
            """Test external API call with mocking"""
            # Mock the external API response
            mock_response = Mock()
            mock_response.json.return_value = {'price': 100.00}
            mock_response.status_code = 200
            mock_get.return_value = mock_response
            
            # Test your function that makes the API call
            from .utils import get_external_price
            price = get_external_price('TEST_PRODUCT')
            
            self.assertEqual(price, 100.00)
            mock_get.assert_called_once_with('https://api.example.com/price/TEST_PRODUCT')
        
        def test_database_queries(self):
            """Test database query optimization"""
            from django.test.utils import override_settings
            from django.db import connection
            
            # Create test data
            category = Category.objects.create(name='Test Category')
            for i in range(10):
                Product.objects.create(
                    name=f'Product {i}',
                    category=category,
                    price=100.00,
                    created_by=User.objects.create_user(f'user{i}', 'pass')
                )
            
            # Test query count
            with self.assertNumQueries(1):
                # This should use select_related to avoid N+1 queries
                products = Product.objects.select_related('category').all()
                for product in products:
                    print(product.category.name)  # Should not hit database
        
        def test_custom_management_command(self):
            """Test custom management command"""
            from django.core.management import call_command
            from io import StringIO
            
            out = StringIO()
            call_command('your_custom_command', stdout=out)
            
            self.assertIn('Expected output', out.getvalue())
    ```

# Django Signals - Complete Guide

## Built-in Signals

68. **Django signals basics** - Signals allow decoupled applications to get notified when certain actions occur.

    ```python
    # signals.py
    from django.db.models.signals import post_save, pre_save, post_delete
    from django.dispatch import receiver
    from django.contrib.auth.models import User
    from .models import Product, UserProfile
    
    @receiver(post_save, sender=User)
    def create_user_profile(sender, instance, created, **kwargs):
        """Create user profile when user is created"""
        if created:
            UserProfile.objects.create(user=instance)
    
    @receiver(post_save, sender=User)
    def save_user_profile(sender, instance, **kwargs):
        """Save user profile when user is saved"""
        if hasattr(instance, 'profile'):
            instance.profile.save()
    
    @receiver(pre_save, sender=Product)
    def product_pre_save(sender, instance, **kwargs):
        """Actions before product is saved"""
        # Generate slug if not provided
        if not instance.slug and instance.name:
            from django.utils.text import slugify
            instance.slug = slugify(instance.name)
        
        # Validate price
        if instance.price <= 0:
            raise ValueError("Price must be positive")
    
    @receiver(post_save, sender=Product)
    def product_post_save(sender, instance, created, **kwargs):
        """Actions after product is saved"""
        if created:
            # Send welcome email to product owner
            from django.core.mail import send_mail
            send_mail(
                'Product Created',
                f'Your product "{instance.name}" has been created successfully.',
                'from@example.com',
                [instance.created_by.email],
                fail_silently=False,
            )
        
        # Clear cache
        from django.core.cache import cache
        cache.delete('product_list')
    
    @receiver(post_delete, sender=Product)
    def product_post_delete(sender, instance, **kwargs):
        """Actions after product is deleted"""
        # Delete associated files
        if instance.image:
            instance.image.delete(save=False)
        
        # Clear cache
        from django.core.cache import cache
        cache.delete('product_list')
    ```

69. **Custom signals** - Create your own signals for application-specific events.

    ```python
    # signals.py
    import django.dispatch
    
    # Create custom signals
    product_viewed = django.dispatch.Signal()
    product_purchased = django.dispatch.Signal()
    user_achievement_unlocked = django.dispatch.Signal()
    
    # Example of sending custom signals
    def track_product_view(request, product):
        """Track when a product is viewed"""
        product_viewed.send(
            sender=product.__class__,
            product=product,
            user=request.user,
            ip_address=request.META.get('REMOTE_ADDR'),
            user_agent=request.META.get('HTTP_USER_AGENT')
        )
    
    def process_purchase(user, product, quantity):
        """Process a purchase and send signal"""
        # Purchase logic here
        order = Order.objects.create(
            user=user,
            product=product,
            quantity=quantity,
            total_price=product.price * quantity
        )
        
        # Send purchase signal
        product_purchased.send(
            sender=Order,
            order=order,
            user=user,
            product=product,
            quantity=quantity
        )
        
        return order
    
    # Signal receivers for custom signals
    @receiver(product_viewed)
    def handle_product_view(sender, product, user, **kwargs):
        """Handle product view event"""
        # Update view count
        product.view_count += 1
        product.save(update_fields=['view_count'])
        
        # Track user behavior
        if user.is_authenticated:
            ProductView.objects.create(
                product=product,
                user=user,
                ip_address=kwargs.get('ip_address'),
                user_agent=kwargs.get('user_agent')
            )
    
    @receiver(product_purchased)
    def handle_product_purchase(sender, order, user, product, quantity, **kwargs):
        """Handle product purchase event"""
        # Update product sales count
        product.sales_count += quantity
        product.save(update_fields=['sales_count'])
        
        # Check for achievements
        total_purchases = Order.objects.filter(user=user).count()
        if total_purchases == 1:
            user_achievement_unlocked.send(
                sender=user.__class__,
                user=user,
                achievement='first_purchase'
            )
    
    @receiver(user_achievement_unlocked)
    def handle_user_achievement(sender, user, achievement, **kwargs):
        """Handle user achievement unlocked"""
        # Create achievement record
        UserAchievement.objects.create(
            user=user,
            achievement=achievement,
            unlocked_at=timezone.now()
        )
        
        # Send congratulations email
        send_achievement_email(user, achievement)
    ```

70. **Signal connection and disconnection** - Programmatically connect and disconnect signals.

    ```python
    # utils.py
    from django.db.models.signals import post_save
    from django.dispatch import receiver
    
    # Manual signal connection
    def connect_signals():
        """Connect signals manually"""
        post_save.connect(my_callback, sender=Product)
    
    def disconnect_signals():
        """Disconnect signals manually"""
        post_save.disconnect(my_callback, sender=Product)
    
    def my_callback(sender, instance, created, **kwargs):
        """Signal callback function"""
        if created:
            print(f"New product created: {instance.name}")
    
    # Context manager for temporary signal disconnection
    from contextlib import contextmanager
    
    @contextmanager
    def disable_signals():
        """Context manager to temporarily disable signals"""
        post_save.disconnect(my_callback, sender=Product)
        try:
            yield
        finally:
            post_save.connect(my_callback, sender=Product)
    
    # Usage
    with disable_signals():
        # Signals are disabled in this block
        Product.objects.create(name="Test Product")
    
    # Conditional signal handling
    @receiver(post_save, sender=Product)
    def conditional_signal_handler(sender, instance, **kwargs):
        """Signal handler that runs conditionally"""
        # Skip signal processing in tests
        if hasattr(settings, 'TESTING') and settings.TESTING:
            return
        
        # Skip for certain users
        if instance.created_by.username == 'system':
            return
        
        # Your signal logic here
        process_product_created(instance)
    ```

# Django Security - Complete Guide

## Security Best Practices

71. **CSRF protection** - Protect against Cross-Site Request Forgery attacks.

    ```python
    # settings.py
    MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.middleware.common.CommonMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',  # CSRF protection
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
        'django.middleware.clickjacking.XFrameOptionsMiddleware',
    ]
    
    # CSRF settings
    CSRF_COOKIE_SECURE = True  # Only send over HTTPS
    CSRF_COOKIE_HTTPONLY = True  # Prevent JavaScript access
    CSRF_COOKIE_SAMESITE = 'Strict'  # Prevent cross-site requests
    CSRF_TRUSTED_ORIGINS = ['https://yourdomain.com']  # Trusted domains
    ```

    ```html
    <!-- In templates - include CSRF token -->
    <form method="post">
        {% csrf_token %}
        <input type="text" name="username">
        <input type="password" name="password">
        <button type="submit">Login</button>
    </form>
    ```

    ```python
    # views.py - CSRF exemption (use carefully)
    from django.views.decorators.csrf import csrf_exempt
    from django.utils.decorators import method_decorator
    
    @csrf_exempt
    def api_endpoint(request):
        """API endpoint that doesn't require CSRF token"""
        # Only use for APIs that have other authentication
        pass
    
    @method_decorator(csrf_exempt, name='dispatch')
    class APIView(View):
        """Class-based view with CSRF exemption"""
        pass
    ```

72. **XSS protection** - Prevent Cross-Site Scripting attacks.

    ```python
    # settings.py
    SECURE_BROWSER_XSS_FILTER = True  # Enable XSS filtering
    SECURE_CONTENT_TYPE_NOSNIFF = True  # Prevent MIME type sniffing
    
    # Template security
    from django.utils.safestring import mark_safe
    from django.utils.html import escape, format_html
    
    def safe_template_rendering(request):
        user_input = request.GET.get('message', '')
        
        # BAD - vulnerable to XSS
        # dangerous_html = f"<p>{user_input}</p>"
        
        # GOOD - escape user input
        safe_html = format_html("<p>{}</p>", user_input)
        
        # Or use escape function
        escaped_input = escape(user_input)
        
        return render(request, 'template.html', {
            'safe_content': safe_html,
            'escaped_content': escaped_input
        })
    ```

    ```html
    <!-- In templates - Django auto-escapes by default -->
    <p>{{ user_input }}</p>  <!-- Automatically escaped -->
    
    <!-- To display raw HTML (be very careful) -->
    <p>{{ trusted_html|safe }}</p>
    
    <!-- Or use autoescape tag -->
    {% autoescape off %}
        {{ trusted_html }}
    {% endautoescape %}
    ```

73. **SQL injection prevention** - Use Django ORM properly to prevent SQL injection.

    ```python
    # GOOD - Using Django ORM (safe)
    def get_products_safe(request):
        category = request.GET.get('category')
        
        # Django ORM automatically escapes parameters
        products = Product.objects.filter(category__name=category)
        
        # Using Q objects (also safe)
        from django.db.models import Q
        products = Product.objects.filter(Q(category__name=category))
        
        return render(request, 'products.html', {'products': products})
    
    # BAD - Raw SQL without proper escaping (vulnerable)
    def get_products_unsafe(request):
        category = request.GET.get('category')
        
        # NEVER DO THIS - vulnerable to SQL injection
        # products = Product.objects.raw(
        #     f"SELECT * FROM products WHERE category = '{category}'"
        # )
        
        # GOOD - Raw SQL with proper parameterization
        products = Product.objects.raw(
            "SELECT * FROM products WHERE category = %s", [category]
        )
        
        return render(request, 'products.html', {'products': products})
    
    # Using extra() safely
    def get_products_with_extra(request):
        min_price = request.GET.get('min_price', 0)
        
        # Safe use of extra()
        products = Product.objects.extra(
            where=["price >= %s"],
            params=[min_price]
        )
        
        return render(request, 'products.html', {'products': products})
    ```

74. **Authentication security** - Secure user authentication and session management.

    ```python
    # settings.py
    # Password validation
    AUTH_PASSWORD_VALIDATORS = [
        {
            'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
        },
        {
            'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
            'OPTIONS': {
                'min_length': 8,
            }
        },
        {
            'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
        },
        {
            'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
        },
    ]
    
    # Session security
    SESSION_COOKIE_SECURE = True  # Only send over HTTPS
    SESSION_COOKIE_HTTPONLY = True  # Prevent JavaScript access
    SESSION_COOKIE_AGE = 3600  # 1 hour session timeout
    SESSION_EXPIRE_AT_BROWSER_CLOSE = True  # Expire when browser closes
    
    # Login security
    LOGIN_URL = '/login/'
    LOGIN_REDIRECT_URL = '/dashboard/'
    LOGOUT_REDIRECT_URL = '/'
    
    # views.py - Secure authentication views
    from django.contrib.auth import authenticate, login
    from django.contrib.auth.decorators import login_required
    from django.contrib.auth.forms import AuthenticationForm
    from django.core.cache import cache
    import time
    
    def secure_login(request):
        """Secure login with rate limiting"""
        ip_address = request.META.get('REMOTE_ADDR')
        cache_key = f"login_attempts_{ip_address}"
        
        # Check rate limiting
        attempts = cache.get(cache_key, 0)
        if attempts >= 5:
            return render(request, 'login.html', {
                'error': 'Too many login attempts. Please try again later.'
            })
        
        if request.method == 'POST':
            form = AuthenticationForm(request, data=request.POST)
            if form.is_valid():
                username = form.cleaned_data.get('username')
                password = form.cleaned_data.get('password')
                
                user = authenticate(username=username, password=password)
                if user is not None:
                    login(request, user)
                    # Clear failed attempts on successful login
                    cache.delete(cache_key)
                    return redirect('dashboard')
            
            # Increment failed attempts
            cache.set(cache_key, attempts + 1, 3600)  # 1 hour timeout
        
        else:
            form = AuthenticationForm()
        
        return render(request, 'login.html', {'form': form})
    ```

75. **Data validation and sanitization** - Validate and sanitize all user input.

    ```python
    # forms.py
    from django import forms
    from django.core.exceptions import ValidationError
    import re
    
    class SecureContactForm(forms.Form):
        name = forms.CharField(max_length=100)
        email = forms.EmailField()
        message = forms.CharField(widget=forms.Textarea)
        
        def clean_name(self):
            name = self.cleaned_data.get('name')
            
            # Remove potentially dangerous characters
            if not re.match(r'^[a-zA-Z\s]+$', name):
                raise ValidationError("Name can only contain letters and spaces")
            
            # Check for common injection patterns
            dangerous_patterns = ['<script', 'javascript:', 'onload=', 'onerror=']
            name_lower = name.lower()
            for pattern in dangerous_patterns:
                if pattern in name_lower:
                    raise ValidationError("Invalid characters in name")
            
            return name
        
        def clean_message(self):
            message = self.cleaned_data.get('message')
            
            # Limit message length
            if len(message) > 1000:
                raise ValidationError("Message too long")
            
            # Remove or escape HTML tags
            from django.utils.html import strip_tags
            message = strip_tags(message)
            
            return message
    
    # Custom validators
    def validate_safe_filename(value):
        """Validate uploaded file names"""
        import os
        
        # Get just the filename
        filename = os.path.basename(value)
        
        # Check for dangerous patterns
        dangerous_patterns = ['..', '/', '\\', '<', '>', ':', '"', '|', '?', '*']
        for pattern in dangerous_patterns:
            if pattern in filename:
                raise ValidationError("Invalid characters in filename")
        
        return value
    
    # models.py
    class SecureFileUpload(models.Model):
        name = models.CharField(max_length=100)
        file = models.FileField(
            upload_to='secure_uploads/',
            validators=[validate_safe_filename]
        )
        uploaded_by = models.ForeignKey(User, on_delete=models.CASCADE)
        
        def save(self, *args, **kwargs):
            # Additional security check
            if self.file:
                # Check file size
                if self.file.size > 5 * 1024 * 1024:  # 5MB limit
                    raise ValidationError("File too large")
                
                # Check file type
                allowed_types = ['image/jpeg', 'image/png', 'application/pdf']
                if hasattr(self.file, 'content_type'):
                    if self.file.content_type not in allowed_types:
                        raise ValidationError("File type not allowed")
            
                         super().save(*args, **kwargs)
    ```

# Django Email - Complete Guide

## Email Configuration and Sending

76. **Email backend configuration** - Configure Django to send emails through various backends.

    ```python
    # settings.py - Email configuration
    
    # SMTP Configuration (Gmail example)
    EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
    EMAIL_HOST = 'smtp.gmail.com'
    EMAIL_PORT = 587
    EMAIL_USE_TLS = True
    EMAIL_HOST_USER = 'your-email@gmail.com'
    EMAIL_HOST_PASSWORD = 'your-app-password'  # Use app-specific password
    DEFAULT_FROM_EMAIL = 'your-email@gmail.com'
    
    # Console backend for development
    EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
    
    # File backend for testing
    EMAIL_BACKEND = 'django.core.mail.backends.filebased.EmailBackend'
    EMAIL_FILE_PATH = '/tmp/app-messages'  # Where to store emails
    
    # Dummy backend (sends nothing)
    EMAIL_BACKEND = 'django.core.mail.backends.dummy.EmailBackend'
    ```

77. **Basic email sending** - Send simple and HTML emails programmatically.

    ```python
    # views.py
    from django.core.mail import send_mail, send_mass_mail
    from django.template.loader import render_to_string
    from django.utils.html import strip_tags
    from django.conf import settings
    
    def send_simple_email(request):
        """Send a simple text email"""
        send_mail(
            subject='Subject here',
            message='Here is the message.',
            from_email=settings.DEFAULT_FROM_EMAIL,
            recipient_list=['to@example.com'],
            fail_silently=False,
        )
        return HttpResponse('Email sent successfully!')
    
    def send_html_email(request):
        """Send an HTML email"""
        subject = 'Welcome to our site!'
        from_email = settings.DEFAULT_FROM_EMAIL
        to_email = 'user@example.com'
        
        # Render HTML template
        html_message = render_to_string('emails/welcome.html', {
            'user': request.user,
            'site_name': 'Our Awesome Site'
        })
        
        # Create plain text version
        plain_message = strip_tags(html_message)
        
        send_mail(
            subject=subject,
            message=plain_message,
            from_email=from_email,
            recipient_list=[to_email],
            html_message=html_message,
            fail_silently=False
        )
        
        return HttpResponse('HTML email sent!')
    
    def send_bulk_emails(request):
        """Send multiple emails efficiently"""
        message1 = ('Subject here', 'Message body', 'from@example.com', ['to1@example.com'])
        message2 = ('Another subject', 'Another message', 'from@example.com', ['to2@example.com'])
        
        send_mass_mail((message1, message2), fail_silently=False)
        return HttpResponse('Bulk emails sent!')
    ```

78. **Advanced email features** - Send emails with attachments, custom headers, and multiple recipients.

    ```python
    from django.core.mail import EmailMessage, EmailMultiAlternatives
    from django.core.files.base import ContentFile
    import os
    
    def send_email_with_attachment(request):
        """Send email with file attachment"""
        email = EmailMessage(
            subject='Email with attachment',
            body='Please find the attached file.',
            from_email='from@example.com',
            to=['to@example.com'],
            cc=['cc@example.com'],
            bcc=['bcc@example.com'],
            reply_to=['reply@example.com'],
            headers={'Message-ID': 'foo'},
        )
        
        # Attach file from filesystem
        email.attach_file('/path/to/file.pdf')
        
        # Attach file from memory
        email.attach('filename.txt', 'file content', 'text/plain')
        
        # Attach file from model FileField
        # email.attach(model_instance.file.name, model_instance.file.read())
        
        email.send()
        return HttpResponse('Email with attachment sent!')
    
    def send_multipart_email(request):
        """Send email with both text and HTML versions"""
        subject = 'Multipart Email'
        from_email = 'from@example.com'
        to_email = 'to@example.com'
        
        text_content = 'This is an important message.'
        html_content = '<p>This is an <strong>important</strong> message.</p>'
        
        msg = EmailMultiAlternatives(subject, text_content, from_email, [to_email])
        msg.attach_alternative(html_content, "text/html")
        msg.send()
        
        return HttpResponse('Multipart email sent!')
    ```

79. **Email templates and context** - Create reusable email templates with dynamic content.

    ```html
    <!-- templates/emails/welcome.html -->
    <!DOCTYPE html>
    <html>
    <head>
        <title>Welcome to {{ site_name }}</title>
        <style>
            body { font-family: Arial, sans-serif; }
            .header { background-color: #f0f0f0; padding: 20px; }
            .content { padding: 20px; }
            .footer { background-color: #333; color: white; padding: 10px; text-align: center; }
        </style>
    </head>
    <body>
        <div class="header">
            <h1>Welcome to {{ site_name }}!</h1>
        </div>
        
        <div class="content">
            <p>Hello {{ user.first_name }},</p>
            
            <p>Thank you for joining {{ site_name }}. We're excited to have you on board!</p>
            
            <p>Here are some things you can do to get started:</p>
            <ul>
                <li>Complete your profile</li>
                <li>Explore our features</li>
                <li>Connect with other users</li>
            </ul>
            
            <p>If you have any questions, feel free to contact us.</p>
            
            <p>Best regards,<br>The {{ site_name }} Team</p>
        </div>
        
        <div class="footer">
            <p>&copy; 2023 {{ site_name }}. All rights reserved.</p>
        </div>
    </body>
    </html>
    ```

    ```python
    # utils.py - Email utility functions
    from django.core.mail import EmailMultiAlternatives
    from django.template.loader import render_to_string
    from django.utils.html import strip_tags
    from django.conf import settings
    
    def send_templated_email(template_name, context, subject, to_emails, from_email=None):
        """Send email using template"""
        if not from_email:
            from_email = settings.DEFAULT_FROM_EMAIL
        
        # Render HTML template
        html_content = render_to_string(f'emails/{template_name}.html', context)
        
        # Create text version
        text_content = strip_tags(html_content)
        
        # Send email
        msg = EmailMultiAlternatives(
            subject=subject,
            body=text_content,
            from_email=from_email,
            to=to_emails
        )
        msg.attach_alternative(html_content, "text/html")
        msg.send()
    
    # Usage examples
    def send_welcome_email(user):
        """Send welcome email to new user"""
        send_templated_email(
            template_name='welcome',
            context={'user': user, 'site_name': 'Our Site'},
            subject='Welcome to Our Site!',
            to_emails=[user.email]
        )
    
    def send_password_reset_email(user, reset_url):
        """Send password reset email"""
        send_templated_email(
            template_name='password_reset',
            context={
                'user': user,
                'reset_url': reset_url,
                'site_name': 'Our Site'
            },
            subject='Password Reset Request',
            to_emails=[user.email]
        )
    ```

# Django Middleware - Complete Guide

## Understanding Middleware

80. **Middleware basics** - Middleware is a framework of hooks into Django's request/response processing.

    ```python
    # custom_middleware.py
    from django.utils.deprecation import MiddlewareMixin
    import time
    import logging
    
    class TimingMiddleware(MiddlewareMixin):
        """Middleware to track request processing time"""
        
        def process_request(self, request):
            """Called before Django decides which view to execute"""
            request.start_time = time.time()
            return None  # Continue processing
        
        def process_view(self, request, view_func, view_args, view_kwargs):
            """Called after Django determines which view to execute"""
            print(f"About to execute view: {view_func.__name__}")
            return None  # Continue processing
        
        def process_response(self, request, response):
            """Called after view returns response"""
            if hasattr(request, 'start_time'):
                duration = time.time() - request.start_time
                response['X-Response-Time'] = str(duration)
                print(f"Request took {duration:.2f} seconds")
            return response
        
        def process_exception(self, request, exception):
            """Called when view raises an exception"""
            print(f"Exception occurred: {exception}")
            return None  # Let Django handle the exception
    
    # Function-based middleware (Django 1.10+)
    def timing_middleware(get_response):
        """Function-based middleware for timing requests"""
        
        def middleware(request):
            # Code to be executed for each request before
            # the view (and later middleware) are called
            start_time = time.time()
            
            response = get_response(request)
            
            # Code to be executed for each request/response after
            # the view is called
            duration = time.time() - start_time
            response['X-Response-Time'] = str(duration)
            
            return response
        
        return middleware
    ```

81. **Custom middleware examples** - Practical middleware for common use cases.

    ```python
    # security_middleware.py
    import json
    from django.http import JsonResponse
    from django.core.cache import cache
    
    class RateLimitMiddleware:
        """Rate limiting middleware"""
        
        def __init__(self, get_response):
            self.get_response = get_response
        
        def __call__(self, request):
            # Get client IP
            ip = self.get_client_ip(request)
            
            # Create cache key
            cache_key = f"rate_limit_{ip}"
            
            # Get current request count
            request_count = cache.get(cache_key, 0)
            
            # Check if limit exceeded
            if request_count >= 100:  # 100 requests per hour
                return JsonResponse({
                    'error': 'Rate limit exceeded'
                }, status=429)
            
            # Increment counter
            cache.set(cache_key, request_count + 1, 3600)  # 1 hour
            
            response = self.get_response(request)
            return response
        
        def get_client_ip(self, request):
            x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
            if x_forwarded_for:
                ip = x_forwarded_for.split(',')[0]
            else:
                ip = request.META.get('REMOTE_ADDR')
            return ip
    
    class RequestLoggingMiddleware:
        """Log all requests"""
        
        def __init__(self, get_response):
            self.get_response = get_response
        
        def __call__(self, request):
            # Log request
            print(f"Request: {request.method} {request.get_full_path()}")
            
            if request.user.is_authenticated:
                print(f"User: {request.user.username}")
            
            response = self.get_response(request)
            
            # Log response
            print(f"Response: {response.status_code}")
            
            return response
    
    class CORSMiddleware:
        """Custom CORS middleware"""
        
        def __init__(self, get_response):
            self.get_response = get_response
        
        def __call__(self, request):
            response = self.get_response(request)
            
            # Add CORS headers
            response['Access-Control-Allow-Origin'] = '*'
            response['Access-Control-Allow-Methods'] = 'GET, POST, PUT, DELETE, OPTIONS'
            response['Access-Control-Allow-Headers'] = 'Content-Type, Authorization'
            
            return response
    ```

82. **Middleware configuration and ordering** - Proper middleware setup and understanding execution order.

    ```python
    # settings.py - Middleware configuration
    MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'corsheaders.middleware.CorsMiddleware',  # Third-party CORS
        'django.middleware.common.CommonMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
        'django.middleware.clickjacking.XFrameOptionsMiddleware',
        
        # Custom middleware
        'myapp.middleware.TimingMiddleware',
        'myapp.middleware.RateLimitMiddleware',
        'myapp.middleware.RequestLoggingMiddleware',
    ]
    
    # Middleware execution order:
    # 1. Request flows DOWN the middleware stack
    # 2. View is executed
    # 3. Response flows UP the middleware stack
    
    # For example, with the above order:
    # Request: Security -> Sessions -> CORS -> Common -> CSRF -> Auth -> Messages -> Clickjacking -> Timing -> RateLimit -> Logging -> VIEW
    # Response: VIEW -> Logging -> RateLimit -> Timing -> Clickjacking -> Messages -> Auth -> CSRF -> Common -> CORS -> Sessions -> Security
    ```

# Django Management Commands - Complete Guide

## Creating Custom Commands

83. **Basic management command structure** - Create custom Django management commands for automated tasks.

    ```python
    # management/commands/hello_world.py
    from django.core.management.base import BaseCommand
    from django.contrib.auth.models import User
    
    class Command(BaseCommand):
        help = 'Prints hello world message'
        
        def add_arguments(self, parser):
            """Add command arguments"""
            parser.add_argument(
                '--name',
                type=str,
                help='Name to greet',
                default='World'
            )
            parser.add_argument(
                '--count',
                type=int,
                help='Number of times to greet',
                default=1
            )
            parser.add_argument(
                '--uppercase',
                action='store_true',
                help='Print in uppercase',
            )
        
        def handle(self, *args, **options):
            """Command logic"""
            name = options['name']
            count = options['count']
            uppercase = options['uppercase']
            
            message = f"Hello, {name}!"
            
            if uppercase:
                message = message.upper()
            
            for i in range(count):
                self.stdout.write(
                    self.style.SUCCESS(message)
                )
    
    # Usage: python manage.py hello_world --name=Django --count=3 --uppercase
    ```

84. **Data management commands** - Commands for database operations and data processing.

    ```python
    # management/commands/cleanup_old_data.py
    from django.core.management.base import BaseCommand
    from django.utils import timezone
    from datetime import timedelta
    from myapp.models import Product, Order
    
    class Command(BaseCommand):
        help = 'Clean up old data from the database'
        
        def add_arguments(self, parser):
            parser.add_argument(
                '--days',
                type=int,
                default=30,
                help='Delete data older than X days'
            )
            parser.add_argument(
                '--dry-run',
                action='store_true',
                help='Show what would be deleted without actually deleting'
            )
        
        def handle(self, *args, **options):
            days = options['days']
            dry_run = options['dry_run']
            
            # Calculate cutoff date
            cutoff_date = timezone.now() - timedelta(days=days)
            
            # Find old orders
            old_orders = Order.objects.filter(created_at__lt=cutoff_date)
            old_products = Product.objects.filter(
                created_at__lt=cutoff_date,
                is_active=False
            )
            
            self.stdout.write(f"Found {old_orders.count()} old orders")
            self.stdout.write(f"Found {old_products.count()} old inactive products")
            
            if dry_run:
                self.stdout.write(
                    self.style.WARNING("DRY RUN: No data will be deleted")
                )
                return
            
            # Delete data
            if old_orders.exists():
                deleted_orders = old_orders.delete()
                self.stdout.write(
                    self.style.SUCCESS(f"Deleted {deleted_orders[0]} orders")
                )
            
            if old_products.exists():
                deleted_products = old_products.delete()
                self.stdout.write(
                    self.style.SUCCESS(f"Deleted {deleted_products[0]} products")
                )
            
            self.stdout.write(
                self.style.SUCCESS("Cleanup completed successfully")
            )
    
    # management/commands/import_data.py
    import csv
    from django.core.management.base import BaseCommand, CommandError
    from django.db import transaction
    from myapp.models import Product, Category
    
    class Command(BaseCommand):
        help = 'Import products from CSV file'
        
        def add_arguments(self, parser):
            parser.add_argument('csv_file', type=str, help='Path to CSV file')
            parser.add_argument(
                '--batch-size',
                type=int,
                default=1000,
                help='Batch size for bulk operations'
            )
        
        def handle(self, *args, **options):
            csv_file = options['csv_file']
            batch_size = options['batch_size']
            
            try:
                with open(csv_file, 'r') as file:
                    reader = csv.DictReader(file)
                    products = []
                    
                    for i, row in enumerate(reader, 1):
                        try:
                            # Get or create category
                            category, created = Category.objects.get_or_create(
                                name=row['category']
                            )
                            
                            # Create product
                            product = Product(
                                name=row['name'],
                                description=row.get('description', ''),
                                price=float(row['price']),
                                category=category
                            )
                            products.append(product)
                            
                            # Bulk create in batches
                            if len(products) >= batch_size:
                                Product.objects.bulk_create(products)
                                products = []
                                self.stdout.write(f"Imported {i} products...")
                        
                        except Exception as e:
                            self.stdout.write(
                                self.style.WARNING(f"Error in row {i}: {e}")
                            )
                    
                    # Create remaining products
                    if products:
                        Product.objects.bulk_create(products)
                    
                    self.stdout.write(
                        self.style.SUCCESS(f"Successfully imported {i} products")
                    )
            
            except FileNotFoundError:
                raise CommandError(f"File '{csv_file}' not found")
            except Exception as e:
                raise CommandError(f"Error importing data: {e}")
    ```

85. **Advanced command features** - Commands with progress bars, user input, and error handling.

    ```python
    # management/commands/advanced_example.py
    from django.core.management.base import BaseCommand, CommandError
    from django.contrib.auth.models import User
    import time
    
    class Command(BaseCommand):
        help = 'Advanced command example with progress and user input'
        
        def add_arguments(self, parser):
            parser.add_argument(
                '--interactive',
                action='store_true',
                help='Run in interactive mode'
            )
            parser.add_argument(
                '--verbosity',
                type=int,
                choices=[0, 1, 2, 3],
                default=1,
                help='Verbosity level'
            )
        
        def handle(self, *args, **options):
            interactive = options['interactive']
            verbosity = options['verbosity']
            
            if interactive:
                self.interactive_mode()
            else:
                self.batch_mode(verbosity)
        
        def interactive_mode(self):
            """Interactive mode with user input"""
            self.stdout.write("Running in interactive mode...")
            
            # Get user input
            username = input("Enter username: ")
            email = input("Enter email: ")
            
            # Confirm action
            confirm = input(f"Create user '{username}' with email '{email}'? (y/N): ")
            
            if confirm.lower() == 'y':
                try:
                    user = User.objects.create_user(
                        username=username,
                        email=email,
                        password='temporary_password'
                    )
                    self.stdout.write(
                        self.style.SUCCESS(f"User '{username}' created successfully")
                    )
                except Exception as e:
                    self.stdout.write(
                        self.style.ERROR(f"Error creating user: {e}")
                    )
            else:
                self.stdout.write("Operation cancelled")
        
        def batch_mode(self, verbosity):
            """Batch mode with progress tracking"""
            users = User.objects.all()
            total_users = users.count()
            
            if verbosity >= 1:
                self.stdout.write(f"Processing {total_users} users...")
            
            for i, user in enumerate(users, 1):
                # Simulate processing
                time.sleep(0.1)
                
                # Update user (example operation)
                user.last_login = timezone.now()
                user.save()
                
                # Show progress
                if verbosity >= 2:
                    progress = (i / total_users) * 100
                    self.stdout.write(f"Progress: {progress:.1f}% ({i}/{total_users})")
                elif verbosity >= 1 and i % 10 == 0:
                    self.stdout.write(f"Processed {i}/{total_users} users")
            
            if verbosity >= 1:
                self.stdout.write(
                    self.style.SUCCESS(f"Successfully processed {total_users} users")
                )
    
    # management/commands/send_notifications.py
    from django.core.management.base import BaseCommand
    from django.core.mail import send_mail
    from django.template.loader import render_to_string
    from django.contrib.auth.models import User
    
    class Command(BaseCommand):
        help = 'Send notification emails to users'
        
        def add_arguments(self, parser):
            parser.add_argument(
                '--template',
                type=str,
                required=True,
                help='Email template name'
            )
            parser.add_argument(
                '--subject',
                type=str,
                required=True,
                help='Email subject'
            )
            parser.add_argument(
                '--users',
                nargs='+',
                type=str,
                help='Specific usernames to send to'
            )
        
        def handle(self, *args, **options):
            template = options['template']
            subject = options['subject']
            usernames = options.get('users')
            
            # Get users to send to
            if usernames:
                users = User.objects.filter(username__in=usernames)
            else:
                users = User.objects.filter(is_active=True)
            
            sent_count = 0
            error_count = 0
            
            for user in users:
                try:
                    # Render email content
                    message = render_to_string(f'emails/{template}.txt', {
                        'user': user,
                        'site_name': 'Our Site'
                    })
                    
                    # Send email
                    send_mail(
                        subject=subject,
                        message=message,
                        from_email='noreply@example.com',
                        recipient_list=[user.email],
                        fail_silently=False
                    )
                    
                    sent_count += 1
                    self.stdout.write(f"Sent email to {user.email}")
                
                except Exception as e:
                    error_count += 1
                    self.stdout.write(
                        self.style.WARNING(f"Failed to send email to {user.email}: {e}")
                    )
            
            self.stdout.write(
                self.style.SUCCESS(f"Sent {sent_count} emails, {error_count} errors")
            )
    ```

# Django REST Framework (DRF) Serializers - Complete Guide

## What are Serializers?

86. **Serializers** convert complex Python objects (like Django model instances) into Python native data types (dict, list, etc.) that can then be easily rendered into JSON, XML or other content types. They also work in reverse - converting JSON/form data back into Python objects.

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

# Django REST Framework (DRF) Authentication & Permissions - Complete Guide

## Custom Authentication Classes

87. **Creating custom authentication classes** - Implement your own authentication logic for APIs.

    ```python
    # authentication.py
    from rest_framework import authentication
    from rest_framework import exceptions
    from django.contrib.auth.models import User
    from django.conf import settings
    import jwt
    import redis
    
    class CustomTokenAuthentication(authentication.BaseAuthentication):
        """
        Custom token authentication using JWT tokens
        """
        
        def authenticate(self, request):
            """
            Returns a two-tuple of (user, token) if authentication succeeds,
            or None otherwise.
            """
            auth_header = authentication.get_authorization_header(request).split()
            
            if not auth_header or auth_header[0].lower() != b'bearer':
                return None
            
            if len(auth_header) == 1:
                msg = 'Invalid token header. No credentials provided.'
                raise exceptions.AuthenticationFailed(msg)
            elif len(auth_header) > 2:
                msg = 'Invalid token header. Token string should not contain spaces.'
                raise exceptions.AuthenticationFailed(msg)
            
            try:
                token = auth_header[1].decode('utf-8')
            except UnicodeError:
                msg = 'Invalid token header. Token string should not contain invalid characters.'
                raise exceptions.AuthenticationFailed(msg)
            
            return self.authenticate_credentials(token)
        
        def authenticate_credentials(self, token):
            """
            Authenticate the token and return user
            """
            try:
                payload = jwt.decode(token, settings.SECRET_KEY, algorithms=['HS256'])
            except jwt.ExpiredSignatureError:
                raise exceptions.AuthenticationFailed('Token has expired')
            except jwt.InvalidTokenError:
                raise exceptions.AuthenticationFailed('Invalid token')
            
            try:
                user = User.objects.get(id=payload['user_id'])
            except User.DoesNotExist:
                raise exceptions.AuthenticationFailed('Invalid token')
            
            if not user.is_active:
                raise exceptions.AuthenticationFailed('User inactive or deleted')
            
            return (user, token)
        
        def authenticate_header(self, request):
            """
            Return a string to be used as the value of the WWW-Authenticate
            header in a 401 Unauthenticated response.
            """
            return 'Bearer'
    
    class ApiKeyAuthentication(authentication.BaseAuthentication):
        """
        API Key authentication for machine-to-machine communication
        """
        
        def authenticate(self, request):
            api_key = request.META.get('HTTP_X_API_KEY')
            
            if not api_key:
                return None
            
            try:
                user = User.objects.get(profile__api_key=api_key)
            except User.DoesNotExist:
                raise exceptions.AuthenticationFailed('Invalid API key')
            
            if not user.is_active:
                raise exceptions.AuthenticationFailed('User inactive')
            
            return (user, api_key)
    
    class SessionTokenAuthentication(authentication.BaseAuthentication):
        """
        Hybrid authentication using both session and token
        """
        
        def authenticate(self, request):
            # Try session authentication first
            if hasattr(request, 'user') and request.user.is_authenticated:
                return (request.user, None)
            
            # Fall back to token authentication
            token_auth = CustomTokenAuthentication()
            return token_auth.authenticate(request)
    ```

88. **Advanced authentication patterns** - Implement complex authentication scenarios with caching and validation.

    ```python
    # advanced_authentication.py
    import redis
    import json
    from datetime import datetime, timedelta
    from django.core.cache import cache
    from rest_framework import authentication, exceptions
    from django.contrib.auth.models import User
    
    class RedisTokenAuthentication(authentication.BaseAuthentication):
        """
        Token authentication with Redis storage for fast lookup
        """
        
        def __init__(self):
            self.redis_client = redis.Redis(host='localhost', port=6379, db=1)
        
        def authenticate(self, request):
            token = self.get_token_from_request(request)
            if not token:
                return None
            
            return self.authenticate_credentials(token)
        
        def get_token_from_request(self, request):
            """Extract token from various locations"""
            # Try Authorization header first
            auth_header = authentication.get_authorization_header(request).split()
            if auth_header and auth_header[0].lower() == b'bearer':
                if len(auth_header) == 2:
                    return auth_header[1].decode('utf-8')
            
            # Try query parameter
            token = request.query_params.get('token')
            if token:
                return token
            
            # Try custom header
            token = request.META.get('HTTP_X_AUTH_TOKEN')
            if token:
                return token
            
            return None
        
        def authenticate_credentials(self, token):
            """Validate token against Redis cache"""
            try:
                # Check Redis for token
                user_data = self.redis_client.get(f"auth_token:{token}")
                if not user_data:
                    raise exceptions.AuthenticationFailed('Invalid or expired token')
                
                user_info = json.loads(user_data)
                
                # Check token expiry
                expires_at = datetime.fromisoformat(user_info['expires_at'])
                if datetime.now() > expires_at:
                    self.redis_client.delete(f"auth_token:{token}")
                    raise exceptions.AuthenticationFailed('Token has expired')
                
                # Get user from database
                user = User.objects.get(id=user_info['user_id'])
                
                if not user.is_active:
                    raise exceptions.AuthenticationFailed('User account is disabled')
                
                # Update last activity
                self.redis_client.hset(f"auth_token:{token}", "last_activity", datetime.now().isoformat())
                
                return (user, token)
                
            except User.DoesNotExist:
                raise exceptions.AuthenticationFailed('Invalid token')
            except Exception as e:
                raise exceptions.AuthenticationFailed(f'Authentication failed: {str(e)}')
        
        def create_token(self, user):
            """Create a new token for user"""
            import uuid
            token = str(uuid.uuid4())
            
            expires_at = datetime.now() + timedelta(hours=24)
            
            token_data = {
                'user_id': user.id,
                'username': user.username,
                'created_at': datetime.now().isoformat(),
                'expires_at': expires_at.isoformat(),
                'last_activity': datetime.now().isoformat()
            }
            
            # Store in Redis with expiration
            self.redis_client.setex(
                f"auth_token:{token}",
                timedelta(hours=24),
                json.dumps(token_data)
            )
            
            return token
    
    class TwoFactorAuthentication(authentication.BaseAuthentication):
        """
        Two-factor authentication requiring both password and OTP
        """
        
        def authenticate(self, request):
            username = request.data.get('username')
            password = request.data.get('password')
            otp_code = request.data.get('otp_code')
            
            if not all([username, password, otp_code]):
                return None
            
            # Authenticate with username/password
            user = authenticate(username=username, password=password)
            if not user:
                raise exceptions.AuthenticationFailed('Invalid credentials')
            
            # Verify OTP
            if not self.verify_otp(user, otp_code):
                raise exceptions.AuthenticationFailed('Invalid OTP code')
            
            return (user, None)
        
        def verify_otp(self, user, otp_code):
            """Verify OTP code (implement your OTP logic)"""
            # This is a simplified example
            # In production, use libraries like pyotp
            stored_otp = cache.get(f"otp_{user.id}")
            return stored_otp == otp_code
    ```

## Custom Permission Classes

89. **Basic custom permission classes** - Create permission classes for different access control scenarios.

    ```python
    # permissions.py
    from rest_framework import permissions
    from django.contrib.auth.models import Group
    from datetime import datetime
    
    class IsOwnerOrReadOnly(permissions.BasePermission):
        """
        Custom permission to only allow owners of an object to edit it.
        """
        
        def has_object_permission(self, request, view, obj):
            # Read permissions for any request,
            # so we'll always allow GET, HEAD or OPTIONS requests.
            if request.method in permissions.SAFE_METHODS:
                return True
            
            # Write permissions are only for the owner of the object.
            return obj.owner == request.user
    
    class IsAuthorOrReadOnly(permissions.BasePermission):
        """
        Permission class for blog posts, articles, etc.
        """
        
        def has_permission(self, request, view):
            # Authenticated users can view, only authenticated can create
            if request.method in permissions.SAFE_METHODS:
                return True
            return request.user.is_authenticated
        
        def has_object_permission(self, request, view, obj):
            # Read permissions for any request
            if request.method in permissions.SAFE_METHODS:
                return True
            
            # Write permissions only for the author
            return obj.author == request.user
    
    class IsInGroupPermission(permissions.BasePermission):
        """
        Permission class to check if user belongs to specific group
        """
        required_groups = []
        
        def has_permission(self, request, view):
            if not request.user.is_authenticated:
                return False
            
            # Check if user belongs to any of the required groups
            user_groups = request.user.groups.values_list('name', flat=True)
            return any(group in user_groups for group in self.required_groups)
    
    class IsManagerPermission(IsInGroupPermission):
        """Specific permission for managers"""
        required_groups = ['Managers', 'Administrators']
    
    class IsEditorPermission(IsInGroupPermission):
        """Specific permission for editors"""
        required_groups = ['Editors', 'Managers', 'Administrators']
    
    class TimeBoundPermission(permissions.BasePermission):
        """
        Permission that allows access only during business hours
        """
        
        def has_permission(self, request, view):
            if not request.user.is_authenticated:
                return False
            
            # Allow superusers anytime
            if request.user.is_superuser:
                return True
            
            # Check business hours (9 AM to 6 PM)
            current_hour = datetime.now().hour
            return 9 <= current_hour <= 18
    
    class RateLimitedPermission(permissions.BasePermission):
        """
        Permission that implements rate limiting
        """
        
        def has_permission(self, request, view):
            if not request.user.is_authenticated:
                return False
            
            # Check rate limit
            from django.core.cache import cache
            
            cache_key = f"rate_limit_{request.user.id}"
            current_requests = cache.get(cache_key, 0)
            
            # Allow 100 requests per hour
            if current_requests >= 100:
                return False
            
            # Increment counter
            cache.set(cache_key, current_requests + 1, 3600)
            return True
    ```

90. **Advanced permission patterns** - Complex permission logic with dynamic checks and conditions.

    ```python
    # advanced_permissions.py
    from rest_framework import permissions
    from django.db.models import Q
    from django.conf import settings
    
    class DynamicPermission(permissions.BasePermission):
        """
        Permission that changes based on object state and user attributes
        """
        
        def has_permission(self, request, view):
            return request.user.is_authenticated
        
        def has_object_permission(self, request, view, obj):
            user = request.user
            
            # Superusers can do anything
            if user.is_superuser:
                return True
            
            # Object-specific logic
            if hasattr(obj, 'status'):
                # Draft objects can only be edited by author
                if obj.status == 'draft':
                    return obj.author == user
                
                # Published objects can be edited by author or editors
                elif obj.status == 'published':
                    if request.method in permissions.SAFE_METHODS:
                        return True
                    return (obj.author == user or 
                           user.groups.filter(name='Editors').exists())
                
                # Archived objects are read-only except for admins
                elif obj.status == 'archived':
                    if request.method in permissions.SAFE_METHODS:
                        return True
                    return user.groups.filter(name='Administrators').exists()
            
            return False
    
    class SubscriptionBasedPermission(permissions.BasePermission):
        """
        Permission based on user subscription level
        """
        
        def has_permission(self, request, view):
            if not request.user.is_authenticated:
                return False
            
            # Check if user has active subscription
            if not hasattr(request.user, 'subscription'):
                return False
            
            subscription = request.user.subscription
            
            # Check subscription status
            if not subscription.is_active():
                return False
            
            # Check subscription level for specific actions
            if view.action in ['create', 'update', 'partial_update']:
                return subscription.plan in ['premium', 'enterprise']
            
            # Basic plans can only read
            return subscription.plan in ['basic', 'premium', 'enterprise']
    
    class FieldLevelPermission(permissions.BasePermission):
        """
        Permission class that controls access to specific fields
        """
        
        def has_permission(self, request, view):
            return request.user.is_authenticated
        
        def has_object_permission(self, request, view, obj):
            user = request.user
            
            # Define field access rules
            if request.method in ['PUT', 'PATCH']:
                restricted_fields = self.get_restricted_fields(user, obj)
                
                # Check if user is trying to modify restricted fields
                for field in restricted_fields:
                    if field in request.data:
                        return False
            
            return True
        
        def get_restricted_fields(self, user, obj):
            """Define which fields are restricted for this user"""
            restricted_fields = []
            
            # Non-owners can't change ownership
            if obj.owner != user:
                restricted_fields.extend(['owner', 'created_by'])
            
            # Only admins can change status
            if not user.groups.filter(name='Administrators').exists():
                restricted_fields.append('status')
            
            # Only managers can change sensitive fields
            if not user.groups.filter(name__in=['Managers', 'Administrators']).exists():
                restricted_fields.extend(['price', 'commission_rate'])
            
            return restricted_fields
    
    class IPWhitelistPermission(permissions.BasePermission):
        """
        Permission that restricts access based on IP address
        """
        
        def has_permission(self, request, view):
            # Get client IP
            ip = self.get_client_ip(request)
            
            # Check against whitelist
            whitelist = getattr(settings, 'API_IP_WHITELIST', [])
            
            if whitelist and ip not in whitelist:
                return False
            
            return True
        
        def get_client_ip(self, request):
            x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
            if x_forwarded_for:
                ip = x_forwarded_for.split(',')[0]
            else:
                ip = request.META.get('REMOTE_ADDR')
            return ip
    
    class ConditionalPermission(permissions.BasePermission):
        """
        Permission that applies different rules based on conditions
        """
        
        def has_permission(self, request, view):
            user = request.user
            
            if not user.is_authenticated:
                return False
            
            # Different rules for different view actions
            if view.action == 'list':
                # Anyone can list, but results will be filtered
                return True
            
            elif view.action == 'create':
                # Only premium users can create
                return self.is_premium_user(user)
            
            elif view.action in ['update', 'partial_update', 'destroy']:
                # Will be checked at object level
                return True
            
            return False
        
        def has_object_permission(self, request, view, obj):
            user = request.user
            
            # Owners can always access their objects
            if hasattr(obj, 'owner') and obj.owner == user:
                return True
            
            # Shared objects have different rules
            if hasattr(obj, 'shared_with'):
                if user in obj.shared_with.all():
                    # Shared users can read but not modify
                    return request.method in permissions.SAFE_METHODS
            
            # Team members can access team objects
            if hasattr(obj, 'team') and hasattr(user, 'teams'):
                if obj.team in user.teams.all():
                    return True
            
            return False
        
        def is_premium_user(self, user):
            """Check if user has premium subscription"""
            return (hasattr(user, 'subscription') and 
                   user.subscription.is_active() and 
                   user.subscription.plan in ['premium', 'enterprise'])
    ```

## Understanding Base Class Methods

89. **BaseAuthentication class methods** - Complete reference of methods provided by the authentication base class.

    ```python
    # rest_framework/authentication.py
    from rest_framework import authentication, exceptions
    
    class BaseAuthentication:
        """
        All authentication classes should extend BaseAuthentication.
        """
        
        def authenticate(self, request):
            """
            Authenticate the request and return a two-tuple of (user, token).
            
            This is the MAIN method you must implement in your custom auth class.
            
            Args:
                request: The incoming HTTP request object
                
            Returns:
                - None: If this authentication method doesn't apply to this request
                - (user, token): Two-tuple if authentication succeeds
                - Raises AuthenticationFailed: If authentication fails
                
            Example implementations:
            """
            # Return None if this auth method doesn't apply
            if not self.should_authenticate(request):
                return None
                
            # Extract credentials from request
            credentials = self.get_credentials(request)
            if not credentials:
                return None
                
            # Authenticate and return user, token
            try:
                user = self.authenticate_credentials(credentials)
                return (user, credentials)
            except Exception:
                raise exceptions.AuthenticationFailed('Invalid credentials')
        
        def authenticate_header(self, request):
            """
            Return a string to be used as the value of the `WWW-Authenticate`
            header in a `401 Unauthenticated` response.
            
            This method is OPTIONAL but recommended for proper HTTP compliance.
            
            Args:
                request: The incoming HTTP request object
                
            Returns:
                String: The WWW-Authenticate header value
                None: If no header should be returned
                
            Examples:
            """
            return 'Bearer'  # For token auth
            return 'Basic'   # For basic auth
            return 'Bearer realm="api"'  # With realm
            return None      # No header
        
        def authenticate_credentials(self, credentials):
            """
            Helper method to authenticate extracted credentials.
            
            This method is OPTIONAL - you can implement it separately
            or include the logic in authenticate().
            
            Args:
                credentials: The extracted credentials (token, username/password, etc.)
                
            Returns:
                User object if authentication succeeds
                
            Raises:
                AuthenticationFailed: If credentials are invalid
            """
            # Your credential validation logic here
            pass
    
    # Complete example showing all methods
    class ComprehensiveAuthentication(BaseAuthentication):
        """
        Example showing all BaseAuthentication methods
        """
        
        def authenticate(self, request):
            """
            Main authentication method - REQUIRED
            """
            # Check if we should handle this request
            token = self.get_token_from_request(request)
            if not token:
                return None  # Let other auth methods try
            
            # Validate token and return user
            try:
                user = self.authenticate_credentials(token)
                return (user, token)
            except exceptions.AuthenticationFailed:
                # Re-raise authentication errors
                raise
            except Exception as e:
                # Convert other exceptions to auth failed
                raise exceptions.AuthenticationFailed(f'Authentication error: {str(e)}')
        
        def authenticate_header(self, request):
            """
            WWW-Authenticate header - OPTIONAL but recommended
            """
            return 'Bearer realm="API"'
        
        def authenticate_credentials(self, token):
            """
            Validate token and return user - OPTIONAL helper method
            """
            # Decode/validate token
            try:
                user_id = self.decode_token(token)
                user = User.objects.get(id=user_id, is_active=True)
                return user
            except (User.DoesNotExist, ValueError):
                raise exceptions.AuthenticationFailed('Invalid token')
        
        def get_token_from_request(self, request):
            """
            Extract token from request - CUSTOM helper method
            """
            auth_header = authentication.get_authorization_header(request)
            if auth_header.startswith(b'Bearer '):
                return auth_header[7:].decode('utf-8')
            return None
        
        def decode_token(self, token):
            """
            Decode token to get user ID - CUSTOM helper method
            """
            # Your token decoding logic
            pass
    ```

90. **BasePermission class methods** - Complete reference of methods provided by the permission base class.

    ```python
    # rest_framework/permissions.py
    from rest_framework import permissions
    
    class BasePermission:
        """
        A base class from which all permission classes should inherit.
        """
        
        def has_permission(self, request, view):
            """
            Return `True` if permission is granted for the request, `False` otherwise.
            
            This method is called BEFORE the view is executed and BEFORE
            has_object_permission() is called.
            
            Args:
                request: The incoming HTTP request object
                view: The view being accessed
                
            Returns:
                Boolean: True if permission granted, False otherwise
                
            Use cases:
            - Check if user is authenticated
            - Check user roles/groups
            - Check request method (GET, POST, etc.)
            - Check global permissions
            - Rate limiting
            - IP restrictions
            """
            return True  # Default allows all
        
        def has_object_permission(self, request, view, obj):
            """
            Return `True` if permission is granted for the object, `False` otherwise.
            
            This method is called AFTER has_permission() passes and ONLY
            for detail views (retrieve, update, destroy).
            
            Args:
                request: The incoming HTTP request object
                view: The view being accessed
                obj: The specific object being accessed
                
            Returns:
                Boolean: True if permission granted, False otherwise
                
            Use cases:
            - Check if user owns the object
            - Check object-specific permissions
            - Check object state/status
            - Check user relationship to object
            """
            return True  # Default allows all
    
    # Complete example showing all methods
    class ComprehensivePermission(BasePermission):
        """
        Example showing all BasePermission methods and common patterns
        """
        
        def has_permission(self, request, view):
            """
            Global permission check - ALWAYS called first
            """
            # 1. Check authentication
            if not request.user.is_authenticated:
                return False
            
            # 2. Check HTTP method permissions
            if request.method in permissions.SAFE_METHODS:
                # GET, HEAD, OPTIONS are generally allowed
                return True
            
            # 3. Check view-specific permissions
            if view.action == 'create':
                return self.can_create(request.user)
            elif view.action in ['update', 'partial_update', 'destroy']:
                # Will be checked in has_object_permission
                return True
            elif view.action == 'list':
                return self.can_list(request.user)
            
            # 4. Check user roles/groups
            if request.user.groups.filter(name='Editors').exists():
                return True
            
            # 5. Default fallback
            return False
        
        def has_object_permission(self, request, view, obj):
            """
            Object-level permission check - called AFTER has_permission
            """
            # 1. Superusers can do anything
            if request.user.is_superuser:
                return True
            
            # 2. Check ownership
            if hasattr(obj, 'owner') and obj.owner == request.user:
                return True
            
            # 3. Check read-only access
            if request.method in permissions.SAFE_METHODS:
                return self.can_read_object(request.user, obj)
            
            # 4. Check specific actions
            if view.action == 'update':
                return self.can_update_object(request.user, obj)
            elif view.action == 'destroy':
                return self.can_delete_object(request.user, obj)
            
            # 5. Check object state
            if hasattr(obj, 'status'):
                if obj.status == 'published':
                    return self.can_modify_published(request.user, obj)
                elif obj.status == 'draft':
                    return self.can_modify_draft(request.user, obj)
            
            return False
        
        def can_create(self, user):
            """CUSTOM helper method for creation permission"""
            return user.groups.filter(name__in=['Editors', 'Authors']).exists()
        
        def can_list(self, user):
            """CUSTOM helper method for listing permission"""
            return True  # All authenticated users can list
        
        def can_read_object(self, user, obj):
            """CUSTOM helper method for read permission"""
            # Check if object is public or user has access
            if hasattr(obj, 'is_public') and obj.is_public:
                return True
            return hasattr(obj, 'shared_with') and user in obj.shared_with.all()
        
        def can_update_object(self, user, obj):
            """CUSTOM helper method for update permission"""
            # Only owner or editors can update
            if hasattr(obj, 'owner') and obj.owner == user:
                return True
            return user.groups.filter(name='Editors').exists()
        
        def can_delete_object(self, user, obj):
            """CUSTOM helper method for delete permission"""
            # Only owner or admins can delete
            if hasattr(obj, 'owner') and obj.owner == user:
                return True
            return user.groups.filter(name='Administrators').exists()
        
        def can_modify_published(self, user, obj):
            """CUSTOM helper method for published object modification"""
            # Only editors can modify published content
            return user.groups.filter(name__in=['Editors', 'Administrators']).exists()
        
        def can_modify_draft(self, user, obj):
            """CUSTOM helper method for draft object modification"""
            # Authors can modify their own drafts
            return hasattr(obj, 'author') and obj.author == user
    ```

91. **Method execution flow and best practices** - Understanding when and how methods are called.

    ```python
    # Method execution flow documentation
    
    """
    AUTHENTICATION FLOW:
    ===================
    
    1. Django REST Framework calls each authentication class in order
    2. For each auth class, authenticate() is called
    3. If authenticate() returns None, try next auth class
    4. If authenticate() returns (user, token), authentication succeeds
    5. If authenticate() raises AuthenticationFailed, authentication fails
    6. If no auth class returns a user, request.user = AnonymousUser
    
    PERMISSION FLOW:
    ===============
    
    1. has_permission() is called for ALL requests
    2. If has_permission() returns False, request is denied (403)
    3. If has_permission() returns True, continue to view
    4. For detail views (get_object() is called), has_object_permission() is called
    5. If has_object_permission() returns False, request is denied (403)
    6. If has_object_permission() returns True, request proceeds
    
    MULTIPLE PERMISSION CLASSES:
    ===========================
    
    When multiple permission classes are used, ALL must return True.
    If ANY permission class returns False, the request is denied.
    """
    
    # Example showing method call order
    class DebuggingAuthentication(BaseAuthentication):
        """Authentication class that logs method calls"""
        
        def authenticate(self, request):
            print(f"🔐 authenticate() called for {request.path}")
            
            # Your authentication logic here
            token = request.META.get('HTTP_AUTHORIZATION')
            if not token:
                print("❌ No token found, returning None")
                return None
            
            try:
                user = self.get_user_from_token(token)
                print(f"✅ Authentication successful for user: {user.username}")
                return (user, token)
            except Exception as e:
                print(f"❌ Authentication failed: {e}")
                raise exceptions.AuthenticationFailed('Invalid token')
        
        def authenticate_header(self, request):
            print(f"🔗 authenticate_header() called for {request.path}")
            return 'Bearer'
    
    class DebuggingPermission(BasePermission):
        """Permission class that logs method calls"""
        
        def has_permission(self, request, view):
            print(f"🛡️  has_permission() called for {request.path}")
            print(f"   User: {request.user}")
            print(f"   Method: {request.method}")
            print(f"   View: {view.__class__.__name__}")
            print(f"   Action: {getattr(view, 'action', 'N/A')}")
            
            # Your permission logic here
            result = request.user.is_authenticated
            print(f"   Result: {result}")
            return result
        
        def has_object_permission(self, request, view, obj):
            print(f"🎯 has_object_permission() called for {request.path}")
            print(f"   User: {request.user}")
            print(f"   Object: {obj}")
            print(f"   Object ID: {getattr(obj, 'id', 'N/A')}")
            
            # Your object permission logic here
            result = True  # Your logic here
            print(f"   Result: {result}")
            return result
    
    # Best practices for implementing methods
    class BestPracticeExample:
        """
        AUTHENTICATION BEST PRACTICES:
        =============================
        
        1. authenticate() should:
           - Return None if this auth method doesn't apply
           - Return (user, token) if authentication succeeds
           - Raise AuthenticationFailed if authentication fails
           - Never return False or True
        
        2. authenticate_header() should:
           - Return appropriate WWW-Authenticate header
           - Return None if no header needed
           - Be consistent with your auth scheme
        
        3. Always handle exceptions gracefully:
           - Catch specific exceptions
           - Log security-relevant events
           - Don't expose sensitive information in error messages
        
        PERMISSION BEST PRACTICES:
        =========================
        
        1. has_permission() should:
           - Check global permissions first
           - Return True/False only
           - Be fast (avoid expensive queries)
           - Check authentication state
        
        2. has_object_permission() should:
           - Check object-specific permissions
           - Only be called for detail views
           - Assume has_permission() already passed
           - Use select_related/prefetch_related for efficiency
        
        3. Design patterns:
           - Use helper methods for complex logic
           - Cache expensive permission checks
           - Consider using mixins for shared logic
           - Document your permission requirements
        """
        pass
    ```

## Combining Authentication and Permissions

92. **Using custom auth and permissions together** - Practical examples of combining authentication and permission classes.

    ```python
    # views.py
    from rest_framework import viewsets, status
    from rest_framework.decorators import action
    from rest_framework.response import Response
    from .authentication import CustomTokenAuthentication, ApiKeyAuthentication
    from .permissions import IsOwnerOrReadOnly, IsManagerPermission, SubscriptionBasedPermission
    
    class ProductViewSet(viewsets.ModelViewSet):
        """
        Product viewset with custom authentication and permissions
        """
        authentication_classes = [CustomTokenAuthentication, ApiKeyAuthentication]
        permission_classes = [IsOwnerOrReadOnly]
        
        def get_permissions(self):
            """
            Instantiate and return the list of permissions required for this view.
            """
            if self.action == 'list':
                permission_classes = [permissions.IsAuthenticated]
            elif self.action == 'create':
                permission_classes = [SubscriptionBasedPermission]
            elif self.action in ['update', 'partial_update', 'destroy']:
                permission_classes = [IsOwnerOrReadOnly]
            elif self.action == 'set_featured':
                permission_classes = [IsManagerPermission]
            else:
                permission_classes = [permissions.IsAuthenticated]
            
            return [permission() for permission in permission_classes]
        
        def get_authenticators(self):
            """
            Return different authentication for different actions
            """
            if self.action == 'webhook':
                # Webhooks use API key only
                return [ApiKeyAuthentication()]
            else:
                # Regular API uses token auth
                return [CustomTokenAuthentication()]
        
        @action(detail=True, methods=['post'])
        def set_featured(self, request, pk=None):
            """Only managers can feature products"""
            product = self.get_object()
            product.is_featured = True
            product.save()
            return Response({'status': 'featured'})
        
        @action(detail=False, methods=['post'])
        def webhook(self, request):
            """Webhook endpoint for external systems"""
            # Process webhook data
            return Response({'status': 'processed'})
    
    # Custom permission combinations
    class MultiplePermissionViewSet(viewsets.ModelViewSet):
        """
        ViewSet that uses multiple permission classes
        """
        
        def get_permissions(self):
            """
            Apply multiple permissions that all must pass
            """
            base_permissions = [permissions.IsAuthenticated]
            
            if self.action == 'create':
                # User must be authenticated AND have subscription AND be in group
                base_permissions.extend([
                    SubscriptionBasedPermission(),
                    IsEditorPermission(),
                    TimeBoundPermission()
                ])
            elif self.action in ['update', 'partial_update']:
                base_permissions.extend([
                    IsOwnerOrReadOnly(),
                    DynamicPermission()
                ])
            
            return base_permissions
    ```

93. **Complete authentication and permission setup** - Full implementation example with login, logout, and protected endpoints.

    ```python
    # auth_views.py
    from rest_framework import status
    from rest_framework.decorators import api_view, permission_classes
    from rest_framework.permissions import AllowAny
    from rest_framework.response import Response
    from django.contrib.auth import authenticate
    from .authentication import RedisTokenAuthentication
    
    @api_view(['POST'])
    @permission_classes([AllowAny])
    def login(request):
        """
        Login endpoint that returns custom token
        """
        username = request.data.get('username')
        password = request.data.get('password')
        
        if not username or not password:
            return Response({
                'error': 'Username and password required'
            }, status=status.HTTP_400_BAD_REQUEST)
        
        user = authenticate(username=username, password=password)
        if not user:
            return Response({
                'error': 'Invalid credentials'
            }, status=status.HTTP_401_UNAUTHORIZED)
        
        if not user.is_active:
            return Response({
                'error': 'Account is disabled'
            }, status=status.HTTP_401_UNAUTHORIZED)
        
        # Create custom token
        auth = RedisTokenAuthentication()
        token = auth.create_token(user)
        
        return Response({
            'token': token,
            'user': {
                'id': user.id,
                'username': user.username,
                'email': user.email,
                'groups': list(user.groups.values_list('name', flat=True))
            }
        })
    
    @api_view(['POST'])
    def logout(request):
        """
        Logout endpoint that invalidates token
        """
        auth = RedisTokenAuthentication()
        token = auth.get_token_from_request(request)
        
        if token:
            # Delete token from Redis
            auth.redis_client.delete(f"auth_token:{token}")
        
        return Response({'message': 'Logged out successfully'})
    
    @api_view(['GET'])
    def profile(request):
        """
        Protected endpoint that returns user profile
        """
        return Response({
            'user': {
                'id': request.user.id,
                'username': request.user.username,
                'email': request.user.email,
                'is_staff': request.user.is_staff,
                'groups': list(request.user.groups.values_list('name', flat=True)),
                'permissions': list(request.user.user_permissions.values_list('codename', flat=True))
            }
        })
    
    # settings.py configuration
    REST_FRAMEWORK = {
        'DEFAULT_AUTHENTICATION_CLASSES': [
            'myapp.authentication.CustomTokenAuthentication',
            'rest_framework.authentication.SessionAuthentication',
        ],
        'DEFAULT_PERMISSION_CLASSES': [
            'rest_framework.permissions.IsAuthenticated',
        ],
        'DEFAULT_RENDERER_CLASSES': [
            'rest_framework.renderers.JSONRenderer',
        ],
        'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
        'PAGE_SIZE': 20
    }
    
    # urls.py
    from django.urls import path, include
    from rest_framework.routers import DefaultRouter
    from . import auth_views, views
    
    router = DefaultRouter()
    router.register(r'products', views.ProductViewSet)
    
    urlpatterns = [
        path('api/auth/login/', auth_views.login, name='api_login'),
        path('api/auth/logout/', auth_views.logout, name='api_logout'),
        path('api/auth/profile/', auth_views.profile, name='api_profile'),
        path('api/', include(router.urls)),
    ]
    ```

## Best Practices

94. **Security best practices** - Essential security considerations for custom authentication and permissions.

    ```python
    # security_utils.py
    import hashlib
    import hmac
    import secrets
    from django.conf import settings
    from django.core.cache import cache
    
    class SecurityMixin:
        """Mixin for adding security features to auth classes"""
        
        def is_request_secure(self, request):
            """Check if request is over HTTPS in production"""
            if settings.DEBUG:
                return True
            return request.is_secure()
        
        def log_authentication_attempt(self, request, success=True, user=None):
            """Log authentication attempts for security monitoring"""
            import logging
            logger = logging.getLogger('security')
            
            ip_address = self.get_client_ip(request)
            user_agent = request.META.get('HTTP_USER_AGENT', '')
            
            log_data = {
                'ip_address': ip_address,
                'user_agent': user_agent,
                'success': success,
                'timestamp': timezone.now().isoformat()
            }
            
            if user:
                log_data['user_id'] = user.id
                log_data['username'] = user.username
            
            if success:
                logger.info(f"Authentication successful: {log_data}")
            else:
                logger.warning(f"Authentication failed: {log_data}")
        
        def check_brute_force_protection(self, request, identifier):
            """Implement brute force protection"""
            cache_key = f"failed_attempts_{identifier}"
            failed_attempts = cache.get(cache_key, 0)
            
            if failed_attempts >= 5:  # Max 5 attempts
                return False
            
            return True
        
        def record_failed_attempt(self, identifier):
            """Record failed authentication attempt"""
            cache_key = f"failed_attempts_{identifier}"
            failed_attempts = cache.get(cache_key, 0)
            cache.set(cache_key, failed_attempts + 1, 3600)  # 1 hour lockout
        
        def clear_failed_attempts(self, identifier):
            """Clear failed attempts on successful auth"""
            cache_key = f"failed_attempts_{identifier}"
            cache.delete(cache_key)
    
    # Secure token generation
    def generate_secure_token():
        """Generate cryptographically secure token"""
        return secrets.token_urlsafe(32)
    
    def hash_token(token):
        """Hash token for storage"""
        return hashlib.sha256(token.encode()).hexdigest()
    
    # Secure permission checking
    class SecurePermissionMixin:
        """Mixin for adding security to permission classes"""
        
        def has_permission(self, request, view):
            # Always check HTTPS in production
            if not self.is_secure_request(request):
                return False
            
            # Check rate limiting
            if not self.check_rate_limit(request):
                return False
            
            return super().has_permission(request, view)
        
        def is_secure_request(self, request):
            """Ensure request is secure"""
            if settings.DEBUG:
                return True
            
            if not request.is_secure():
                return False
            
            # Check for secure headers
            if request.META.get('HTTP_X_FORWARDED_PROTO') != 'https':
                return False
            
            return True
        
        def check_rate_limit(self, request):
            """Check rate limiting per user/IP"""
            if request.user.is_authenticated:
                identifier = f"user_{request.user.id}"
            else:
                identifier = f"ip_{self.get_client_ip(request)}"
            
            cache_key = f"rate_limit_{identifier}"
            current_requests = cache.get(cache_key, 0)
            
            if current_requests >= 1000:  # 1000 requests per hour
                return False
            
            cache.set(cache_key, current_requests + 1, 3600)
            return True
    ```

## Best Practices

95. **Optimize database queries** - Use `select_related()` and `prefetch_related()` to avoid N+1 query problems when using nested serializers.

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

# Django Development vs Production Deployment - Complete Guide

## Development Environment Setup

97. **Starting Django in development** - Local development setup with database configuration.

    ```bash
    # DEVELOPMENT SETUP STEPS
    
    # 1. Create virtual environment
    python -m venv venv
    
    # 2. Activate virtual environment
    # On macOS/Linux:
    source venv/bin/activate
    # On Windows:
    # venv\Scripts\activate
    
    # 3. Install dependencies
    pip install -r requirements.txt
    
    # 4. Set up environment variables (create .env file)
    touch .env
    
    # 5. Configure database
    python manage.py migrate
    
    # 6. Create superuser (optional)
    python manage.py createsuperuser
    
    # 7. Collect static files (if using React)
    python manage.py collectstatic --noinput
    
    # 8. Start development server
    python manage.py runserver 8000
    ```

98. **Development environment variables** - Configure settings for local development.

    ```bash
    # .env file for development
    DEBUG=True
    SECRET_KEY=your-dev-secret-key-here
    DATABASE_URL=sqlite:///db.sqlite3
    
    # Or for PostgreSQL in development
    DATABASE_URL=postgresql://username:password@localhost:5432/myproject_dev
    
    # Optional settings
    ALLOWED_HOSTS=localhost,127.0.0.1
    CORS_ALLOWED_ORIGINS=http://localhost:3000
    ```

    ```python
    # settings.py - Environment variable usage
    import os
    from pathlib import Path
    from dotenv import load_dotenv
    
    load_dotenv()  # Load .env file
    
    BASE_DIR = Path(__file__).resolve().parent.parent
    
    # Environment variables
    DEBUG = os.getenv('DEBUG', 'False').lower() == 'true'
    SECRET_KEY = os.getenv('SECRET_KEY', 'fallback-secret-key-for-dev')
    ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', 'localhost').split(',')
    
    # Database configuration
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': BASE_DIR / 'db.sqlite3',
        }
    }
    
    # Override with DATABASE_URL if provided
    import dj_database_url
    if os.getenv('DATABASE_URL'):
        DATABASES['default'] = dj_database_url.parse(os.getenv('DATABASE_URL'))
    ```

99. **Development with PostgreSQL** - Set up PostgreSQL for local development.

    ```bash
    # Install PostgreSQL (macOS with Homebrew)
    brew install postgresql
    brew services start postgresql
    
    # Install PostgreSQL (Ubuntu)
    sudo apt-get install postgresql postgresql-contrib
    sudo systemctl start postgresql
    
    # Create database and user
    sudo -u postgres psql
    
    # In PostgreSQL shell:
    CREATE DATABASE myproject_dev;
    CREATE USER myproject_user WITH PASSWORD 'your_password';
    GRANT ALL PRIVILEGES ON DATABASE myproject_dev TO myproject_user;
    \q
    
    # Install Python PostgreSQL adapter
    pip install psycopg2-binary
    ```

    ```python
    # settings.py - PostgreSQL configuration
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': os.getenv('DB_NAME', 'myproject_dev'),
            'USER': os.getenv('DB_USER', 'myproject_user'),
            'PASSWORD': os.getenv('DB_PASSWORD', 'your_password'),
            'HOST': os.getenv('DB_HOST', 'localhost'),
            'PORT': os.getenv('DB_PORT', '5432'),
        }
    }
    ```

## Production Environment Setup

100. **Production environment variables** - Secure configuration for production deployment.

     ```bash
     # .env file for production (keep this secure!)
     DEBUG=False
     SECRET_KEY=your-super-secure-production-secret-key
     DATABASE_URL=postgresql://user:password@hostname:5432/production_db
     ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com
     
     # Additional production settings
     SECURE_SSL_REDIRECT=True
     SECURE_PROXY_SSL_HEADER=HTTP_X_FORWARDED_PROTO,https
     SECURE_HSTS_SECONDS=31536000
     SECURE_HSTS_INCLUDE_SUBDOMAINS=True
     
     # Email configuration
     EMAIL_HOST=smtp.gmail.com
     EMAIL_PORT=587
     EMAIL_USE_TLS=True
     EMAIL_HOST_USER=your-email@gmail.com
     EMAIL_HOST_PASSWORD=your-app-password
     
     # Cache configuration
     REDIS_URL=redis://localhost:6379/1
     ```

101. **Production settings configuration** - Separate settings for production environment.

     ```python
     # settings/production.py
     from .base import *
     import dj_database_url
     
     # Security settings
     DEBUG = False
     SECRET_KEY = os.environ['SECRET_KEY']
     ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')
     
     # Database
     DATABASES = {
         'default': dj_database_url.parse(os.environ['DATABASE_URL'])
     }
     
     # Static files
     STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
     STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
     
     # Security settings
     SECURE_SSL_REDIRECT = True
     SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
     SECURE_HSTS_SECONDS = 31536000
     SECURE_HSTS_INCLUDE_SUBDOMAINS = True
     SECURE_HSTS_PRELOAD = True
     SECURE_CONTENT_TYPE_NOSNIFF = True
     SECURE_BROWSER_XSS_FILTER = True
     SESSION_COOKIE_SECURE = True
     CSRF_COOKIE_SECURE = True
     
     # Logging
     LOGGING = {
         'version': 1,
         'disable_existing_loggers': False,
         'handlers': {
             'file': {
                 'level': 'INFO',
                 'class': 'logging.FileHandler',
                 'filename': '/var/log/django/django.log',
             },
         },
         'loggers': {
             'django': {
                 'handlers': ['file'],
                 'level': 'INFO',
                 'propagate': True,
             },
         },
     }
     ```

102. **Production deployment with Gunicorn** - WSGI server for production.

     ```bash
     # Install Gunicorn
     pip install gunicorn
     
     # Basic Gunicorn command
     gunicorn myproject.wsgi:application --bind 0.0.0.0:8000
     
     # Production Gunicorn with optimizations
     gunicorn myproject.wsgi:application \
         --bind 0.0.0.0:8000 \
         --workers 3 \
         --timeout 120 \
         --keep-alive 2 \
         --max-requests 1000 \
         --max-requests-jitter 100 \
         --access-logfile /var/log/gunicorn/access.log \
         --error-logfile /var/log/gunicorn/error.log \
         --log-level info
     ```

     ```python
     # gunicorn.conf.py - Gunicorn configuration file
     import multiprocessing
     
     # Server socket
     bind = "0.0.0.0:8000"
     backlog = 2048
     
     # Worker processes
     workers = multiprocessing.cpu_count() * 2 + 1
     worker_class = "sync"
     worker_connections = 1000
     timeout = 120
     keepalive = 2
     max_requests = 1000
     max_requests_jitter = 100
     
     # Logging
     accesslog = "/var/log/gunicorn/access.log"
     errorlog = "/var/log/gunicorn/error.log"
     loglevel = "info"
     
     # Process naming
     proc_name = "myproject"
     
     # Server mechanics
     daemon = False
     pidfile = "/var/run/gunicorn/myproject.pid"
     user = "www-data"
     group = "www-data"
     tmp_upload_dir = None
     
     # SSL (if terminating SSL at Gunicorn level)
     # keyfile = "/path/to/keyfile"
     # certfile = "/path/to/certfile"
     ```

103. **Systemd service for Django** - Running Django as a system service.

     ```ini
     # /etc/systemd/system/myproject.service
     [Unit]
     Description=Gunicorn instance to serve myproject
     After=network.target
     
     [Service]
     User=www-data
     Group=www-data
     WorkingDirectory=/var/www/myproject
     Environment="PATH=/var/www/myproject/venv/bin"
     EnvironmentFile=/var/www/myproject/.env
     ExecStart=/var/www/myproject/venv/bin/gunicorn \
               --config /var/www/myproject/gunicorn.conf.py \
               myproject.wsgi:application
     ExecReload=/bin/kill -s HUP $MAINPID
     Restart=always
     RestartSec=3
     
     [Install]
     WantedBy=multi-user.target
     ```

     ```bash
     # Systemd service management
     # Enable and start service
     sudo systemctl daemon-reload
     sudo systemctl enable myproject
     sudo systemctl start myproject
     
     # Check service status
     sudo systemctl status myproject
     
     # View logs
     sudo journalctl -u myproject -f
     
     # Restart service
     sudo systemctl restart myproject
     ```

## Database Management

104. **Database setup for different environments** - Managing databases across dev/staging/production.

     ```bash
     # DEVELOPMENT DATABASE SETUP
     
     # SQLite (simplest for development)
     python manage.py migrate
     python manage.py createsuperuser
     python manage.py loaddata fixtures/sample_data.json
     
     # PostgreSQL (recommended for production-like development)
     # 1. Create database
     createdb myproject_dev
     
     # 2. Run migrations
     python manage.py migrate
     
     # 3. Load sample data
     python manage.py loaddata fixtures/sample_data.json
     ```

     ```bash
     # PRODUCTION DATABASE SETUP
     
     # 1. Create production database
     sudo -u postgres createdb myproject_prod
     sudo -u postgres createuser myproject_user
     sudo -u postgres psql -c "ALTER USER myproject_user WITH PASSWORD 'secure_password';"
     sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE myproject_prod TO myproject_user;"
     
     # 2. Set environment variables
     export DATABASE_URL="postgresql://myproject_user:secure_password@localhost:5432/myproject_prod"
     
     # 3. Run migrations
     python manage.py migrate --settings=myproject.settings.production
     
     # 4. Create superuser
     python manage.py createsuperuser --settings=myproject.settings.production
     
     # 5. Load production data (if any)
     python manage.py loaddata fixtures/production_data.json --settings=myproject.settings.production
     ```

105. **Database backup and restore** - Essential database management commands.

     ```bash
     # POSTGRESQL BACKUP AND RESTORE
     
     # Create backup
     pg_dump myproject_prod > backup_$(date +%Y%m%d_%H%M%S).sql
     
     # Create compressed backup
     pg_dump myproject_prod | gzip > backup_$(date +%Y%m%d_%H%M%S).sql.gz
     
     # Restore from backup
     psql myproject_prod < backup_20231201_143000.sql
     
     # Restore from compressed backup
     gunzip -c backup_20231201_143000.sql.gz | psql myproject_prod
     
     # Django fixtures (for small datasets)
     # Export data
     python manage.py dumpdata myapp.Model --indent 2 > fixtures/myapp_data.json
     
     # Import data
     python manage.py loaddata fixtures/myapp_data.json
     ```

## Complete Deployment Workflows

106. **Development workflow** - Daily development process.

     ```bash
     # DAILY DEVELOPMENT WORKFLOW
     
     # 1. Start development environment
     source venv/bin/activate
     
     # 2. Pull latest changes
     git pull origin main
     
     # 3. Install new dependencies (if requirements.txt changed)
     pip install -r requirements.txt
     
     # 4. Run migrations (if models changed)
     python manage.py migrate
     
     # 5. Start React development server (if using React)
     cd frontend
     npm start &
     cd ..
     
     # 6. Start Django development server
     python manage.py runserver
     
     # Development is ready at:
     # Django API: http://localhost:8000
     # React App: http://localhost:3000 (if separate)
     # Or just: http://localhost:8000 (if Django serves React)
     ```

107. **Production deployment workflow** - Step-by-step production deployment.

     ```bash
     # PRODUCTION DEPLOYMENT WORKFLOW
     
     # 1. Prepare code
     git clone https://github.com/yourusername/myproject.git /var/www/myproject
     cd /var/www/myproject
     
     # 2. Set up Python environment
     python3 -m venv venv
     source venv/bin/activate
     pip install -r requirements.txt
     
     # 3. Build React application (if using React)
     cd frontend
     npm install
     npm run build
     cd ..
     
     # 4. Set up environment variables
     cp .env.example .env
     # Edit .env with production values
     nano .env
     
     # 5. Set up database
     sudo -u postgres createdb myproject_prod
     python manage.py migrate --settings=myproject.settings.production
     
     # 6. Collect static files
     python manage.py collectstatic --noinput --settings=myproject.settings.production
     
     # 7. Create superuser
     python manage.py createsuperuser --settings=myproject.settings.production
     
     # 8. Set up Gunicorn service
     sudo cp deploy/myproject.service /etc/systemd/system/
     sudo systemctl daemon-reload
     sudo systemctl enable myproject
     sudo systemctl start myproject
     
     # 9. Set up Nginx (reverse proxy)
     sudo cp deploy/nginx.conf /etc/nginx/sites-available/myproject
     sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled/
     sudo nginx -t
     sudo systemctl restart nginx
     
     # 10. Set up SSL (Let's Encrypt)
     sudo certbot --nginx -d yourdomain.com
     ```

108. **Nginx configuration for Django** - Reverse proxy and static file serving.

     ```nginx
     # /etc/nginx/sites-available/myproject
     server {
         listen 80;
         server_name yourdomain.com www.yourdomain.com;
         
         # Redirect HTTP to HTTPS
         return 301 https://$server_name$request_uri;
     }
     
     server {
         listen 443 ssl http2;
         server_name yourdomain.com www.yourdomain.com;
         
         # SSL configuration (managed by Certbot)
         ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
         ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
         
         # Security headers
         add_header X-Frame-Options "SAMEORIGIN" always;
         add_header X-XSS-Protection "1; mode=block" always;
         add_header X-Content-Type-Options "nosniff" always;
         add_header Referrer-Policy "no-referrer-when-downgrade" always;
         add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
         
         # Static files
         location /static/ {
             alias /var/www/myproject/staticfiles/;
             expires 1y;
             add_header Cache-Control "public, immutable";
         }
         
         # Media files
         location /media/ {
             alias /var/www/myproject/media/;
             expires 1y;
             add_header Cache-Control "public";
         }
         
         # Proxy to Gunicorn
         location / {
             proxy_pass http://127.0.0.1:8000;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
             proxy_set_header Host $http_host;
             proxy_redirect off;
             
             # Timeouts
             proxy_connect_timeout 60s;
             proxy_send_timeout 60s;
             proxy_read_timeout 60s;
         }
     }
     ```

109. **Deployment automation script** - Automate the deployment process.

     ```bash
     #!/bin/bash
     # deploy.sh - Production deployment script
     
     set -e  # Exit on any error
     
     PROJECT_NAME="myproject"
     PROJECT_DIR="/var/www/$PROJECT_NAME"
     SERVICE_NAME="$PROJECT_NAME"
     
     echo "🚀 Starting deployment of $PROJECT_NAME..."
     
     # 1. Pull latest code
     echo "📥 Pulling latest code..."
     cd $PROJECT_DIR
     git pull origin main
     
     # 2. Activate virtual environment
     echo "🐍 Activating virtual environment..."
     source venv/bin/activate
     
     # 3. Install/update dependencies
     echo "📦 Installing dependencies..."
     pip install -r requirements.txt
     
     # 4. Build React app (if exists)
     if [ -d "frontend" ]; then
         echo "⚛️ Building React application..."
         cd frontend
         npm install
         npm run build
         cd ..
     fi
     
     # 5. Run database migrations
     echo "🗄️ Running database migrations..."
     python manage.py migrate --settings=$PROJECT_NAME.settings.production
     
     # 6. Collect static files
     echo "📁 Collecting static files..."
     python manage.py collectstatic --noinput --settings=$PROJECT_NAME.settings.production
     
     # 7. Restart services
     echo "🔄 Restarting services..."
     sudo systemctl restart $SERVICE_NAME
     sudo systemctl restart nginx
     
     # 8. Check service status
     echo "✅ Checking service status..."
     sudo systemctl is-active --quiet $SERVICE_NAME && echo "✅ $SERVICE_NAME is running" || echo "❌ $SERVICE_NAME failed to start"
     sudo systemctl is-active --quiet nginx && echo "✅ Nginx is running" || echo "❌ Nginx failed to start"
     
     echo "🎉 Deployment completed successfully!"
     ```

110. **Environment-specific commands** - Quick reference for different environments.

     ```bash
     # DEVELOPMENT COMMANDS
     
     # Start development server
     python manage.py runserver
     
     # Run with specific settings
     python manage.py runserver --settings=myproject.settings.development
     
     # Database operations
     python manage.py makemigrations
     python manage.py migrate
     python manage.py dbshell
     
     # Create test data
     python manage.py loaddata fixtures/test_data.json
     
     # PRODUCTION COMMANDS
     
     # Run with production settings
     python manage.py migrate --settings=myproject.settings.production
     python manage.py collectstatic --settings=myproject.settings.production
     python manage.py createsuperuser --settings=myproject.settings.production
     
     # Start with Gunicorn
     gunicorn myproject.wsgi:application --settings=myproject.settings.production
     
     # Service management
     sudo systemctl start myproject
     sudo systemctl stop myproject
     sudo systemctl restart myproject
     sudo systemctl status myproject
     
     # View logs
     sudo journalctl -u myproject -f
     tail -f /var/log/gunicorn/error.log
     ```
