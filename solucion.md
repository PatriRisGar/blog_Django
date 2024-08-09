Ejercicio realizado siguiendo las pautas de https://tutorial.djangogirls.org/es/

INSTALACIÓN DE DJANGO

Teniendo en cuenta que ya tenemos instalado python3 y un entorno virtual, que yo en mi caso he usado virtualenvwrapper, pasamos a hacer la instalacion de Django.

·Abrimos la terminal y creamos el entorno virtual, y entraremos automaticamente en el promt del entorno, veremos (myvenv).

    mkvirtualenv myvenv

·Creamos una carpeta donde almacenaremos todo lo relacionado con el framework.

    mkdir djangogirls

·Dentro creamos un requirements.txt y escribimos en su interior: Django~=3.2.10. Una vez creado, en la terminal actualizamos pip , cuando finalice instalamos django con el .txt que hemos creado.

    python -m pip install --upgrade pip
    pip install -r requirements.txt

CONSEJO -- Vamos a generar BBDD y entorno virtual, esto junto con otros archivos no nos interesa que esten publicados en gitHub asi que antes de comenzar todo el proceso, recomiendo generar un .gitignore en la carpeta djangogirls. Dentro del archivo escribimos lo siguietne para que al hacer commits y push no los incluyan:

    _.pyc
    _~
    **pycache**
    myvenv
    db.sqlite3
    /static
    .DS_Store

PROYECTO NUEVO

El objetivo de este curso es realizar un blog.

En MacOS o Linux deberías ejecutar el siguiente comando en la consola. no te olvides de añadir el punto . al final.

    (myvenv) ~/djangogirls$ django-admin startproject mysite .

Vamos a hacer algunos cambios en mysite/settings.py, en cada apartado haremos sus correspondientes modificaciones:

    TIME_ZONE = 'Europe/Berlin'
    LANGUAGE_CODE = 'es-es'
    STATIC_URL = '/static/'
    STATIC_ROOT = BASE_DIR / 'static'
    ALLOWED_HOSTS = ['127.0.0.1', '.pythonanywhere.com']
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': BASE_DIR / 'db.sqlite3',
        }
    }

Crear e iniciar la base de datos.

    (myvenv) ~/djangogirls$ python manage.py migrate

Ya tenemos el ervidor listo, arranquemos y enteremos desde el navegador en el que devemos ver un cohete.

    (myvenv) ~/djangogirls$ python manage.py runserver

http://127.0.0.1:8000/

CREAR APLICACIÓN
Vamos a crear la aplicacion blog

    (myvenv) ~/djangogirls$ python manage.py startapp blog

La hacemos visible para Django

    mysite/settings.py
        INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'blog.apps.BlogConfig',
        ]

Creamos modelo

    blog/models.py
    from django.conf import settings
    from django.db import models
    from django.utils import timezone

            class Post(models.Model):
                author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
                title = models.CharField(max_length=200)
                text = models.TextField()
                created_date = models.DateTimeField(
                        default=timezone.now)
                published_date = models.DateTimeField(
                        blank=True, null=True)

                def publish(self):
                    self.published_date = timezone.now()
                    self.save()

                def __str__(self):
                    return self.title

La clase Post va a ser una tabla de la BBDD asiqeu tenemos que preparar el modelo para migrarlo a la misma.

    Preparacion
    (myvenv) ~/djangogirls$ python manage.py makemigrations blog
    Migración
    (myvenv) ~/djangogirls$ python manage.py migrate blog

ADMINISTRADOR DE DJANGO
Sjango cuenta con un administrador, vamos a crear uno para añadir,editar y borrar las entradas de nuestra aplicacion blog.

    blog/admin.py
        from django.contrib import admin
        from .models import Post
        admin.site.register(Post)

Recuerda ejecutar python manage.py runserver en la consola para correr el servidor web. Ve a tu navegador y escribe la dirección http://127.0.0.1:8000/admin/.
Ahora creamos desde la terminal un super usuario. Dejamos la terminal con el servidor iniciado, entramos myvenv y ejecutamos el siguiente comando:

    (myvenv) ~/djangogirls$ python manage.py createsuperuser

