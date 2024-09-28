# Authentication

## [Token Based Authentication](#token-based-authentication-in-django-rest-framework-drf)<br>

## [Using Djoser for Authentication in DRF](#using-djoser-for-authentication-in-django-rest-framework-drf)

## [JWT Authentication with DRF](#jwt-authentication-with-django-rest-framework-drf)

## Token-Based Authentication in Django REST Framework (DRF)

Token-based authentication is a simple, stateless, and scalable method for securing APIs. In Django REST Framework (DRF), token-based authentication involves issuing a token to a user after they authenticate, which is then used to identify the user on subsequent requests. This allows the API to verify requests based on the provided token without needing to reauthenticate the user for every request.

#### How Token-Based Authentication Works:
1. **User logs in** by providing credentials (username, password) to a login endpoint.
2. **Token is generated** for the authenticated user and sent back as a response.
3. **Token is stored** on the client (usually in local storage or cookies).
4. For subsequent API requests, the client sends the **token in the request header** for authentication.
5. The server validates the token and allows access to the requested resources.

### Setting Up Token-Based Authentication in DRF

To implement token-based authentication in DRF, you can follow these steps:

#### 1. Install `djangorestframework` and `djangorestframework-authtoken`

Make sure you have installed the DRF and the `djangorestframework-authtoken` package:

```bash
pip install djangorestframework
pip install djangorestframework-authtoken
```

#### 2. Add `'rest_framework.authtoken'` to Installed Apps

In your Django `settings.py` file, add `rest_framework` and `rest_framework.authtoken` to the `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    'rest_framework',
    'rest_framework.authtoken',
    ...
]
```

#### 3. Migrate the Database

Run the migration to create the token table:

```bash
python manage.py migrate
```

#### 4. Create a Token Authentication Endpoint

You can either create a custom login view or use DRF’s built-in view to handle token generation. DRF provides a built-in view called `ObtainAuthToken` to generate a token when a user logs in.

In your `urls.py` file, include the following:

```python
from django.urls import path
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path('api-token-auth/', obtain_auth_token),  # Built-in DRF token view
]
```

This endpoint accepts a `POST` request with the user's credentials (username and password) and returns a token.

#### 5. Configure DRF to Use Token Authentication

In the `settings.py` file, configure DRF to use token-based authentication as the default authentication method:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',  # Require authentication globally
    ],
}
```

#### 6. How to Use the Token in Requests

Once a token is obtained from the `api-token-auth/` endpoint, it must be sent in the header of subsequent requests.

**Example Header:**
```http
Authorization: Token your_token_here
```

#### 7. Example Code

Here’s an example of using token authentication in a DRF project.

##### Model:

```python
from django.contrib.auth.models import User
from django.db import models

class Post(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    title = models.CharField(max_length=100)
    content = models.TextField()

    def __str__(self):
        return self.title
```

##### Serializer:

```python
from rest_framework import serializers
from .models import Post

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'author']
```

##### Views:

```python
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated
from rest_framework.authentication import TokenAuthentication
from .models import Post
from .serializers import PostSerializer

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    authentication_classes = [TokenAuthentication]
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
```

##### URL Configuration:

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import PostViewSet

router = DefaultRouter()
router.register(r'posts', PostViewSet)

urlpatterns = [
    path('', include(router.urls)),
    path('api-token-auth/', obtain_auth_token),  # Token authentication endpoint
]
```

#### 8. Testing with cURL

Here’s how you can test token authentication using `cURL`:

1. **Obtain Token** (Login):

```bash
curl -X POST -d "username=your_username&password=your_password" http://127.0.0.1:8000/api-token-auth/
```

Response:

```json
{
    "token": "your_generated_token"
}
```

2. **Make Authenticated Request**:

Once you have the token, you can make authenticated requests by including the token in the `Authorization` header.

```bash
curl -H "Authorization: Token your_generated_token" http://127.0.0.1:8000/posts/
```

#### 9. Token Expiry and Management

DRF’s default token system does **not expire tokens**. To manage token expiration or implement refresh tokens, you might want to use third-party packages like `djangorestframework-simplejwt` or `django-rest-knox`.

---

### Summary

- Token-based authentication in DRF is a simple way to authenticate users for APIs.
- Tokens are generated for users and sent with subsequent API requests via the `Authorization` header.
- DRF offers built-in views to obtain tokens and settings to configure token-based authentication.


## Using Djoser for Authentication in Django REST Framework (DRF)

Djoser is a powerful and easy-to-use library for handling user authentication in Django REST Framework (DRF). It provides a set of views and endpoints to handle user registration, login, password management, token-based authentication, and more. Djoser integrates seamlessly with DRF and allows you to easily implement authentication mechanisms without the need to manually write the logic for these features.

