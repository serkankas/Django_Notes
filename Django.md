## Notes from the Full Course

The full course video in the [Link](https://www.youtube.com/watch?v=F5mRW0jo-U4&t=1996s).<br>
The context may have missing or additional part accordingly. Along with the way I add/remove topics according to my needs. Feel free to do your own research according to your own needs.

> Installation

Written in Desktop/ThingsHaveBeenDone

> activate the venv
```console
$ source bin/activate
```

> deactivate the venv
```console
(DjangoTest)$ deactivate
```

> Creating new project
```console
(DjangoTest)$ django-admin startproject <project-name> <project-path>
```

> Running Virtual Environment in built-in server
```console
(DjangoTest)$ python manage.py runserver
```

> Setting up the Sublime with Working Directory

Project > Add Folder to Project...<br>
Select the file (without open it)

> Setting File inside the Django Project

__/home/\<username>/\<path-to-project>/\<project-name>/settings.py__ 

> Migrate project
```console
(DjangoTest)$ python manage.py migrate
```

> Creating Super user for Django
```console
(DjangoTest)$ python manage.py createsuperuser
```

> Login from superuser in website.

__In Browser__, search for ___http://\<ip_address>:<port_number_default_8000>/admin/___<br>
It will lead you to login page. The superuser that you create can access the admin page through __INSTALLED_APP__ ~~admin & auth~~ that you can see inside the __settings.py__.

> Creating the App for Django

Before write the relative codes, let me explain this. App inside the django is used for small portion of code like microservices or data-model etc.<br>
With that being sad, that's how you do it in code vice.
```console
(DjangoTest)$ python manage.py startapp <application_name>
```

![Picture](/Pics/1.PNG "File manager after created app")

As you can see above, we create __profiles__ app. To have a Database-model for that app, we are going to change/modify __models.py__.
```python
from django.db import models

# Create your models here.
class Profile(models.Model):
	fullname = models.TextField()
	mail = models.TextField()
	online_status = models.BooleanField()
```

We are going to add our app into the ___INSTALLED_APP___ inside the __settings.py__
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # Own Apps
    'profiles',
]
```

you will use coma at the end eventhough that seems conflict with python.<br>
After that, make migration.
```console
(DjangoTest)$ python manage.py makemigrations
(DjangoTest)$ python manage.py migrate
```

These two lines should be used for anything change after database-model.<br>
Then, we should change __admin.py__ accordingly. ~~__admin.py__ file is located same as __models.py__~~.

```python
from django.contrib import admin

# Register your models here.
from .models import Profile

admin.site.register(Profile)
```
After this step, you should able to see the Product section inside the website.

> Adding New Object inside the Shell of Django

```console
(DjangoTest)$ python manage.py shell
```

After that, we access the shell

```python
>>> from profiles.models import Profile
>>> Profile.objects.all()
>>> Profile.objects.create(fullname='NS', mail='enes@test.com', online_status=False)
```

to exit from shell
```python
>>> exit()
```

> Adding New Field

To start over, delete every file located under __migrations__ directory except __\_\_init\_\_.py__. Also, delete the sql-database.

You may want to look at the [field-types](https://docs.djangoproject.com/en/4.1/ref/models/fields/#field-types) that can be used inside the models. I will change the model and it will look like this

```python
from django.db import models

# Create your models here.
class Profile(models.Model):
	fullname = models.CharField(max_length = 64)
	bio = models.TextField()
	mail = models.TextField()
	age = models.IntegerField()
	online_status = models.BooleanField()
	achievement = models.CharField(max_length = 64)
```

After that, migrate and create a new superuser (since we create the new one).

> Creating Homepage

First thing first, we create a app for __pages__. We design the __views.py__ under the __pages__ directory.
```python
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.
def homepage_view(*args, **kwargs):
	return HttpResponse("<h1>Hello World</h1>")
```

After changing __views.py__, we change our __urls.py__ under the project file. we add two line codes as follow

1. from pages import views.

This section will be added the library section.<br>
inside the urlpatterns, we will add this

1. path('', views.homepage_view, name='home'),

another way to do it is

```python
from django.contrib import admin
from django.urls import path