Ya podemos logarnos desde el navegadorcon las credenciales. Creamos varias entradas en el blog.

DESPLIEGE

En caso de no haber estado subiendo ya el codigo a gitHub, es momento de crear un repositorio en gitHub y subir el proyecto. IMPORTANTE haber realizado el consejo del .gitignore

CONFIGURAR NUESTRO BLOG EN PYTHONANYWHERE

Accedemos a www.pythonanywhere.com y nos registramos con una cuentra de principiante (Beginner). Tras ello vamos a la pestaña API Token y generamos una.

Para hacer el despliege desde git a pythonanywhere tenemos que abrir una terminal bash desde pythonanywhere e instalar.

    $ pip3.6 install --user pythonanywhere

Ahora ejecutaremos el asistente para configurar automáticamente nuestra aplicación desde GitHub. Teclea lo siguiente en la consola de PythonAnywhere (no te olvides de usar tu propio nombre de usuario de GitHub en lugar de <your-github-username>, para que la URL sea como la URL de clonar el repo de GitHub):

    $ pa_autoconfigure_django.py --python=3.6 https://github.com/<your-github-username>/my-first-blog.git

Nos logamos con el superusuario

    (ola.pythonanywhere.com) $ python manage.py createsuperuser

Hacemos ls en la termianl para ver que se ha hecho el pull correctamente del repositorio.

Queremos de http://127.0.0.1:8000/ sea la pagina de inicio del blos y muestre las entradas publicadas asi que añadimos esta URL al archivo de mysite/urls.py

    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('blog.urls')),
    ]

Tambien tenemos que añadir el fichero urls.py en blog

    from django.urls import path
    from . import views

    urlpatterns = [
    path('', views.post_list, name='post_list'),
    ]

Necesitamos un fichero en el que plasmar la lógica de nuestra aplicacion (view). Accedemos desde la carpeta blog y añadimos las lineas que faltan:

    from django.shortcuts import render
    from django.utils import timezone
    from .models import Post
    def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})

Vamos a crear una plantilla. Necesitamos crear una carpeta templates en la carpeta de blog, y dentro de templates otra carpeta llamada blog. Ahora creamos un archivo llamado post_list.html que escribiremos lo siguiente, teniendo en cuenta que podemos personalizarlo como mas nos guste:

    <html>
        <head>
            <title>Django Girls blog</title>
        </head>
        <body>
            <div>
                <h1><a href="/">Django Girls Blog</a></h1>
            </div>

            <div>
                <p>published: 14.06.2014, 12:14</p>
                <h2><a href="">My first post</a></h2>
                <p>Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Donec id elit non mi porta gravida at eget metus. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.</p>
            </div>

            <div>
                <p>published: 14.06.2014, 12:14</p>
                <h2><a href="">My second post</a></h2>
                <p>Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Donec id elit non mi porta gravida at eget metus. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut f.</p>
            </div>
        </body>
    </html>

Es momento de volver a desplegar en pythonanywhere para tenerlo disponible en internet.
Accedemos a la terminal de myvenv y usamos los siguientes comandos.

    git add --all .
    git commit -m "Cambie el HTML para la página."
    git push

Accedemos ahora a la consola bash de python anywhere para actualizar.

    git pull

QUERYSET
Queremos entrar en la consola de Django. Asi que en el cmd de myvenv, estando en la carpeta djangogirls escribimos...

    python manage.py shell

Una vez dentro queremos ver todas las entradas que hemos creado hasta el momento, para ello importamos Post y usamos sus metodos.

    >>> from blog.models import Post
    >>> Post.objects.all()

Desde aqui podemos generar objetos tipo Post. Pero necesitamos genera run usuario en la BBDD

    >>> from django.contrib.auth.models import User
    >>> User.objects.all()
    >>> me = User.objects.get(username='ola')
    >>> Post.objects.create(author=me, title='Sample title', text='Test')
    >>> Post.objects.all()

Como vemos una vez creado el usuario, podemos generar una nueva entrada.

- También podemos hacer filtrados >>> Post.objects.filter(title\_\_contains='title')

Para publicar la entradad que acabamos de generar usamos:

    >>> post.publish()