### Features of Djoser:
- User registration and activation
- Login and logout
- Password reset and change
- Token-based authentication (with Django REST Framework's `rest_framework.authtoken`)
- JWT (JSON Web Token) support with third-party libraries like `djangorestframework-simplejwt`

### Steps to Implement Authentication Using Djoser in DRF

#### 1. Install Required Packages

You need to install the following packages:
- `djangorestframework`
- `djoser`
- `djangorestframework.authtoken` (for token-based authentication)

```bash
pip install djangorestframework
pip install djoser
pip install djangorestframework-authtoken
```

#### 2. Add to `INSTALLED_APPS`

In your Django `settings.py` file, add `rest_framework`, `rest_framework.authtoken`, and `djoser` to the `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    'rest_framework',
    'rest_framework.authtoken',  # Required for token-based authentication
    'djoser',                    # Djoser app
    ...
]
```

#### 3. Configure DRF Authentication Classes

In your `settings.py`, configure the authentication classes in the `REST_FRAMEWORK` dictionary. For token-based authentication, you can use `TokenAuthentication`.

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',  # Global authentication
    ],
}
```

#### 4. Configure Djoser Settings (Optional)

Djoser provides customizable settings. You can define them in your `settings.py` file to control aspects like username usage, activation URLs, token lifetimes, etc.

Here’s an example configuration:

```python
DJOSER = {
    'LOGIN_FIELD': 'email',  # To use email for login instead of username
    'USER_CREATE_PASSWORD_RETYPE': True,  # Confirm password during registration
    'SEND_ACTIVATION_EMAIL': False,  # Set True if you want activation via email
    'SERIALIZERS': {
        'user_create': 'djoser.serializers.UserCreateSerializer',  # Custom registration serializer
        'user': 'djoser.serializers.UserSerializer',  # Custom user serializer
        'current_user': 'djoser.serializers.UserSerializer',
    },
}
```

#### 5. Add URL Patterns for Djoser

Djoser provides predefined endpoints that you can use for user authentication. You need to include the Djoser URLs in your `urls.py` file.

```python
from django.urls import path, include

urlpatterns = [
    path('auth/', include('djoser.urls')),  # Djoser core endpoints (registration, login, etc.)
    path('auth/', include('djoser.urls.authtoken')),  # Token-based authentication endpoints
]
```

#### 6. Available Endpoints in Djoser

Djoser provides a variety of built-in endpoints for handling user authentication, including:

- **User Registration**:
  - `POST /auth/users/`: Register a new user
  - `POST /auth/users/activation/`: Activate a user account (if email activation is enabled)
  
- **Login & Logout**:
  - `POST /auth/token/login/`: Obtain an authentication token
  - `POST /auth/token/logout/`: Log out the user (invalidate the token)

- **Password Management**:
  - `POST /auth/users/reset_password/`: Send password reset email
  - `POST /auth/users/reset_password_confirm/`: Reset password
  - `POST /auth/users/set_password/`: Change password for logged-in user

- **User Information**:
  - `GET /auth/users/me/`: Get details of the current logged-in user
  - `GET /auth/users/{id}/`: Get user details by ID

#### 7. Example Usage of Token-Based Authentication

**User Registration** (`POST /auth/users/`):

```bash
curl -X POST http://127.0.0.1:8000/auth/users/ -d \
'{"email": "user@example.com", "username": "username", "password": "password"}'
```

**Login (Obtain Token)** (`POST /auth/token/login/`):

```bash
curl -X POST http://127.0.0.1:8000/auth/token/login/ -d \
'{"email": "user@example.com", "password": "password"}'
```

Response:

```json
{
    "auth_token": "your_token_here"
}
```

**Access Authenticated Endpoints**:

After obtaining the token, you can use it in the `Authorization` header for accessing authenticated endpoints.

```bash
curl -H "Authorization: Token your_token_here" http://127.0.0.1:8000/your-protected-endpoint/
```

**Logout** (`POST /auth/token/logout/`):

```bash
curl -X POST -H "Authorization: Token your_token_here" http://127.0.0.1:8000/auth/token/logout/
```

### 8. Customizing Djoser Serializers

Djoser allows you to customize the serializers used for registration and user management. For example, if you want to add more fields to the user registration serializer, you can create a custom serializer.

```python
from djoser.serializers import UserCreateSerializer as BaseUserCreateSerializer
from rest_framework import serializers
from django.contrib.auth import get_user_model

User = get_user_model()

class UserCreateSerializer(BaseUserCreateSerializer):
    class Meta(BaseUserCreateSerializer.Meta):
        model = User
        fields = ['id', 'email', 'username', 'password', 'first_name', 'last_name']
