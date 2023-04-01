## REST API with Django Framework

There is a full-crash course of Django and Implementation of React.JS in the [link](https://www.youtube.com/watch?v=c708Nf0cHrs).
<hr>

Technologies Binded/Connected Eachother.
* Django [Rest Framework](https://www.django-rest-framework.org/)
* Connected To [Django](https://www.djangoproject.com/)
* Written With [Python](https://www.python.org/)
* Going to test with [Requests](https://requests.readthedocs.io/en/latest/)
* Django [Cors Headers](https://pypi.org/project/django-cors-headers/) ??? (I don't know why we need it)

<hr>

python calling api via requests.
```python
get_response = requests.get(endpoint, params={"abc":123}, json{"query":"hello world"})
```

As accessing data through django it will be something like this inside the views.py
```python
from django.http import JsonResponse

def api_home(request, *args, **kwargs):

	body = request.body
	data = {}

	try:
		data = json.loads(body)
	except:
		pass

	data['params'] = dict(request.GET)              # info 1
	data['headers'] = dict(request.headers)         # info 2
	data['content_type'] = request.content_type     # info 3

	print(data)

	return JsonResponse({"message": "Test API Response"})
```
The ___request___ is not same as ___requests___. This request is captured by website, not the API calling library ~~___requests___~~. The data dictionary created before hand. If the __request.body__ is not empty, then add that information into variable __data__. Additionally, you can capture other information as shown.
* Info 1: The __params__ variable sended with requests. In this case ```{"abc":123}```
* Info 2: Headers captured by request, have bunch of information if you need them.
* Info 3: Example information that captured inside the __request.headers__. Example, content_type

Create two model object inside the django website. Now we are gonna send one of two object via API.

views.py
```python
from django.http import JsonResponse
import json
from product.models import Product

def api_home(request, *args, **kwargs):
	model_data = Product.objects.all().order_by("?").first()
	data = {}

	if model_data:
		data['id'] = model_data.id
		data['title'] = model_data.title
		data['content'] = model_data.content
		data['price'] = model_data.price

	return JsonResponse(data)
```

test.py
```python
import requests

endpoint = "http://127.0.0.1:8000/api/"

get_response = requests.get(endpoint, params={"abc":123}, json={"query":"hello"})
print(get_response.text)
```

> Converting model to dict.

```python
from django.http import JsonResponse
from django.forms.models import model_to_dict
import json
from product.models import Product

def api_home(request, *args, **kwargs):
	model_data = Product.objects.all().order_by("?").first()
	data = {}

	if model_data:
		data = model_to_dict(model_data, fields=['id','price'])

	return JsonResponse(data)
```

> Using rest framework serializer

In this session, I will show how to send property as well as another complicated function directly return as your preffered name. Let's write the codes and dive into it.

__basic.py__
```python
import requests

endpoint = "http://127.0.0.1:8000/api/"

get_response = requests.get(endpoint, params={"abc":123}, json={"query":"hello"})
print(get_response.text)
```

__api/views.py__
```python
# from django.http import JsonResponse
from django.forms.models import model_to_dict
import json
from product.models import Product

from rest_framework.decorators import api_view
from rest_framework.response import Response
from product.serializers import ProductSerializer

@api_view(["GET", "POST"])
def api_home(request, *args, **kwargs):
	instance = Product.objects.all().order_by("?").first()
	data = {}

	if instance:
		data = ProductSerializer(instance).data

	return Response(data)
```

__product/models.py__
```python
from django.db import models

# Create your models here.
class Product(models.Model):
	title 		= models.CharField(max_length=64)
	content 	= models.TextField(blank=True, null=True)
	price 		= models.DecimalField(max_digits=15, decimal_places=2, default=0.00)
	
	@property
	def sale_price(self):
		return "%.2f" %(float(self.price) * 0.8)

	def get_discount(self):
		return "1234"
```

__product/serializer.py__
```python
from rest_framework import serializers

from .models import Product

class ProductSerializer(serializers.ModelSerializer):

	personal_discount = serializers.SerializerMethodField(read_only = True)

	class Meta:
		model = Product
		fields = [
			'title',
			'content',
			'price',
			'sale_price',
			'personal_discount',
		]

	def get_personal_discount(self, obj):
		return obj.get_discount()
```

```console
(venv)$ python basic.py
> {"title": "Item 1", "content": "Whole another level", "price": "12.99", "sale_price": "10.39", "personal_discount": "1234"}
```

To sum up what happens is, first look at the model itself. We declare three constant field, as well as one function with property decorator. And one (assume complex) function with it. In serializer, we (like we do in forms) create MethodFeilds and got however we want to call it. One problem in this section is, we don't have field called in __personal_discount__, but we want to match the return value of __get_discount__ function. We bind that relation inside the __serializer.py__. After that point, we ~~instead of manually~~ make our data serialize our data thanks to rest_framework. Prepare our data inside the __views.py__ and respond as Json Format. With this structure, we don't have to fill all the field by ourselves.

___OQN___: obj inside the __get_personal_discount()__ function is accessing the instance that we send from views.py You can access varied parameters as needed.

> Ingest Data w/ REST Framework

We are going to change the __views.py__ file.

__views.py__
```python
# from django.http import JsonResponse
from django.forms.models import model_to_dict
import json
from product.models import Product

from rest_framework.decorators import api_view
from rest_framework.response import Response
from product.serializers import ProductSerializer

@api_view(["POST"])
def api_home(request, *args, **kwargs):

	# DRF API View -> Django Rest Framework
	serializer = ProductSerializer(data=request.data)
	if serializer.is_valid(raise_exception=True):
		print(serializer.data)
		return Response(serializer.data)
```

Now let's dive what this code does. Get data request (method as POST), and validate with serializer. If the value valid, it will return the data itself. However, if not it will send the problem with the post. Now let's look at the first program.

```python
import requests

endpoint = "http://127.0.0.1:8000/api/"

get_response = requests.post(endpoint, params={"abc":123}, json={"query":"hello"})
print(get_response.text)
```

```console
(venv)$ python basic.py 
{"title":["This field is required."]}
```

When we check the model, we can see that our title is not nullable and not blankable.<br>
```title = models.CharField(max_length=64)```

Now let's change our post to something like this<br>
```get_response = requests.post(endpoint, json={"title":"hello", "price":"mistake"})```

When we change like this we will end it up like that output
```console
(venv)$ python basic.py 
{"price":["A valid number is required."]}
```

As we can see here, it creates validation like form fields. But now, we create validation via rest framework.

> Django Rest Framework different API Views and end point connection.

__views.py__
```python
from rest_framework import generics

from .models import Product
from .serializers import ProductSerializer


class ProductDetailAPIView(generics.RetrieveAPIView):
	queryset = Product.objects.all()
	serializer_class = ProductSerializer


class ProductCreateAPIView(generics.CreateAPIView):
	queryset = Product.objects.all()
	serializer_class = ProductSerializer
	
	def perform_create(self, serializer):
		title = serializer.validated_data.get('title')
		content = serializer.validated_data.get('content') or None
		if content is None:
			content = title
		serializer.save(content=content)


class ProductListAPIView(generics.ListAPIView):
	queryset = Product.objects.all()
	serializer_class = ProductSerializer


class ProductListCreateAPIView(generics.ListCreateAPIView):
	queryset = Product.objects.all()
	serializer_class = ProductSerializer

	def perform_create(self, serializer):
		title = serializer.validated_data.get('title')
		content = serializer.validated_data.get('content') or None
		if content is None:
			content = title
		serializer.save(content=content)
```

__urls.py__
```python
from django.urls import path

from . import views

urlpatterns = [
	path('detail/<int:pk>/', views.ProductDetailAPIView.as_view()),
	path('create/', views.ProductListCreateAPIView.as_view()),
]
```

> Update and Delete

__views.py__
```python
class ProductDestroyAPIView(generics.DestroyAPIView):
	queryset = Product.objects.all()
	serializer_class = ProductSerializer
	lookup_field = 'pk'

	def perform_destroy(self, instance):
		super().perform_destroy(instance)

class ProductUpdateAPIView(generics.UpdateAPIView):
	queryset = Product.objects.all()
	serializer_class = ProductSerializer
	lookup_field = 'pk'

	def perform_update(self, serializer):
		instance = serializer.save()
		if not instance.content:
			instance.content = instance.title
```

__urls.py__
```python
path('<int:pk>/delete/', views.ProductDestroyAPIView.as_view()),
path('<int:pk>/update/', views.ProductUpdateAPIView.as_view()),
```

> Mixins and Generic API View (Not recommended.) 2:04:00 - 2:17:00

> Session Authentication & Permissions

Django [authentication](https://www.django-rest-framework.org/api-guide/authentication/) and [permission](https://www.django-rest-framework.org/api-guide/permissions/). When we check the official django website, we can see that [first code block](https://www.django-rest-framework.org/api-guide/views/#class-based-views) has example usage. Let's dive which line do what?
```python
authentication_classes = [authentication.TokenAuthentication]
permission_classes = [permissions.IsAdminUser]
```

As we can see here, we have two lists called ```authentication_classes``` and ```permission_classes```. These lists, basically, the place where you set your authentication and permission. More information and usages can be found in official links. That is usage for Class-Based views. However, it is also possible Function-Based views via [decorators](https://www.django-rest-framework.org/api-guide/views/#api-policy-decorators). Now in practical example, when you have no permission you'll get error in such.
```JSON
{"detail": "Authentication credentials were not provided."}
```

> User & Group Permission

First, we create dummy user to test things. Here's the configuration for that user.<br>
```
admin user:		dja
password:		admin123

Username:		dummy
Password:		dja12345
Status:			True
Staff Status:	True
```

From there, we add give permission to our user to see products. Note that able to that, you should register your product class to admin page.

![Picture](/Pics/11.PNG)

When we set in such way, this user can change the product eventhough we didn't give permission to do so. This is because of the ```ProductListCreateAPIView```. When we change that authentication mode from ```permission_classes = [permissions.IsAuthenticated]``` to ```permission_classes = [permissions.DjangoModelPermissions]``` it also block that ability as well.

But eventhough we remove the user permission to view product. We can get via API since it's default behaviour for permission. In order to prevent that, custom field permission must be created.

> Custom Permission

The official [link](https://www.django-rest-framework.org/api-guide/permissions/#custom-permissions) for custom permission. To control over API, we will modify, ~~overwrite~~, DjangoModelPermissions.

We will create permissions.py under product folder. There is exact control inside class with if statement. However, It gets complicated. In case you want to see how if statement works here it is

```python
from rest_framework import permissions

class IsStaffEditorPermission(permissions.DjangoModelPermissions):
	def has_permission(self, request, view):
		user = request.user
		if user.is_staff:
			if user.has_perm("product.view_product"):
				return True
			return False
		return False
```

With this control stage, we can control if the user has specific permission or not. The usage in ```user.has_perm()``` can be used for control any state. the usage is, ```<appname>```.```<action>```_```<model-name>``` like shown in example.<br>
```
App Name	-> product
Action		-> View
Model Name	-> product
```

The main problem with structure, let's say you gave permission for view but not for add or delete. It's possible to reach those since the return is True about view_product.

__OQN__: To reach those, you need to switch permission __views.py__ in such

```python
from .permissions import IsStaffEditorPermission

class ProductListCreateAPIView(generics.ListCreateAPIView):
	# Codes ...
	permission_classes = [IsStaffEditorPermission]
	# Codes ...
```

To correctly write permission as we wanted, we will find to template permission. As we checked in previous example, ```Django -> permissions -> DjangoModelPermissions``` you able to see how permission structured, in belov.

```python
    perms_map = {
        'GET': [],
        'OPTIONS': [],
        'HEAD': [],
        'POST': ['%(app_label)s.add_%(model_name)s'],
        'PUT': ['%(app_label)s.change_%(model_name)s'],
        'PATCH': ['%(app_label)s.change_%(model_name)s'],
        'DELETE': ['%(app_label)s.delete_%(model_name)s'],
    }
```

But if you remember the structure, you can use get function eventhought we didn't give access to do it. Now, the modified version of permission will look like this.

__permissions.py__
```python
from rest_framework import permissions

class IsStaffEditorPermission(permissions.DjangoModelPermissions):

	perms_map = {
		'GET': ['%(app_label)s.view_%(model_name)s'],
		'OPTIONS': [],
		'HEAD': [],
		'POST': ['%(app_label)s.add_%(model_name)s'],
		'PUT': ['%(app_label)s.change_%(model_name)s'],
		'PATCH': ['%(app_label)s.change_%(model_name)s'],
		'DELETE': ['%(app_label)s.delete_%(model_name)s'],
	}
```

And, To control over if it's admin or not you will slightly change the views.py as well

```python
class ProductListAPIView(generics.ListAPIView):
	queryset = Product.objects.all()
	serializer_class = ProductSerializer


class ProductListCreateAPIView(generics.ListCreateAPIView):
	queryset = Product.objects.all()
	serializer_class = ProductSerializer
	permission_classes = [permissions.IsAdminUser, IsStaffEditorPermission]

	def perform_create(self, serializer):
		title = serializer.validated_data.get('title')
		content = serializer.validated_data.get('content') or None
		if content is None:
			content = title
		serializer.save(content=content)
```

If you wish to check first if it's admin or not, you'll first check control via ```permissions.IsAdminUser``` then you'll check the and process ```IsStaffEditorPermission```.

> Token Authentication

Before continue, you need to set up the ```rest_framework.authtoken``` inside the installed_apps in settings.<br>
First, we need to make rest_framework can access and gives to token as wanted.

__api/urls.py__
```python
from django.urls import path
from rest_framework.authtoken.views import obtain_auth_token
from . import views

urlpatterns = [
	path('auth/', obtain_auth_token),
	path('', views.api_home),
]
```
The line "```from rest_framework.authtoken.views import obtain_auth_token```" makes this accessing process. Now, we will enable our views with token application via these code block

```python
class ProductListCreateAPIView(generics.ListCreateAPIView):
	queryset = Product.objects.all()
	serializer_class = ProductSerializer
	permission_classes = [permissions.IsAdminUser, IsStaffEditorPermission]
	authentication_classes = [authentication.SessionAuthentication, authentication.TokenAuthentication]

	def perform_create(self, serializer):
		title = serializer.validated_data.get('title')
		content = serializer.validated_data.get('content') or None
		if content is None:
			content = title
		serializer.save(content=content)
```

As you can see here, we give Session authentication as well as with Token Authentication. Now, let's see how we access the token for dummy user.

```python
import requests

endpoint = "http://localhost:8000/api/auth/"
password = "dja12345"

auth_token = requests.post(endpoint, json={'username': 'dummy', 'password' : password})
print(auth_token.json())
```

or alternatively,
```python
import requests
from getpass import getpass

auth_end_point = "http://localhost:8000/api/auth/"

username = input("What is your Username:\n")
password = getpass("What is your Password:\n")

auth_token = requests.post(endpoint, json={'username': password, 'password' : password})
print(auth_token.json())

```

The endpoint is clear, the sending data via ```json={}``` is clear. I believe everything is make clear now. The token access can be only possible with ___POST___ method of HTTP request.

There is another way to make your token more secure (or it claims). However, I didn't find that much useful, if interested check in video between @2:52:00 - @2:56:00.

>> Okay now I will going to add couple word to this section because at that time I didn't quite get it. Since it's been a quite time for me to use JWT, I understand in much better way. We can change the Keyword for token. In my standard usage, JWT uses Bearer. That parts shows you that.

___OQN___: The token doesn't have expiration by default. You may want to check that info as well. @2:57:00

<hr>

Until that point, I was working at one virtual environment. However now, I am working from another computer. That's why do not follow this section as application. But you can follow for learning more.

<hr>

> Default Django Rest Framework Settings

Here's the [link] of full documentation if you need it. We can put some default lists that contains some classes. With those, you can add default permission and/or authentication to your project if you want it too. In order to make that work, open your __\<project_folder>/<project_name>/<project_name>/settings.py__ and insert a dictionary like this.

```python
REST_FRAMEWORK = {
	'DEFAULT_AUTHENTICATION_CLASSES': [
		'rest_framework.authentication.SessionAuthentication',
		'api.authentication.TokenAuthentication',
	],
	'DEFAULT_PERMISSION_CLASSES': [
		'rest_framework.permission.IsAuthenticatedOrReadOnly'
	]
}
```

In order to import the classes, we are not going to import the module itself, ___but we will provide the path for it___. ~~I am not sure if I was following the course directly, but I am going to explain the codes like what I ment by them.~~ Let's dive into list of ```DEFAULT_AUTHENTICATION_CLASSES```.

```python
'DEFAULT_AUTHENTICATION_CLASSES' : [
	#path_to_app. #path_to_python_file_name. #the_class/function_name
]
```

So the structure of added/desired classes will be identified as such.

> Using Mixins for Permissions

In usage, let's assume that you mixed permission that you using for your application. For that, let's create related python file first.

__\<application_name>/mixins.py__
```python
from rest_framework import permissions
from .permissions import IsStaffEditorPermission

class StaffEditorPermissionMixin():
	permission_classes = [permissions.IsAdminUser, IsStaffEditorPermission]
```

with that applied to a folder, you can use your mixin while you creating your class like ```class RandomClassView(RandomMixin)```.
Also, you can create variety mixin for other things as well like, querysets, serializer_classes, lookup_fields, etc.