from pages.views import homepage_view 

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', homepage_view, name='home'),
]
```

Sligthly different way, but it's same result.

> Django Templates 

We will change the __views.py__ inside the __pages__ directory

```python
from django.shortcuts import render

# Create your views here.
def homepage_view(request, *args, **kwargs):
	return render(request, "home.html", {})
```

We need to create a directory that can be store all the html files in it. Easier to do with, inside the src folder. We create a folder __templates__ under the src.

![Picture](/Pics/2.PNG "Templates Directory Created")

We create __home.html__ file inside the __template__ directory. However, __template__ folder doesn't shown/set inside django. In order to make that, we have to change the __settings.py__ inside the project.<br>
Find the ___TEMPLATES___ section inside the __settings.py__. Inside that section, there is a place called __'DIRS'__, we have to tell django where our templates located inside that folder.
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, "templates")],
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
As you can see here, we implement the __DIRS__ for all OS with functions.

> Django Templates Engine Basics

With help of django engine, you can import/inherit base-model html website. And put some blocks inside that base. So that, you can change desired part accordingly.

__base.html__
```django
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<title>Test 4 Django Server</title>
</head>
<body>
		{% block content1 %}
		replace with content1
		{% endblock content1%}

		<hr>
		<p> Imported place from base </p>

		{% block content2 %}
		replace with content2
		{% endblock content2 %}
</body>
</html>
```
Now we will replace the __content1__ and __content2__ blocks inside our __home.html__ file

__home.html__
```django
{% extends "base.html"%}

{% block content1 %}
<h1>Home Page</h1>
<p>Testing paragraph</p>
{% endblock content1 %}

{%block content2%}
<p>Testing paragraph 2</p>
{%endblock content2%}
```

Now let's have a look at the website.

![Picture](/Pics/3.PNG "Inheritted Web page")

As we can see in here our home page imported from base page. If I create another webpage, same structure can be taken/usable.

> Include Template Tag

This can be explained in certain simple way. Let's assume that you want to have navbar or footer tag. You want to implement those but not inside the __base.html__. You want them to have their own navbar.html or footer.html. Let's continue to last example and add navbar with include tag.<br>
I did change __base.html__ slightly, so now it's look like this

```django
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<title>Test 4 Django Server</title>
</head>
<body>
		{% block content1 %}
		replace with content1
		{% endblock content1%}

		{% include 'navbar.html' %}

		<hr>
		<p> Imported place from base </p>

		{% block content2 %}
		replace with content2
		{% endblock content2 %}
</body>
</html>
```

After that, I create a __navbar.html__ that look like this.

```django
<nav>
	<ul>
		<li>Brand</li>
		<li>Contact</li>
		<li>Info</li>
	</ul>
</nav>
```

Since we use relative path for include tag ~~I assume it's relative~~, I put the navbar inside the __templates__ folder.

![Picture](/Pics/4.PNG)

The end result will look like this.

![Picture](/Pics/5.PNG)

> CSS Implementation

This is one section of another [course](https://www.youtube.com/watch?v=PtQiiknWUcI&t=14005s). Static file implementation is also important in here since the CSS, Images, etc. will be implemented. This section a bit more complex than the other parts. I will dive into that part later on.

> Passing Context and Get Data From It

As we can guess, having beautiful website is cool. But if we have interaction with database and stuff, it would be better.<br>
Now, let's remember how our __homepage_view()__ function was called.
```python
def homepage_view(request, *args, **kwargs):
	return render(request, "home.html", {})
```

As we can see there, we send a empty dictionary to render function as third parameter. Let's make small dictionary and send to our __home.html__. After that, put that values inside the home page.

```python
def homepage_view(request, *args, **kwargs):
	my_dic = {
		"test_string" : "Test String 123",
		"test_int" : 123456789,
		"test_list" : [1, 2, 3, 4, 5, 6]
	}
	return render(request, "home.html", my_dic)
```

The __home.html__ file will also change. This is the last version of the __home.html__ file.
```django
{% extends "base.html"%}

