=========
Mango-JWT
=========

Mango-JWT is a minimal JWT User Authentication tool for Django Rest Framework and MongoDB. Recommended for developers using django-rest-framework and pymongo. Not supported in versions below Django 2.0. ::

    pip install mango-jwt



Quick start
-----------

1. Add "mongo_auth" to your INSTALLED_APPS setting below "rest_framework"::

    INSTALLED_APPS = [
        ...
        'rest_framework',
        'mongo_auth',
    ]


2. Include the mongo_auth URLconf in your project urls.py like this::

    path('mongo_auth/', include('mongo_auth.urls')),

3. Defining connections between applications and MongoDB in settings.py :- ::

    # URI formats
    MANGO_JWT_SETTINGS = {
        "uri_connection": "mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[defaultauthdb][?options]]",
        "db_name": "for_example_auth_db"
    }
    # Example (URI formats):
        # For a standalone:
            "mongodb://mongodb0.example.com:27017"
        # For a standalone that enforces access control:
            "mongodb://myDBReader:D1fficultP%40ssw0rd@mongodb0.example.com:27017/?authSource=admin"

    # Minimal settings (all mandatory)
    MANGO_JWT_SETTINGS = {
        "db_host": "some_db_host", # Use srv host if connecting with MongoDB Atlas Cluster
        "db_port": "some_db_port", # Don't include this field if connecting with MongoDB Atlas Cluster
        "db_name": "for_example_auth_db",
        "db_user": "username",
        "db_pass": "password"
    }


    # Or use Advanced Settings (including optional settings)
    MANGO_JWT_SETTINGS = {
        "db_host": "some_db_host",
        "db_port": "some_db_port",
        "db_name": "for_example_auth_db",
        "db_user": "username",
        "db_pass": "password",
        "auth_collection": "name_your_auth_collection", # default is "user_profile"
        "fields": ("email", "password"), # default
        "jwt_secret": "secret", # default
        "jwt_life": 7, # default (in days)
        "secondary_username_field": "mobile" # default is None
    }


**PLEASE NOTE: If you are connecting MongoDB Atlas Cluster, don't include "db_port" and use srv host in "db_host" e.g. if your host is showing mongodb+srv://something.mongodb.net/test in your account, then use "something.mongodb.net" as your host.**

4. If **secondary_username_field** is provided, users will be able to login with this field as well as "email". This is best for scenarios where you want users to login with either of their unique fields.

   For example, you may want users to login with "email" or "mobile".

5. You may or may not include "secondary_username_field" in "fields".

    **Note: "secondary_username_field" cannot be "email" as its "primary_username" and "secondary_username_field" will be set to None instead.**

6. Make a POST request on http://127.0.0.1:8000/mongo_auth/signup/ with body as :- ::

    {
        "email": "some_email@email.com",
        "password": "some_password",
        other_fields
        ...
    }

7. Now login with these credentials at http://127.0.0.1:8000/mongo_auth/login/ :- ::

    {
        "username": "some_email@email.com or secondary_username_field_value",
        "password": "some_password"
    }

8. This will return a JWT. Pass this JWT in your request in **"Authorization"** header.

---------------------------
AuthenticatedOnly
---------------------------

The **AuthenticatedOnly** permission class will only allow authenticated users to access your endpoint. ::

    from rest_framework.views import APIView
    from mongo_auth.permissions import AuthenticatedOnly
    from rest_framework.response import Response
    from rest_framework import status

    class GetTest(APIView):

        permission_classes = [AuthenticatedOnly]

        def get(self, request, format=None):
            try:
                print(request.user)  # This is where magic happens
                return Response(status=status.HTTP_200_OK,
                                data={"data": {"msg": "User Authenticated"}})
            except:
                return Response(status=status.HTTP_404_NOT_FOUND)


Or, if you're using the **@api_view** decorator with function based views. ::

    from mongo_auth.permissions import AuthenticatedOnly
    from rest_framework.decorators import permission_classes
    from rest_framework.decorators import api_view
    from rest_framework.response import Response
    from rest_framework import status

    @api_view(["GET"])
    @permission_classes([AuthenticatedOnly])
    def get_test(request):
        try:
            print(request.user)
            return Response(status=status.HTTP_200_OK,
                            data={"data": {"msg": "User Authenticated"}})
        except:
            return Response(status=status.HTTP_404_NOT_FOUND)


Don't forget to pass **"Authorization"** Header in your requests while using your views with **"AuthenticatedOnly"** Permission Class.

----------------------
mongo_auth.db.database
----------------------

As the Mongo Connection Object has already been initialised in the package, you can use it directly::

    from mongo_auth.db import database

    print(list(database["collection_name"].find({}, {"_id": 0}).limit(10)))


More Info
---------

1. Passlib is used for password encryption with default scheme as "django_pbkdf2_sha256".

2. Only for Django 2.0 and above.

3. Dependent on "django-rest-framework" and "pymongo".
