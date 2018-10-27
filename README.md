# Django_Login Part -1

## 1.Create a New Django Project
```
    mkdir accounts && cd accounts
    virtualenv -p pyhton3 .
    source bin/activate
    pip install django==2.0.7
    mkdir src && cd src
    django-admin startproject my_project .
    pyhton manage.py makemigrations
    python manage.py migrate
    python manage.py runserver
```
## 2.Edit urls.py
```
    #my_project/urls.py
    from django.contrib import admin
    from django.urls import path, include # new

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('accounts/', include('django.contrib.auth.urls')), # new
    ]
```

## 3.Create Login page
```
 From the command line type Control-c to quit our local server and enter the following commands:

    (accounts) ➜  src mkdir templates
    (accounts) ➜  src mkdir templates/registration
    (accounts) ➜  src touch templates/registration/login.html

```
## 4.Edit Login.html
```
    <!-- templates/registration/login.html -->
    <h2>Login</h2>
    <form method="post">
      {% csrf_token %}
      {{ form.as_p }}
      <button type="submit">Login</button>
    </form>
```
## 5.Updater settings.py
```
    # my_project/settings.py
        TEMPLATES = [
            {
                ...
                'DIRS': [os.path.join(BASE_DIR, 'templates')],# add only this line
                ...
            },
        ]
```
## 6.Redirect Login page
```
    # my_project/settings.py
    LOGIN_REDIRECT_URL = '/'
```
## 7.Create Users
```
    (accounts) ➜  src python manage.py createsuperuser
    Username (leave blank to use 'mainul'): accounts #you can use any username
    Email address: account@gmail.com
    Password:
    Password (again):
    Superuser created successfully.
```
## 8. Create Homepage
```
    (accounts) ➜  src touch templates/base.html
    (accounts) ➜  src touch templates/home.html

    <!-- templates/base.html -->
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="utf-8">
      <title>{% block title %}Django Auth Tutorial{% endblock %}</title>
    </head>
    <body>
      <main>
        {% block content %}
        {% endblock %}
      </main>
    </body>
    </html>


    <!-- templates/home.html -->
    {% extends 'base.html' %}

    {% block title %}Home{% endblock %}

    {% block content %}
    {% if user.is_authenticated %}
      Hi {{ user.username }}!
    {% else %}
      <p>You are not logged in</p>
      <a href="{% url 'login' %}">login</a>
    {% endif %}
    {% endblock %}
```
## 9. Edit Login.html
```
    <!-- templates/registration/login.html -->
    {% extends 'base.html' %}

    {% block title %}Login{% endblock %}

    {% block content %}
    <h2>Login</h2>
    <form method="post">
      {% csrf_token %}
      {{ form.as_p }}
      <button type="submit">Login</button>
    </form>
    {% endblock %}

```
## 10.Update Urls.py
```Now update our urls.py file so we display the homepage. On the third line, import TemplateView and then add a urlpattern for it.
```
```
    from django.contrib import admin
    from django.urls import path, include
    from django.views.generic.base import TemplateView # new

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('accounts/', include('django.contrib.auth.urls')),
        path('', TemplateView.as_view(template_name='home.html'), name='home'), # new
    ]
```
## 11.Logout
```
So let’s first add a link to the built-in logout url in our base.html file:
```
```
    <!-- templates/home.html-->
    {% extends 'base.html' %}

    {% block title %}Home{% endblock %}

    {% block content %}
    {% if user.is_authenticated %}
      Hi {{ user.username }}!
      <p><a href="{% url 'logout' %}">logout</a></p>
    {% else %}
      <p>You are not logged in</p>
      <a href="{% url 'login' %}">login</a>
    {% endif %}
    {% endblock %}
```


## 12.Replace Redirect
```
So we can replace '/' with home at the bottom of the settings.py file:

# my_project/settings.py
LOGIN_REDIRECT_URL = 'home'
LOGOUT_REDIRECT_URL = 'home'
```

# Django_Login Part -2(Signup Page)
## 13. Users app
```
    Since we’re making our own view and url for registration, we need to create a dedicated app. Let’s call it accounts.

    (accounts) ➜  src python manage.py startapp accounts
    Make sure to add the new app to the INSTALLED_APPS setting in our my_project/settings.py file:

    # my_project/settings.py
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'accounts.apps.AccountsConfig', # new
]
```
## 14. Edit urls
```

    # my_project/urls.py
    from django.contrib import admin
    from django.urls import path, include
    from django.views.generic.base import TemplateView

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('accounts/', include('accounts.urls')), # new
        path('accounts/', include('django.contrib.auth.urls')),
        path('', TemplateView.as_view(template_name='home.html'), name='home'),
    ]
```
## 15.Create urls.py in account directory
```
(accounts) ➜  src touch accounts/urls.py
```
```

# accounts/urls.py
from django.urls import path

from . import views


urlpatterns = [
    path('signup/', views.SignUp.as_view(), name='signup'),
]

```
## 16.EDit views.py
```
    # accounts/views.py
    from django.contrib.auth.forms import UserCreationForm
    from django.urls import reverse_lazy
    from django.views import generic


    class SignUp(generic.CreateView):
        form_class = UserCreationForm
        success_url = reverse_lazy('login')
        template_name = 'signup.html'
    ```
## 17.Create Signup

```
(accounts) ➜  src touch templates/signup.html

    <!-- templates/signup.html -->
    {% extends 'base.html' %}

    {% block title %}Sign Up{% endblock %}

    {% block content %}
      <h2>Sign up</h2>
      <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Sign up</button>
      </form>
    {% endblock %}
```
# Django_Login Part -3(PasswordReset)
## 18.Add code on setting.py
```
    # my_project/settings.py
    EMAIL_BACKEND = "django.core.mail.backends.filebased.EmailBackend"
    EMAIL_FILE_PATH = os.path.join(BASE_DIR, "sent_emails")

```
## 19.Password Reset Form
```
(accounts) ➜  src touch templates/registration/password_reset_form.html


<!-- templates/registration/password_reset_form.html -->
{% extends 'base.html' %}

{% block title %}Forgot Your Password?{% endblock %}

{% block content %}
  <h1>Forgot your password?</h1>
  <p>Enter your email address below, and we'll email instructions for setting a new one.</p>

  <form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit" value="Send me instructions!">
  </form>
</div>
{% endblock %}
```