{% block content1 %}
<h1>Home Page</h1>
<p>Testing paragraph</p>
<p> {{ test_string }} </p>
<p> {{ test_int }} </p>
{% endblock content1 %}

{%block content2%}
<p>Testing paragraph 2</p>
{%endblock content2%}
```

As we can see in here, the sended parameters/keys can be called as such ___{{ key-name }}___. The result should look like this.

![Picture](/Pics/6.PNG)

Also, don't worry about having the JSON data. If you have JSON file/data, you can convert from json to dictionary easily. Here's the sample [Link](https://www.geeksforgeeks.org/convert-json-to-dictionary-in-python/). Also, you can handle all the data type with simple-python. There is no complexity to convert your ~~any simple~~ data format to dictionary.

> For Loop

In last example, I didn't show the List directly. That's a built-in tag section inside the official website [link](https://docs.djangoproject.com/en/4.1/ref/templates/builtins/).<br>
Here's the example that I came up with.

__views.py__
```python
from django.shortcuts import render

# Create your views here.
def homepage_view(request, *args, **kwargs):
	my_dic = {
		"test_string" : "Test String 123",
		"test_int" : 123456789,
		"test_list" : ["ii", "iii", "iiii", "i", "ii", "iii"]
	}
	return render(request, "home.html", my_dic)

def index_view(request, *args, **kwargs):
	return render(request, "index.html", {})
```

__home.html__
```django
{% extends "base.html"%}

{% block content1 %}
<h1>Home Page</h1>
<p>Testing paragraph</p>
<p> {{ test_string }} </p>
<p> {{ test_int }} </p>
{% endblock content1 %}


{%block content2%}
<p>Testing paragraph 2</p>
<ul>
{% for test in test_list %}
	<li>{{forloop.counter}} - {{test}}</li>
{% endfor %}
</ul>
{%endblock content2%}
```
The part that we need to focus on this example is

```django
<ul>
{% for test in test_list %}
	<li>{{forloop.counter}} - {{test}}</li>
{% endfor %}
</ul>
```

The __{{forloop.counter}}__ is built-in tag just wrote the index of the for loop. That's completely straightforward for loop that doesn't need any explanation.

> Using Condition in Template

I am not going to implement the conditional state in my website but here is the quick recap

```django
{% if <variable_name> <conditional operator> <comparison>%}
... some code ...
{% elif <other conditional statment> %}
... some code ...
...
...
...
{% else %}
... some code ...
{% endif %}
```

More details can be checked from [built-in tag](https://docs.djangoproject.com/en/4.1/ref/templates/builtins/) in Django.

> Template Tags and Filters

We already use and explain built-in tags. However, there is another concept called filters. Thats ~~as far as I understood~~ a functionlike programs that you can use in your program. In sample, I am going to add 5 to sended variable called ___add_me_5___.

```python
{{add_me_5|add:"5"}}
```

As you can see here, I used built-in filter called __add__ and send parameter of __"5"__ since it's my desired value. You can stack filter on top of each other. I am not sure if the use case that much valuable, but you can...

> Render Data from the Database with a Model

Now we will deal with multiple view pages at the same time. Everything related to Data-model for DB, will handle under that section. In our example, we already create an app (data-model) called ___profiles___. We open __views.py__ located under profiles folder.

__profiles/views.py__
```python
from django.shortcuts import render

from .models import Profile
# Create your views here.
def profile_detail_view(request):
	obj = Profile.objects.get(id=1)
	print(obj.mail)
	print(obj.fullname)
	context = {
		'object' : obj
	}
	return render(request, "profile/detailed.html", context)
```

Now, we will create new folder (profile folder) under the __templates__ folder. Inside that __profile__ folder, we will create __detailed.html__ file.

__templates/profile/detailed.html__
```django
{% extends 'base.html' %}

{% block content1 %}
<h2>
	Hello, {{ object.fullname }}
</h2>
<p>
	Mail address: {{ object.mail }}
</p>
{% endblock content1 %}
```

After we configured the all steps, we need to give path to our __urls.py__ file.

```python
from django.contrib import admin
from django.urls import path

