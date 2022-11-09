## Django Authentication and Authorization

This is not primitive django example. Instead it's strict to one concept only called (Authentication and Authorization). The video [link](https://www.youtube.com/playlist?list=PL-51WBLyFTg2vW-_6XBoUpE7vpmoR3ztO) need to be checked for full version.<br>
For the environment, I will create seperate environment
<hr>

Setup from console
```console
$ sudo apt install python3-vitualenv
$ virtualenv -p python3 .
$ mkdir src
$ source bin/activate
(venv)$ pip install django
(venv)$ cd src
(venv)$ django-admin startproject <projectName> .
```
This is a basic setup that we store every file inside the src folder. Since we will handle the SQLite3 Database, I also installed the workbench for it.

```$sudo apt install sqlitebrowser```

The whole video series cover lot's of basics, that's why I am not going to write every detail of it. I will cover the info that I need, feel free to check if you need more.

___video 6___ and ___video 7___ is covering one-to-many and many-to-many relationship for DB. That might be important for future explanations.<br>
___video 8___ covering the querying from DB.<br>

```python
#<variable-name> = <model-name>.<model-object-attribute>.<method>
queryset = Customer.objects.all()
```

> User specified webpage

That's a new technique that I saw first in this place.<br>
Now, we are going to deal with this problem as follow. Getting information for customer

__urls.py__
```python
urlpatterns=[path('customer/<str:pk_test>', views.relativefoo, name='customer'),]
```

__views.py__
```python
def relativefoo(request, pk_test):
    customer = Customer.object.get(id=pk_test)
    orders = customer. 
    context = {'customer':customer}
    return render(request, 'accounts/customer.html', context)
```

Inside the html, you can call via

```django
<a href="{% url 'customer' customer.id%}"></a>
```

___video 11___ has CRUD functionallaty.<br>
![Picture](/Pics/9.png)

___video 12___ contains multiple input handling at the same time (Inline formset)

> Filter from table search

___video 13___ contains filtering for search database models. 

> Login and Register File

It's fairly complicated compared to everything I saw so far. First, I will look at the register-page

__views.py__
```python
def registerpage(requests):
	form = UserRegister()
	if requests.method == 'POST':
		form = UserRegister(requests.POST)
		if form.is_valid():
			form.save()
	context={ 'form' : form }
	return render(requests, 'accounts/register.html', context)
```

__register.html__
```django
{% extends 'accounts/base.html' %}

{%block page-head-title%}
register
{%endblock page-head-title%}

{% block content1 %}
<h2>Register</h2>
<hr>
<form method="POST" action="">
	{% csrf_token %}
	<table cellpadding="2" style="border: 0px;" >
			{%for field in form%}
		<tr style="border:0px">
				<td style="border:0px; text-align:right">
					{{field.label}}
				</td>
				<td style="border:0px; text-align:center;">
					{{field}}
				</td>
		</tr>
        {% endfor %}
        <tr>
            <td colspan="2" style="text-align:center; border: 0px;">
                <input value="Create User" type="submit" name="Create User">
            </td>
        </tr>
	</table>
</form>
{% endblock content1 %}
```

__forms.py__
```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class UserRegister(UserCreationForm):
	class Meta:
		model = User
		fields = ['email', 'username', 'password1', 'password2']
```

