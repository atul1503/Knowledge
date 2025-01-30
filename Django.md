Notes:
1. Settings.py path is determined first from environment variable then if you have defined in manage.py file the path of settings.py file. Even in manage.py also you use os.envrion only to set settings.py module path.
2. You can use different settings.py for different environements.
3. If any settings is missing from your settings.py file it will be fetched from the default settings.py that comes with django or if you are using an external library then the library might have provided the default value, it will take that in worst case.
4.  
