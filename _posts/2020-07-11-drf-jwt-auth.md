---
title:  "Django REST Framework & JWT Authentication"
layout: post
---

![Django REST Framework](https://files.realpython.com/media/djang-rest-framework-logo.37921ea75c09.png)

Simple auth system w/ user registration. Powered by `djangorestframework-simplejwt` lib.

## Table of Contents

- [Requirements](#requirements)
- [Project Setup](#project-setup)
- [App Setup](#app-setup)
- [References](#references)

## Requirements

- **Python** (3.6, 3.7, 3.8)
- **Django** (2.0, 2.1, 2.2, 3.0)
- **Django REST Framework** (3.8, 3.9, 3.10)

## Project Setup

Add `djangorestframework-simplejwt` to `requirements.txt`. Make sure you installed it to your environment:

    pip install djangorestframework-simplejwt

Add new entry to `DEFAULT_AUTHENTICATION_CLASSES` in `project/settings.py`:

```python
REST_FRAMEWORK = {
	'DEFAULT_AUTHENTICATION_CLASSES': (
		'rest_framework_simplejwt.authentication.JWTAuthentication',
	)
}
```

Include in `ulrs.py` (for root or app) routes for `TokenObtainPairView` and `TokenRefreshView`:

```python
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

urlpatterns = [
	path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
	path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh')
]
```

> *Optionally*: add route for `TokenVerifyView`:
>   ```python
>   urlpatterns = [
>  	    ...
>		path('api/token/verify/', TokenVerifyView.as_view(), name='token_verify'),
>	]
>   ```

## App Setup

Create new Django app `jwtauth`:

    python manage.py startapp jwtauth

Add app definition in `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
	...
	'jwtauth',
]
```

Add `UserCreateSerializer` in `jwtauth/serializers.py`:

```python
from django.contrib.auth import get_user_model
from rest_framework import serializers

User = get_user_model()


class UserCreateSerializer(serializers.ModelSerializer):
    password = serializers.CharField(
        write_only=True,
        required=True,
        style={
            "input_type": "password"
        }
    )
    password2 = serializers.CharField(
        write_only=True,
        label="Confirm password",
        style={
            "input_type": "password"
        }
    )

    class Meta:
        model = User
        fields = ('username', 'email', 'password', 'password2', )
        extra_kwargs = {
            "password": {
                "write_only": True
            }
        }

    def create(self, validated_data):
        username = validated_data["username"]
        email = validated_data["email"]
        password = validated_data["password"]
        password2 = validated_data["password2"]

        if email and User.objects.filter(email=email).exclude(username=username).exists():
            raise serializers.ValidationError(
                {
                    "email": "Email addresses must be unique."
                }
            )

        if password != password2:
            raise serializers.ValidationError(
                {
                    "password": "The two passwords differ."
                }
            )

        user = User(username=username, email=email)
        user.set_password(password)
        user.save()

        return user
```

Add `UserRegistrationView` in `jwtauth/views.py`:

```python
from rest_framework import status
from rest_framework.generics import CreateAPIView
from rest_framework.permissions import AllowAny
from rest_framework.response import Response
from rest_framework_simplejwt.tokens import RefreshToken

from jwtauth.serializers import UserCreateSerializer


class UserRegistrationView(CreateAPIView):
    """
    API endpoint that allows user to be created through
    simple registration process.
    """
    permission_classes = (AllowAny, )
    serializer_class = UserCreateSerializer

    def post(self, request, *args, **kwargs):
        serializer = UserCreateSerializer(data=request.data)

        if not serializer.is_valid():
            return Response(serializer.errors, status.HTTP_400_BAD_REQUEST)

        user = serializer.save()

        refresh = RefreshToken.for_user(user)

        res = {
            "refresh": str(refresh),
            "access": str(refresh.access_token),
        }

        return Response(res, status.HTTP_201_CREATED)
```

Add route for `register/` in `jwtauth/urls.py`:

```python
from django.conf.urls import path
from .views import registration

urlpatterns = [
    path('register/', registration, name='register')
]
```

Include `jwtauth` routes in `project/urls.py`:

```python
...
urlpatterns = [
	...
	path('api/jwtauth/', include('jwtauth.urls'), name='jwtauth'),
]
```

## References

1. [JWT Auth with DjangoREST API](https://medium.com/swlh/jwt-auth-with-djangorest-api-9fb32b99b33c)
2. [DRF: Generic views](https://www.django-rest-framework.org/api-guide/generic-views/)
3. [django-rest-framework-simplejwt](https://github.com/SimpleJWT/django-rest-framework-simplejwt)