# WORKFLOW TO IMPLEMENT AUTHENTICATION SYSTEM WITH DJANGO REST FRAMEWORK USING JWT

## TABLE OF CONTENT 
* [Introduction](#introduction)
* [Techs Used](#tech-used)
* [Initial Setup](#initial-set-up)
* [Authentification](#authentification) 
* [JWT](#jwt-authentication)
* [Access Control](#access-control)
* [Logout](#logout)
* [Adding Authentication](#adding-authentication)
* [Test Links](#test-links)
* [References](#references)


## INTRODUCTION

With more Django developers moving away from using Django frontend and implementing React with Django Rest Framework, this repo highlights the main procedures involved in implementing such a stack using Token authentication.

## TECH USED

* Python
* Django Rest Framework
* dj-rest-auth with django-allauth for registration
* simple JWT
* swagger for API documentation
* corsheaders

## Initial Set Up

1. Install django rest framework : ``` pip install djangorestframework ```.
2. Add ``` rest_framework ``` to apps.

```
    INSTALLED_APPS = [
        ...
        "rest_framework",
        ...
    ]
```
3. Install drf-spectacular to document endpoints and then add ```drf_spectacular ``` yo apps and update urls in main app. 

```
    INSTALLED_APPS = [
        ...
        "drf_spectacular",
        ...
    ]
```

```
    from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView
```


```
     path("docs/",
        SpectacularSwaggerView.as_view(
            template_name="swagger-ui.html", url_name="schema"
        ),
        name="swagger-ui",
    ),
```

Include templates and update templates in settings to point to new template folder. Add drf-spectacular to REST_FRAMEWORK in settings.py.

```
    REST_FRAMEWORK = {
        "DEFAULT_SCHEMA_CLASS": "drf_spectacular.openapi.AutoSchema",
    }
```

The final swagger interface can be found [here](https://drfapi.theflyu2.com/docs/)

## Authentification

1. pip install ``` pip install dj-rest-auth ```

If registration is required we must install django-allauth:

* ```pip install django-allauth``` and add following to apps:

``` 
    ...
    'django.contrib.sites'
    'allauth'
    'allauth.account'
    'allauth.socialaccount'
    'dj_rest_auth'
    'rest_framework.authtoken'
    'dj_rest_auth.registration'
    ...
    SITE_ID = 1
```

* add allauth url to main.app urls.py ``` path("accounts/", include("allauth.urls")), ``` to make use of allauth templates and views

* dj-rest-auth registration email confirmation urls must be overwrite to make use of allauth's email confirmation.

2. update urls in main app to include dj-rest-framework urls.

``` path('dj-rest-auth/', include('dj_rest_auth.urls')), ```
``` path('dj-rest-auth/registration/', include('dj_rest_auth.registration.urls')) ```

At this stage we are using dj-rest-framework default athentification system which returns a token when using the login endpoint.


## JWT Authentication

JsonWebToken authentication is implemented using simple jwt as per document located [here](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/)

1. ``` pip install djangorestframework-simplejwt ```
2. update REST_AUTH in settings.py by adding ```rest_framework_simplejwt.authentication.JWTAuthentication ```
3. update urls in main.app urls with:
    ```
        from rest_framework_simplejwt.views import (
        TokenObtainPairView,
            TokenRefreshView,
        )
        urlpatterns = [
            ...
            path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
            path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
            ...
        ]
    ```

At this stage with have created the login token endpoints which will return token, and refresh token.

4. Customise cookie headers as follows:

```JWT_AUTH_COOKIE = 'my-app-auth'```
```JWT_AUTH_REFRESH_COOKIE = 'my-refresh-token'```

```
    SIMPLE_JWT = {
        'ACCESS_TOKEN_LIFETIME': timedelta(minutes=5),
        'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
        }
```

Setting REST_USE_JWT = True will make the /drf-rest-auth/login return a bearer token is so required. My preference is to set it to False and use the ```api/token``` endpoint.

Notes: The email confirmation template does not exist with dj-rest-auth so it must me overwritten to use django-allauth confirm mail or a custom template.

Update REST_FRAMEWORK settings:
 ```
    REST_FRAMEWORK = {
    ...
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    }
 ```

# Access Control

Access Control is implemented using django cors headers package:

``` pip install django-cors-headers ```

Add ``` corsheaders ``` headers to app and add ``` corsheaders.middleware.CorsMiddleware ``` to middleware to allow cross origin resource sharing and add the domain of your frontend app to ```CORS_ORIGIN_WHITELIST ``` and set ``` CORS_ORIGIN_ALLOW_ALL=True ```

# Logout

Delete Token from local storage client side and limit the token lifespan.

# Adding Authentication

With the above setup, incorporating authentification into views can be done by adding the permission class.
    ``` permission_classes = [IsAuthenticated] ``` 

For example, let's add the above permission class to the GetUser view:

```
    class GetUser(generics.ListAPIView):

    queryset = User.objects.all()
    serializer_class = UserSerializer

    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        """
        view to return current user
        """
        user_ = self.request.user.username
        return self.queryset.filter(username=user_)

```

If we were to inspect our ```admin_panel/get_user/``` endpoint using [swagger](https://drfapi.theflyu2.com/docs/), it will be seen that we now require a token to utilize the endpoint.

We can add more authentication types by updating our permissions:

``` permission_classes = [IsAdminUser, IsAuthenticated] ```

# Test Links

The following links provides a frontend interface implementing the authentification system
detailed above when logging in.

* [Frontend](https://drf.hansolo.digital/)

* [swagger](https://drfapi.theflyu2.com/docs/)

# References

* https://www.django-rest-framework.org/
* https://dj-rest-auth.readthedocs.io/en/latest/
* https://django-rest-framework-simplejwt.readthedocs.io/en/latest/




