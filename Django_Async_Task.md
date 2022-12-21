## This file will contain the Asynchronous Tasks and Websocket implementation for Django

The main course in the [Link](https://www.youtube.com/playlist?list=PLOLrQ9Pn6caz-6WpcBYxV84g9gwptoN20). However, I use another links as well. The learning process is going fairly complex.<br>
Honestly, the asynchronous tasks can work with RabbitMQ without having any problem. However, channels (websocket implementation) doesn't support anything but Redis.That may lead us to use Redis. IMHO, That's not my first choice, but channels_rabbitmq has implementation issues. I couldn't fixed it yet.

> Introduction

Technologies:
* Celery
* RabbitMQ
* Django

[REDIS vs RabbitMQ](https://www.educba.com/rabbitmq-vs-redis/)

Celery
* Task(Queue)/Process Manager
* Execute processes in different thread (on demand / periodically)
* Queue Tasks

Message Broker
* Django -> Task Messages -> Message Broker
* RabbitMQ is alternative of Redis and/or Kafka
 
> Configurations

Installation of celery and RabbitMQ Server
```console
(venv)$ pip install celery
$ sudo apt-get install rabbitmq-server
```

Then Start and Check the status of RabbitMQ
```console
$ sudo systemctl start rabbitmq-server
$ sudo systemctl status rabbitmq-server
```

Now, to create celery configurations, we open the main project file (where the settings.py is installed) and create the __celery.py__ inside.

__celery.py__
```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'website.settings')

app = Celery('website')

app.config_from_object('django.conf:settings', namespace='CELERY')

app.autodiscover_tasks()
```

My project called ___website___, you can change the places called ___website___ to your project name respectfully. Now, the celery.py searching for the __tasks.py__ which is not default file that comes with your project/app. We want to create __tasks.py__ inside our app.

For my purposes, I create a simple sleep task like this

__tasks.py__
```python
from __future__ import absolute_import, unicode_literals
from celery import shared_task
from time import sleep
from random import randint

@shared_task()
def sleepSomeAmountOfTime():
	for i in range(10):
		temp_time = randint(0,5)
		print(f'Going to sleep for {temp_time} second')
		sleep(temp_time)

	del temp_time
	del i
```

Then I run the command
```console
(venv)..<path>/website/$ celery -A website worker -l info
```

When you get the worker running, You can simply access your tasks and make it working via shell. Then you can simply implement in your website.

```console
(venv)$ python manage.py shell
>>> from <app-name>.tasks import <task-name>    # App name will be switched as yours
>>> <task-name>.delay()                         # Task name will be switched as yours
>>> # Free usage eventhough the function has sleep 
```

The output can be seen at celery output page.<br>
Instead of using __delay()__ function, you can use  __apply_async()__ function with countdown sleep time. It can process after certain amount of time that you wish.

```console
>>> # <task-name>.apply_async((<parameter_if_any>), countdown=<desired_time_in_sec>)
>>> # In my example
>>> sleepSomeAmountOfTime.apply_async((), countdown=2)
>>> # Started after 2 second delay. 
```

> WebSocket & Channels

To be clear, I will start from the I am going to follow the [official](https://channels.readthedocs.io/en/stable/tutorial/part_1.html) documentary.

* Install channels and daphne via ```python -m pip install -U channels["daphne"]```
* Set the daphne in INSTALLED_APPS
* Set the __asgi.y__ file.
* Set ___ASGI_APPLICATION___ inside the __settings.py__
* Create a project called mysite
* Create an app called chat
* Delete all the unnecessary files. It should be shown like this.

![Picture](/Pics/10.PNG)

* Create the templates folder and create the directory inside the settings.
* Then create an index html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Rooms</title>
</head>
<body>
    What chat room would you like to enter?<br>
    <input id="room-name-input" type="text" size="100"><br>
    <input id="room-name-submit" type="button" value="Enter">

    <script>
        document.querySelector('#room-name-input').focus();
        document.querySelector('#room-name-input').onkeyup = function(e) {
            if (e.keyCode === 13) {  // enter, return
                document.querySelector('#room-name-submit').click();
            }
        };

        document.querySelector('#room-name-submit').onclick = function(e) {
            var roomName = document.querySelector('#room-name-input').value;
            window.location.pathname = '/chat/' + roomName + '/';
        };
    </script>
</body>
</html>
```
This is also contains some javascript.

If you followed until here, when you start your django via runserver, you should be able to see this output.

```console
$ python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
November 14, 2022 - 13:23:19
Django version 4.1.3, using settings 'mysite.settings'
Starting ASGI/Daphne version 4.0.0 development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

You should pay attention to the part written ```Starting ASGI/Daphne...```. If this part is not written in your output, then you couldn't load your website as ASGI.

* Create a __room.html__ inside your templates folder and paste the relative code.

```django
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Room</title>
</head>
<body>
    <textarea id="chat-log" cols="100" rows="20"></textarea><br>
    <input id="chat-message-input" type="text" size="100"><br>
    <input id="chat-message-submit" type="button" value="Send">
    {{ room_name|json_script:"room-name" }}
    <script>
        const roomName = JSON.parse(document.getElementById('room-name').textContent);

        const chatSocket = new WebSocket(
            'ws://'
            + window.location.host
            + '/ws/chat/'
            + roomName
            + '/'
        );

        chatSocket.onmessage = function(e) {
            const data = JSON.parse(e.data);
            document.querySelector('#chat-log').value += (data.message + '\n');
        };

        chatSocket.onclose = function(e) {
            console.error('Chat socket closed unexpectedly');
        };

        document.querySelector('#chat-message-input').focus();
        document.querySelector('#chat-message-input').onkeyup = function(e) {
            if (e.keyCode === 13) {  // enter, return
                document.querySelector('#chat-message-submit').click();
            }
        };

        document.querySelector('#chat-message-submit').onclick = function(e) {
            const messageInputDom = document.querySelector('#chat-message-input');
            const message = messageInputDom.value;
            chatSocket.send(JSON.stringify({
                'message': message
            }));
            messageInputDom.value = '';
        };
    </script>
</body>
</html>
```

The __views.py__ and __urls.py__ changed accordingly.

__views.py__
```python
from django.shortcuts import render

# Create your views here.
def index(request):
	return render(request, 'index.html')


def room(request, room_name):
	return render(request, "room.html", {"room_name": room_name})
```
__urls.py__
```python
from django.urls import path
from . import views

urlpatterns = [
	path('', views.index, name="index"),
	path("<str:room_name>/", views.room, name='room')
]
```

So far, We create the lobby name and pass through by python/django. The pages are handled as html. However, websocket is not handling yet. In order to handle the websocket, we need to change ASGI configuration, create a consumer and routing properly.

* Create __consumers.py__ inside the chat app and fill with necessary information

```python
import json
from channels.generic.websocket import WebsocketConsumer

class ChatConsumer(WebsocketConsumer):
	def connect(self):
		self.accept()

	def disconnect(self, close_code):
		pass

	def receive(self, text_data):
		text_data_json = json.loads(text_data)
		message = text_data_json["message"]
		self.send(text_data=json.dumps({"message": message}))
```

The Chat Consumer/Subscriber/Member has three function. connect, disconnect and receive. The names are self-explanatory. We need to create a __routing.py__ to make connection between consumer file (websocket) with requests for urls.

* Create a __routing.py__ inside the chat app.

```python
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r"ws/chat/(?P<room_name>\w+)/$", consumers.ChatConsumer.as_asgi()),
]
```

* Configure __asgi.py__ respectfully. 

```python
"""
ASGI config for mysite project.

It exposes the ASGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/4.1/howto/deployment/asgi/
"""

import os

from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.security.websocket import AllowedHostsOriginValidator
from channels.auth import AuthMiddlewareStack
import chat.routing


os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings')

application = ProtocolTypeRouter({
	'http': get_asgi_application(),
	"websocket": AllowedHostsOriginValidator(
		AuthMiddlewareStack(URLRouter(chat.routing.websocket_urlpatterns))
	),
})
```

Since the websocket needs authorization and stuffs, you need to make migration in your project.

* Make migration to your project.
* Run yourserver.

At this point, whatever lobby you logged in, you can send some message and see through the table. However, other people won't able to see it. Which means we need to send the message back to the backend handle it and sended to other people as well. In our example, it calls enabling channel layer. In official website, it uses Redis. ~~However, I am going to use RabbitMQ since it's already set it up on my system.~~ Official Channel layer is stacked up as

* Set the CHANNEL_LAYERS in __settings.py__

```python
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [("127.0.0.1", 6379)],
        },
    },
}
```

The RabbitMQ won't working right now. I set the memory channel layer. This is not proper option hence it's not actually ready for production. It is easy shortcut for testing environment. The CHANNEL_LAYERS should be setted as Redis/RabbitMQ/Kafka or preferred Message Broker or similar structure program.

```python
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels.layers.InMemoryChannelLayer"
    }
}
```

* Change the __consumer.py__ accordingly.

```python
import json

from asgiref.sync import async_to_sync
from channels.generic.websocket import WebsocketConsumer


class ChatConsumer(WebsocketConsumer):
	def connect(self):
		self.room_name = self.scope["url_route"]["kwargs"]["room_name"]
		self.room_group_name = f"chat_{self.room_name}"
		async_to_sync(self.channel_layer.group_add)(self.room_group_name, self.channel_name)
		self.accept()

	def disconnect(self, close_code):
		# Leave room group
		async_to_sync(self.channel_layer.group_discard)(self.room_group_name, self.channel_name)

	# Receive message from WebSocket
	def receive(self, text_data):
		text_data_json = json.loads(text_data)
		message = text_data_json["message"]

		# Send message to room group
		async_to_sync(self.channel_layer.group_send)(self.room_group_name, {"type": "chat_message", "message": message})

	# Receive message from room group
	def chat_message(self, event):
		message = event["message"]

		# Send message to WebSocket
		self.send(text_data=json.dumps({"message": message}))
```

#### What does this code do ?
It has 3 main functions and one addition function.

|Function Name| Function
| --- | ---
| Connect| self.scope gives you output with JSON format. From that JSON, we get the data room_name from kwargs from url_route. We set that Set thet info as room_name. Then we set our room_group_name as chat_<room_name>. The channel_layer is handled iside the WebsocketConsumer. Then accept the connection via self.accept()
| Disconnect| Discard person from channel_layer
| Receive| Load the json data. Then filter the information of _message_ key. ~~I believe this process capture the whatever message we try to send via JavaScript inside the HTML!~~. Send that message to group.
| chat_message| Capture the response from chat_message. Is it captured thanks to function name? or something else?. I need to check the JavaScript codes inside the HTML and Control it.

* We could make the Server as Asynchronous. In order to make that happen, change the consumer.py with async functions and awaits.

~~I believe I could implement my RabbitMQ after this stage~~.<br>
Here is what we have done Change every function as ___async___, give ___await___ keyword to every operation (not the declaration but operation.) and change the websocket as ___AsyncWebsocketConsumer___. Here's the result

```python
import json

from asgiref.sync import async_to_sync
from channels.generic.websocket import AsyncWebsocketConsumer


class ChatConsumer(AsyncWebsocketConsumer):
	async def connect(self):
		self.room_name = self.scope["url_route"]["kwargs"]["room_name"]
		self.room_group_name = f"chat_{self.room_name}"
		await self.channel_layer.group_add(self.room_group_name, self.channel_name)
		await self.accept()

	async def disconnect(self, close_code):
		# Leave room group
		await self.channel_layer.group_discard(self.room_group_name, self.channel_name)

	# Receive message from WebSocket
	async def receive(self, text_data):
		text_data_json = json.loads(text_data)
		message = text_data_json["message"]

		# Send message to room group
		await self.channel_layer.group_send(self.room_group_name, {"type": "chat_message", "message": message})

	# Receive message from room group
	async def chat_message(self, event):
		message = event["message"]

		# Send message to WebSocket
		await self.send(text_data=json.dumps({"message": message}))
```
<hr>
Now I built the setup and going to explain as my best.

Firstly, we use same consumer structure with one additional asynchronous function. I called the function as soon as it gets connection requests. The process handled inside the consumer file.

__consumer.py__
```python
import json
import asyncio
from channels.generic.websocket import AsyncWebsocketConsumer


class ChatConsumer(AsyncWebsocketConsumer):
	async def connect(self):
		self.room_name = self.scope["url_route"]["kwargs"]["room_name"]
		self.room_group_name = f"chat_{self.room_name}"
		self.counter = 0
		await self.channel_layer.group_add(self.room_group_name, self.channel_name)
		await self.accept()
		await self.printLog()
		
	async def disconnect(self, close_code):
		# Leave room group
		await self.channel_layer.group_discard(self.room_group_name, self.channel_name)

	# Receive message from WebSocket
	async def receive(self, text_data):
		text_data_json = json.loads(text_data)
		message = text_data_json["message"]

		# Send message to room group
		await self.channel_layer.group_send(self.room_group_name, {"type": "chat_message", "message": message})

	# Receive message from room group
	async def chat_message(self, event):
		message = event["message"]

		# Send message to WebSocket
		await self.send(text_data=json.dumps({"message": message}))

	async def printLog(self):
		while(True):
			file_op = open("test_file", "r")
			lines = file_op.readlines()
			file_op.close()
			context = ""
			for line in lines:
				context += line
			await self.send(text_data=json.dumps({"message": context}))
			await asyncio.sleep(0.2)
```

___KEY THINGS___: Calling the function asynchronous is important since we have to deal with while function. This function is by itself thread-locking structure. The sleep from time is also thread-locking algorithm. We can set/call the while in a async function and called via await function. If you need more details, you have to search it. Instead of time.sleep, we use also asyncio.sleep to make our asynchronous function not locking itself. The test file is setted inside the project folder. Not main app, not created app. You may want to locate in more suitable place too. It can be set with MEDIA_ROOT and MEDIA_URL. It is already talked inside the main [django](/Django.md) course.<br>
We didn't change anything inside the routing.py

__routing.py__
```python
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
	re_path(r"ws/chat/(?P<room_name>\w+)/$", consumers.ChatConsumer.as_asgi()),
]
```

We activate this with __asgi.py__. So far, we handle every part related with back-end. Now we are going to check the html difference.

```django
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Room</title>
</head>
<body>
    <textarea id="chat-log" cols="100" rows="20"></textarea><br>
    <input id="chat-message-input" type="text" size="100"><br>
    <input id="chat-message-submit" type="button" value="Send">
    {{ room_name|json_script:"room-name" }}
    <script>
        const roomName = JSON.parse(document.getElementById('room-name').textContent);

        const chatSocket = new WebSocket(
            'ws://'
            + window.location.host
            + '/ws/chat/'
            + roomName
            + '/'
        );

        chatSocket.onmessage = function(e) {
            const data = JSON.parse(e.data);
            document.querySelector('#chat-log').value = (data.message + '\n');
        };

        chatSocket.onclose = function(e) {
            console.error('Chat socket closed unexpectedly');
        };

        document.querySelector('#chat-message-input').focus();
        document.querySelector('#chat-message-input').onkeyup = function(e) {
            if (e.keyCode === 13) {  // enter, return
                document.querySelector('#chat-message-submit').click();
            }
        };

        document.querySelector('#chat-message-submit').onclick = function(e) {
            const messageInputDom = document.querySelector('#chat-message-input');
            const message = messageInputDom.value;
            chatSocket.send(JSON.stringify({
                'message': message
            }));
            messageInputDom.value = '';
        };
    </script>
</body>
</html>
```

The actual difference is now, I did change the chatSocket.onmessage function. Instead of adding data on top of previous datas, I set as whatever value comes with it. The reason behind that is, inside the __consumer.py__ I set my printLog file as sending all the file content inside the every each time. If I leave as it is, It will continue to put output at top of the log field. That's not we want it, that's why I change it.

> Turning Celery with Redis

There are slight changes with default Celery application, Now let's see what we did, and how we manage with it.

First, we create Celery instance inside the __tasks.py__
```python
from __future__ import absolute_import, unicode_literals
from celery import shared_task
from time import sleep
from random import randint

@shared_task()
def sleepSomeAmountOfTime():
	for i in range(10):
		temp_time = randint(0,5)
		print(f'Going to sleep for {temp_time} second')
		sleep(temp_time)

	del temp_time
	del i
```

__celery.py__
```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'website.settings')

app = Celery('website')

app.config_from_object('django.conf:settings', namespace='CELERY')

app.autodiscover_tasks()

```

So far, we didn't change anything from the previous example. However, now since we are going to use redis, we will implement broker inside the settings.py

addons in __settings.py__
```python
CELERY_BROKER_URL = os.environ.get("CELERY_BROKER", "redis://localhost:6379")
CELERY_RESULT_BACKEND = os.environ.get("CELERY_BROKER", "redis://localhost:6379")
```

addition to that, in order to trigger our celery tasks, we will add two lines inside the __\_\_init\_\_.py__ (where the settings.py is located.)

```python
from .celery import app as celery_app
__all__ = ("celery_app", )
```

From here, we can simply run as follow

First bash
```console
(venv)$ celery -A <main_project_name> worker -l info
```

Second bash
```console
(venv)$ python manage.py shell
>>> from <app_name>.tasks import <task_name>
>>> <task_name>.delay()
<AsyncResult: ...>
```

When you see the line comes with __\<AsyncResult: ...>__ it means it start to progress behind the worker. It means, at the same time you able to see your program running in first bash where you run the celery worker. 