```

Then, in your `settings.py`, you can override the default registration serializer with your custom serializer:

```python
DJOSER = {
    'SERIALIZERS': {
        'user_create': 'your_app.serializers.UserCreateSerializer',
    },
}
```

### 9. Testing Endpoints

You can use tools like `Postman`, `Insomnia`, or even `cURL` to test the endpoints provided by Djoser for registration, login, and other user-related actions.

---

### Summary of Djoser Authentication

- **Djoser** simplifies authentication by providing built-in views and endpoints for user registration, login, and password management.
- **Token-based authentication** is supported out of the box using Django's `rest_framework.authtoken`.
- Djoser can be customized with your own serializers and configurations via the `DJOSER` settings.
- Endpoints are ready to use, so no need to write custom views for common authentication actions.

With Djoser, you can handle all authentication aspects of your DRF application efficiently while maintaining flexibility for customization.

## JWT Authentication with Django REST Framework (DRF)

JSON Web Tokens (JWT) are a popular way to handle authentication in modern web applications. They allow users to authenticate and authorize API requests by providing a token that is sent in the `Authorization` header of HTTP requests. 

Django REST Framework (DRF) provides a package called `djangorestframework-simplejwt` to handle JWT-based authentication.

### Setting up JWT Authentication in DRF

#### 1. Install Required Packages

You need to install the following packages:
- `djangorestframework`
- `djangorestframework-simplejwt`

```bash
pip install djangorestframework
pip install djangorestframework-simplejwt
```

#### 2. Update `INSTALLED_APPS`

In your Django `settings.py`, add `rest_framework` to `INSTALLED_APPS`.

```python
INSTALLED_APPS = [
    'rest_framework',
    ...
]
```

#### 3. Configure JWT Authentication in DRF

In your `settings.py`, configure DRF to use JWT for authentication by adding the `rest_framework_simplejwt.authentication.JWTAuthentication` class to the `DEFAULT_AUTHENTICATION_CLASSES`.

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',  # By default, require authentication for API views
    ],
}
```

#### 4. Add JWT Settings (Optional)

You can also configure JWT settings in `settings.py` if needed. For example, you can set token lifetimes and other security-related options.

```python
from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=30),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': SECRET_KEY,  # Use your Django SECRET_KEY as the signing key
    'AUTH_HEADER_TYPES': ('Bearer',),
}
```

#### 5. Define URLs for JWT Authentication

You can now add JWT-related authentication endpoints to your `urls.py`. The `djangorestframework-simplejwt` library provides views for obtaining, refreshing, and verifying tokens.

```python
from django.urls import path
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
    TokenVerifyView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('api/token/verify/', TokenVerifyView.as_view(), name='token_verify'),
]
```

#### 6. Register User Endpoint

For user registration, you need to define a registration view and serializer. This can be done by creating a custom view and serializer for user registration.

##### Example: User Registration Serializer

```python
from rest_framework import serializers
from django.contrib.auth.models import User

class RegisterSerializer(serializers.ModelSerializer):
    password = serializers.CharField(write_only=True)

    class Meta:
        model = User
        fields = ['username', 'email', 'password']

    def create(self, validated_data):
        user = User(
            username=validated_data['username'],
            email=validated_data['email'],
        )
        user.set_password(validated_data['password'])
        user.save()
        return user
```

##### Example: User Registration View

```python
from rest_framework import generics
from .serializers import RegisterSerializer
from rest_framework.permissions import AllowAny

class RegisterView(generics.CreateAPIView):
    queryset = User.objects.all()
    permission_classes = (AllowAny,)  # Allow anyone to access the registration endpoint
    serializer_class = RegisterSerializer
```

##### Add Registration URL

You also need to add the registration endpoint to your `urls.py`.

```python
from django.urls import path
from .views import RegisterView

urlpatterns = [
    path('api/register/', RegisterView.as_view(), name='register'),
]
```

#### 7. Example JWT Authentication Flow

Here’s a typical flow for user registration and authentication using JWT:

1. **User Registration** (`POST /api/register/`)
    - This endpoint allows new users to register.

   Example request:

    ```json
    {
        "username": "testuser",
        "email": "testuser@example.com",
        "password": "password123"
    }
    ```

2. **Obtain JWT Token** (`POST /api/token/`)
    - This endpoint allows users to log in and get their JWT tokens (access and refresh tokens).

   Example request:

    ```json
    {
        "username": "testuser",
        "password": "password123"
    }
    ```

   Example response:

    ```json
    {
        "refresh": "your_refresh_token",
        "access": "your_access_token"
    }
    ```

3. **Use JWT Token for Authenticated Requests**
    - Once the user has the access token, it can be used to authenticate API requests by passing the token in the `Authorization` header.

    Example request with JWT token:

    ```bash
    curl -H "Authorization: Bearer your_access_token" http://127.0.0.1:8000/api/some-protected-resource/
    ```

4. **Refresh Token** (`POST /api/token/refresh/`)
    - When the access token expires, the user can use the refresh token to obtain a new access token.

    Example request:

    ```json
    {
        "refresh": "your_refresh_token"
    }
    ```

    Example response:

    ```json
    {
        "access": "new_access_token"
    }
    ```

5. **Verify Token** (`POST /api/token/verify/`)
    - This endpoint allows you to verify if a given JWT token is still valid.

    Example request:

    ```json
    {
        "token": "your_access_token"
    }
    ```

### Summary

By using JWT authentication in DRF with the `djangorestframework-simplejwt` library, you can implement a secure and scalable authentication system. You can also combine it with user registration endpoints to allow users to sign up and then log in using their credentials. The JWT tokens can then be used to authenticate requests to your protected API endpoints.