from pages.views import homepage_view, index_view
from profiles.views import profile_detail_view

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', homepage_view, name='home'),
    path('index/', index_view, name='index'),
    path('profile/', profile_detail_view),
]
```

__\<website>/profile/__<br>
![Picture](/Pics/7.PNG "Profile web-page")

data<br>
![Picture](/Pics/8.PNG)

> Django Model Forms

We are going to create __forms.py__ under the __profiles__ folder. (The one used for templates.)

__forms.py__<br>
```python
from django import forms

from .models import Profile

class ProfileForm(forms.ModelForm):
	class Meta:
		model = Profile
		fields = [
			'fullname',
			'bio',
			'mail',
			'age',
			'online_status',
			'achievement'
		]
```

__profiles/views.py__
```python
from django.shortcuts import render

from .forms import ProfileForm

from .models import Profile
# Create your views here.
def profile_detail_view(request):
	obj = Profile.objects.get(id=1)
	print(obj.mail)
	print(obj.fullname)
	context = {
		'object' : obj
	}
	return render(request, "profile/detailed.html", context)


def profile_create_view(request):
	form = ProfileForm(request.POST or None)
	if form.is_valid():
		form.save()

	context = {
		'form' = form
	}
	return render(request, "profile/create.html", context)
```

After create the view function, we will handle the product_create.html

__create.html__<br>
```django
{% extends 'base.html' %}

{% block content1 %}
{% endblock content1 %}

{% block content2 %}
<form method="post"> {% csrf_token %}
	{{ form.as_p }}
	<input type="submit" value="Save">
</form>
{% endblock content2 %}
```

URLs changed accordingly.
```python
from django.contrib import admin
from django.urls import path

from pages.views import homepage_view, index_view
from profiles.views import profile_detail_view,profile_create_view

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', homepage_view, name='home'),
    path('index/', index_view, name='index'),
    path('profile/', profile_detail_view),
    path('create/', profile_create_view),
]
```

the profile_create_view function changed accordingly.
```python
from django.shortcuts import render

from .forms import ProfileForm

from .models import Profile
# Create your views here.
def profile_detail_view(request):
	obj = Profile.objects.get(id=1)
	print(obj.mail)
	print(obj.fullname)
	context = {
		'object' : obj
	}
	return render(request, "profile/detailed.html", context)


def profile_create_view(request):
	form = ProfileForm(request.POST or None)
	if form.is_valid():
		form.save()
		form = ProfileForm()

	context = {
		'form' : form
	}
	return render(request, "profile/create.html", context)
```

> Raw HTML & Pure Django Form

We are going to handle the create with additional form. Let's dive into the codes.<br>
Inside the __profiles__ folder, we opened the __forms.py__ file and redesign accordingly.

__profiles/forms.py__
```python
from django import forms

from .models import Profile

class ProfileForm(forms.ModelForm):
	class Meta:
		model = Profile
		fields = [
			'fullname',
			'bio',
			'mail',
			'age',
			'online_status',
			'achievement'
		]


class RawProductForm(forms.Form):
	fullname = forms.CharField(max_length = 64)
	bio = forms.CharField(max_length = 64)
	mail = forms.CharField(max_length = 64)
	age = forms.IntegerField()
	online_status = forms.BooleanField()
	achievement = forms.CharField(max_length = 64)
```

We create __RawProductForm__ class from zero. That's almost same with the model that we create before. The difference is there is no text-field inside the form fields. That's why we switched with charfield instead.

__profiles/models.py__
```python
class Profile(models.Model):
	fullname = models.CharField(max_length = 64)
	bio = models.TextField()
	mail = models.TextField()
	age = models.IntegerField()
	online_status = models.BooleanField()
	achievement = models.CharField(max_length = 64)
```

Now we will handle this inside the __views.py__ file.

__profiles/views.py__
```python
from django.shortcuts import render

from .forms import ProfileForm, RawProductForm

