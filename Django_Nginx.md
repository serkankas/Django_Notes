## This file will contain the NGINX implementation for Django

This is the [link](https://www.youtube.com/watch?v=ZpR1W-NWnp4) that you can follow for your application. With that being sad, let's directly dive into it.

```console
(venv)$ sudo apt update
(venv)$ sudo apt install python3.10-dev
(venv)$ sudo apt install gcc
(venv)$ pip install uwsgi
```

We install specific python-dev. I currently have python 3.10.6 in my virtual environment. In order to check, you can type
```console
(venv)$ python --version
```

In order to start __uwsgi__ we can write

```console
(venv)$ uwsgi --http :8000 --module <project_name>.wsgi
```

But we want to handle the webserver by nginx so we are going to install nginx

```consol
(venv)$ sudo apt install nginx
```

Create a configuration file at __/etc/nginx/sites-avaliable/\<project-name>.conf

__\<project-name>.conf__
```conf
upstream django{ 
        server unix:///home/<project-full-path>/website/website.sock;
}

server { 

        listen          80;
        server_name     <internal-ip-address>/or-website.com;
        charset         utf-8;

        client_max_body_size 75M;

        location /media { 
                alias /home/<project-full-path>/website/media;
        }
        location /static { 
                alias /home/<project-full-path>/website/static;
        }
        location / {
                uwsgi_pass django;
                include /home/<project-full-path>/website/uwsgi_params;
        }
}
```

create __uwsgi_params__ file and write code below.
```
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

Enable configuration file via line 
```console
sudo ln -s /etc/nginx/sites-available/<project-name>.conf /etc/nginx/sites-enabled/
```

The linking process has to be done with full-path. Otherwise, it's not working properly. The errors can be checked from __/var/log/nginx/error.log__.

The permission denied problem can be fixed via the solution in the [link](https://django.fun/en/qa/146882/).<br>
I follow the rest of the settings from [official website](https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html).

in __\<project-path>/settings.py__ we have to switch STATICFILES_DIRS to STATIC_ROOT.<br>
Then, we should collect the static files via
```console
(venv)$ python manage.py collectstatic
```

Then we start uwsgi & nginx handle everything together.
```console
(venv)$ uwsgi --socket website.sock --module website.wsgi --chmod-socket=664
```

Now we are going to create an initiliaze file to handle from start.

website_uwsgi.ini
```ini
# mysite_uwsgi.ini file
[uwsgi]

# Django-related settings
# the base directory (full path)
chdir           = /path/to/your/project
# Django's wsgi file
module          = <project-name>.wsgi
# the virtualenv (full path)
home            = /path/to/virtualenv

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 5
# the socket (use the full path to be safe
socket          = /<project-path>/website/website.sock
# ... with appropriate permissions - may be needed
# chmod-socket    = 664
# clear environment on exit
vacuum          = true
# deamonize uwsgi and write messages into given log
deamonize	= /<desired-path>/uwsgi_output.log
```

then type 
```console
(venv)$ uwsgi --ini website_uwsgi.ini
(venv)$ reboot
(venv)$ uwsgi --emperor /home/serkan/Desktop/test-djang-website/cloud-test/src/vassals/ --uid www-data --gid www-data
```