Para tener una lista en inicio con los post publicados de forma ordenadas necesitamos añadir el siguiente cogigo al fichero views.py de blog

    from django.shortcuts import render
    from django.utils import timezone
    from .models import Post

    def post_list(request):
        posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
        return render(request, 'blog/post_list.html', {'posts': posts})
        
PLANTILLAS

Generar una plantilla base con css para que usen otras paginas de la app. Creamos otro fichero en blog > templates > blog, de nombre base.html.

    {% load static %}
    <html>
      <head>
        <title>Django Girls blog</title>
        <link
          rel="stylesheet"
          href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css"
        />
        <link
          rel="stylesheet"
          href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css"
        />
        <link
          href="//fonts.googleapis.com/css?family=Lobster&subset=latin,latin-ext"
          rel="stylesheet"
          type="text/css"
        />
        <link rel="stylesheet" href="{% static 'css/blog.css' %}" />
      </head>
      <body>
        <div class="page-header">
          {% if user.is_authenticated %}
          <a href="{% url 'post_new' %}" class="top-menu"
            ><span class="glyphicon glyphicon-plus"></span
          ></a>
          {% endif %}
          <h1><a href="/">Django Girls Blog</a></h1>
        </div>
        <div class="content container">
          <div class="row">
            <div class="col-md-8">{% block content %} {% endblock %}</div>
          </div>
        </div>
      </body>
    </html>  
    
Falta crear la plantilla para listado de posts añadir CSS boostrap. 
En un fichero nuevo que llamaremos post_list.html, que estará en blog > templates > blog ,introducimos el siguiente codigo. En el que combinamos HTML con PYTHON, BOOSTRAP y la plantilla base para mostrar datos de cada una de las publicaciones.

    {% extends 'blog/base.html' %} {% load static %}
    <html>
      <head>
        <title>Django Girls blog</title>
        <link
          rel="stylesheet"
          href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css"
        />
        <link
          rel="stylesheet"
          href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css"
        />
        <link
          href="//fonts.googleapis.com/css?family=Lobster&subset=latin,latin-ext"
          rel="stylesheet"
          type="text/css"
        />
        <link rel="stylesheet" href="{% static 'css/blog.css' %}" />
      </head>
      <body>
        {% block content %} {% for post in posts %}
        <div class="post">
          <div class="date">
            {{ post.published_date }}
          </div>
          <h2>
            <a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a>
          </h2>
          <p>{{ post.text | linebreaksbr }}</p>
        </div>
        {% endfor %} {% endblock %}
      </body>
    </html>
    
Lo añadimos al archivo views.py de blog

    def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})

Crear la pagina donde editar un post usando la plantilla base. 
Añadimos en blog > urls.py lo siguiente:

    path('post/<int:pk>/edit/', views.post_edit, name='post_edit'),
    
Modificamos tambien view.py de blog

    def post_new(request):
    if request.method == "POST":
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    return render(request, 'blog/post_edit.html', {'form': form})
    
Creamos un fichero llamado post_edit.html en templates > blog

    {% extends 'blog/base.html' %} {% block content %}
    <h2>New post</h2>
    <form method="POST" class="post-form">
      {% csrf_token %}
      {{ form.as_p }}
      <button type="submit" class="save btn btn-default">Save</button>
    </form>
    {% endblock %}


Otra para los detalles del post, usando la plantilla base y unirlo a la view

    {% extends 'blog/base.html' %} {% block content %}
    <div class="post">
      {% if post.published_date %}
      <div class="date">
        {{ post.published_date }}
      </div>
      {% endif %} {% if user.is_authenticated %}
      <a class="btn btn-default" href="{% url 'post_edit' pk=post.pk %}"
        ><span class="glyphicon glyphicon-pencil"></span
      ></a>
      {% endif %}
      <h2>{{ post.title }}</h2>
      <p>{{ post.text | linebreaksbr }}</p>
    </div>
    {% endblock %}
    
Volvemos a views.py de blog y añadimos 

    def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'blog/post_detail.html', {'post': post})

APLICACION FINALIZADA.

Quedaria hacer docker pull desde la terminal de pythonAnywhere para actualizar los cambios realizados.