If you put ```{{form.errors}}``` inside the html, you able to see error. Suggest that put under the button/input etc.<br>
There are two important link that we should look in this register file check. First is [UserCreationForm fields](https://docs.djangoproject.com/en/4.1/topics/auth/default/#django.contrib.auth.forms.UserCreationForm) and [Adding Erro Messages](https://docs.djangoproject.com/en/4.1/ref/contrib/messages/#adding-a-message). Message sending is explained in the ___video 14___.

I set the login screen with another application. That's why there might some different to original example. However, it won't be an issue that much.

__urls.py__
```python
from django.contrib import admin
from django.urls import path, include
from django.conf.urls.static import static
from django.conf import settings
from .views import homepageview, loginpageview, profilepageview, filepageview,logoutfoo

urlpatterns = [
	path('', homepageview, name='home'),
	path('login/', loginpageview, name='login'),
	path('profile/', profilepageview, name='profile'),
	path('file/', filepageview, name='file'),
	path('logout',logoutfoo, name='logout'),
]
```

I set an login, home, profile and file page. However, logout is not exactly page. It's a function that makes you logout as user, then redirect to your home page.

views.py
```python
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from django.contrib.auth import authenticate, login, logout

def loginpageview(request):
	if request.user.is_authenticated:
		return redirect('home')
	else:
		if request.method == 'POST':
			username = request.POST.get('username')
			password =request.POST.get('password')

			user = authenticate(request, username=username, password=password)

			if user is not None:
				login(request, user)
				return redirect('home')
		context={}
		return render(request, 'login.html', context)

@login_required(login_url='login')
def logoutfoo(request):
	logout(request)
	return redirect('login')
```

From here, first look at the login. There is a three step that we need to explain. I didn't send any form via context. That's why you see the context at the end of page. We, however, can send fields via form inside the context. That's not the case in this example.
* First we check if the user authenticated or not.
* (Y) -> Just redirect the home without doing anything.
* (N) -> We are render and send login page.
	* If post method called. then we are getting the information. as you can see in the username and password. the '\<name inside the tag>' is captured via post method. Then we called authenticate method to check if the credentials match up.
		* If credentials match up, we login via login function and redirect ourselves to home page.

Logout function can only callable when you are login already. It checked via login_reqired(). The parameters login_url is sended you if you're not logged in already.<br>
the request is already capture the user data that you logged in, that's why we do not use complex code in here.<br>

__login.html__
```django
{% extends 'base.html' %}
{%load static%}
{%block title-content%}Login{%endblock title-content%}
    {% block add-css %}
    <!-- Bootstrap CSS File -->
    <link href="{% static 'css/bootstrap.css' %}" rel="stylesheet" />
    <link rel="stylesheet" href="{% static 'css/login.css' %}" />
    {% endblock add-css %}


{%block body-content%}  

        {%block navbar%}
        {% include 'navbar.html' %}
        {%endblock navbar%}

    <div class="container">
      <div class="row content">
        <div class="col-md-6 mb-3">
          <img
            src="{% static 'img/hand-drawn-flat-design-container-ship_23-2149157494.png' %}"
            alt="image"
            class="img-fluid"
          />
        </div>
        <div class="col-md-6">
          <h3 class="signin-text mb-3">Sign In</h3>
          <form method='POST' action=".">
            {% csrf_token %}
            <div class="form-group">
              <label for="username">Username</label>
              <input type="text" name="username" class="form-control" />
            </div>
            <div class="form-group">
              <label for="password">Password</label>
              <input type="password" name="password" class="form-control" />
            </div>
            <div class="form-group form-check">
              <input
                type="checkbox"
                name="checkbox"
                class="form-check-input"
                id="checkbox"
              />
              <label class="form-check-label" for="checkbox">Remember Me</label>
            </div>
            <button class="btn btn-class">Login</button>
          </form>
        </div>
      </div>
{%endblock body-content%}
```

NOTE: The CSS and JS part are written by one of my college [Enes Günaçtı](https://github.com/enesgunacti). Thanks to his work, the page has better look in it. However, we should focus on the lines

```html
<div class="form-group">
	<label for="username">Username</label>
	<input type="text" name="username" class="form-control" />
</div>
<div class="form-group">
	<label for="password">Password</label>
	<input type="password" name="password" class="form-control" />
</div>
```

As you can see, the username and password data sended via __name__ attributes inside the tags.

> Session Timeout and Expiration Browser

Firslty, I am going to take a look at the [time-out](https://pypi.org/project/django-session-timeout/) for every user.

```console
(venv)$ pip install django-session-timeout
```

In __settings.py__
```python
MIDDLEWARE = [
    # ...
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django_session_timeout.middleware.SessionTimeoutMiddleware',
    # ...
]

SESSION_EXPIRE_AFTER_LAST_ACTIVITY = True
SESSION_EXPIRE_SECONDS = 300                # In Second
SESSION_TIMEOUT_REDIRECT = 'login'
```

This is overall settings for whole django web app. After we set that, we could add adinitional expiration if the browser closed without logged in as ___Remember Me___, it need to be logged in again. For that we need to check the [link](https://docs.djangoproject.com/en/4.1/topics/http/sessions/#django.contrib.sessions.backends.base.SessionBase.set_expiry).<br>
Basically, I need to change two thing inside the __settings.py__. It's already comes with default, but it may not in your cases. Here's the configuration that you have to handle.

```python
MIDDLEWARE = [
    # ...
    'django.contrib.sessions.middleware.SessionMiddleware',
    # ...
]

INSTALLED_APPS = [
    #...
    'django.contrib.sessions',
    #...
]
```

If these lines aren't comes with default, you may need to migrate your project.