from .models import Profile
# Create your views here.
def profile_detail_view(request):
	obj = Profile.objects.get(id=1)
	print(obj.mail)
	print(obj.fullname)
	context = {
		'object' : obj
	}
	return render(request, "profile/detailed.html", context)


def profile_create_view(request):
	form = RawProductForm()
	if request.method == "POST":
		form = RawProductForm(request.POST)
		if form.is_valid():
			Profile.objects.create(**form.cleaned_data)
	context = {
		'form' : form
	}
	return render(request, "profile/create.html", context)
```

HTML link for just reminder.<br>
__create.html__
```django
{% extends 'base.html' %}

{% block content1 %}
{% endblock content1 %}

{% block content2 %}
<form action='.' method="post"> {% csrf_token %}
	{{ form.as_p }}
	<input type="submit" value="Save">
</form>
{% endblock content2 %}
```

Now all these codes have total internal validation inside the django.<br>
1. {% csrf_token %} inside the html
1. form.is_valid() function inside the views.py
1. forms field inside the forms.py

All these section creates validation and authentication inside it. Also we use POST function for our website, which is more convinient with built-in functions.,

> Form Widgets

This is fairly simple concept. This one is used for models/forms shown data.

```python
title = forms.CharField(parameters)
```
Let's assume that this is the case for our usage case. Now, we can change the parameters/attributes of this function accordingly. Details can be search from [official website](https://docs.djangoproject.com/en/4.1/ref/forms/widgets/).

> Form Validation Methods

This validation programmed inside the __forms.py__ file. It should be handle like this.

```python
def clean_<the form that you want to manipulate>(self, *args, **kwargs):
	#handling
	#manipulating
	#etc...
```

Let's move from the example in our example.

```python
from django import forms

from .models import Profile

class ProfileForm(forms.ModelForm):
	mail = forms.EmailField()
	class Meta:
		model = Profile
		fields = [
			'fullname',
			'bio',
			'mail',
			'age',
			'online_status',
			'achievement'
		]

	def clean_mail(self, *args, **kwargs):
		mail = self.cleaned_data("mail")
		if not "@" in mail:
			return mail
		else:
			raise forms.ValidationError("This is not a valid mail!")


class RawProductForm(forms.Form):
	fullname = forms.CharField(max_length = 64)
	bio = forms.CharField(max_length = 64)
	mail = forms.CharField(max_length = 64)
	age = forms.IntegerField()
	online_status = forms.BooleanField()
	achievement = forms.CharField(max_length = 64)
```

The function is shown above. We configured the mail form. Now without having ___@___ sign, the mail form will throw an error.<br>
The ProfileForm has mail field shown above the mail form. inside the meta class we re-process the mail field. At the clean_mail function, we declare a validation for mail addressses.

> Upload File

I create an app called __file_handle__<br>
Inside the __file_handle__ folder, I open __views.py__ file and change accordingly.<br>
I also create __forms.py__ file inside the __file_handle__ folder.<br>
In addition to that, I create __file_handle__ folder inside the templates folder and create an __fileHandle.html__ file inside of it.<br>
I also write the models-part accordingly, inside the __file_handle__ folder __models.py__. The file handling side can be written inside the different app, or we can simply use seperate app for that.<br>
I modify the __urls.py__ and __settings.py__ inside the main file.
Here is the files and modifications.

__forms.py__
```python
from django import forms

class UploadFileForm(forms.Form):
	file = forms.FileField()
```

__models.py__
```python
from django.db import models

# Create your models here.
class Ship(models.Model):
	VDR_File = models.FileField(null=True)
```

__fileHandle.html__
```django
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<title>Uploading File</title>
</head>
<body>
	<form action="." method="post" enctype="multipart/form-data">
		{% csrf_token %}
		<p>
			{{ form.file }}
		</p>
		<p>
			<input type="submit" value="Upload">
		</p>
	</form>
</body>
</html>
```
__views.py__
```python
from django.shortcuts import render
from django.shortcuts import redirect
# Create your views here.

from django.http import HttpResponse
from .forms import UploadFileForm
from .models import Ship

def file_handling(request):
	if request.method == 'POST':
		form = UploadFileForm(request.POST, request.FILES)
		file = request.FILES['file']
		vdr_data = Ship.objects.create(VDR_File = file)
		vdr_data.save()
		return redirect('home')
	else:
		form = UploadFileForm()
		return render(request, "file_handle/fileHandle.html", {'form' : form})
```

addition to the __settings.py__
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # Own Apps
    'profiles',
    'file_handle',		# Adding Handler!
]

MEDIA_URL = '/vdr_files/'
MEDIA_ROOT = os.path.join(BASE_DIR, "vdr_files")
```

modification inside the __urls.py__
```python
# file upload
from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', homepage_view, name='home'),
    path('index/', index_view, name='index'),
    path('profile/', profile_detail_view),
    path('create/', profile_create_view),
    path('upload/', file_handling),
] + static(settings.MEDIA_URL, document_root = settings.MEDIA_ROOT)
```

Now, let's dive into the what we have done.
__forms.py__, __models.py__ and __fileHandle.html__ is fairly straight-forward. I just skip those parts.<br>
In __views.py__, we write simple POST/GET handling function.<br>
We add our own app inside the ___INSTALLED_APPS___ array, we also create the app with 

```console
(Django)$ python manage.py startapp file_handle
```

Then we add two different variable called _MEDIA_URL_ and _MEDIA_ROOT_ accordingly.<br>
Lastly, we import our file directories inside the __urls.py__ to urlpatterns as in shown inside the codes. Then we type the migration with

```console
(DjangoTest)$ python manage.py makemigration
(DjangoTest)$ python manage.py migrate
```

Hopefully, that should worked!

___ADDITION___: If you want to see the model under the admin page, you need to register the model inside the admin panel. Simply open the __admin.py__ under the __file_handle__ folder.

```python
from django.contrib import admin

# Register your models here.
from .models import Ship

admin.site.register(Ship)
```

> Download File

Firstly, there are couple way to download the file as you wish. I create an "a" tag in html and download inside the tag with attribute. People follow different way to have access and download from it. However, I follow the way that I understand and make sense to my situation. With that being sad, let's dive into it.

First, I create a path for my download link.

__urls.py__
```python
from file_handle.views import file_handling, show_files

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', homepage_view, name='home'),
    path('index/', index_view, name='index'),
    path('profile/', profile_detail_view),
    path('create/', profile_create_view),
    path('upload/', file_handling),
    path('download/', show_files),
]

urlpatterns += static(settings.MEDIA_URL, document_root = settings.MEDIA_ROOT)
```

As you can see here, I crete download path inside the __urls.py__, and bind to the function called show_files. I use same __views.py__ file with last topic (upload file).

addition to the __views.py__
```python
def show_files(request):
	if request.method == 'GET':
		file_list = os.listdir(settings.MEDIA_ROOT)
		context = {
		'media_url' : settings.MEDIA_URL,
		'my_list' : file_list,
		}

	else:
		context = {}
	return render(request, "file_handle/test.html", context)
```

This file is also fairly straight-forward. We create a list of file that we want to achieve that can be iterated as wish. Sended via MEDIA_ROOT for os. function. MEDIA_ROOT gives an absolute path for file control. However, MEDIA_URL should be used inside the URLS. That's why we send URL inside to HTML file. As I mentioned before, the file download handled under the HTML.

__test.html__
```django
{% extends 'base.html' %}

{% block content1 %}

<ul>
{% for file in my_list %}
<li><a href="{{media_url}}/{{file}}" download>
	{{file}}
</a></li>
{% endfor %}
</ul>

{% endblock content1 %}


{%block content2%}
{%endblock content2%}
```

The files are download link created via ___{{media_url}}/{{file}}___ reference. However, we send all of the list of files with my_list, which is get individiual by for loop.

<hr>

Following topics will be covered inside those files.
|Filename | Topic
|-- |--
| [DjangoAA.md](/DjangoAA.md) | Authentication
| [DjangoAndReact.md](/DjangoAndReact.md) | React project inside Django
| [DjangoRestAPI.md](/DjangoRestAPI.md) | Rest API with Django Application