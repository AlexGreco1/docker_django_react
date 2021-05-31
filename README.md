# docker_django_react
test docker django react


Steps:

1.- Install Django:

python3 -m pip install django

django-admin startproject admin

cd admin

python3 manage.py runserver



2.- In the main folder create a folder named requirements.txt

Django==3.1.3
djangorestframework==3.12.2
mysqlclient==2.0.1
django-mysql==3.9


3.- Create a Dockerfile and docker.compose.yaml in the main folder

Dockerfile:

FROM python:3.9
ENV PYTHONYNBUFFERED 1
WORKDIR /app
COPY requirements.txt /app/requirements.txt
RUN pip install -r requirements.txt
COPY . /app

CMD python manage.py runserver 0.0.0.0:8000

4 docker-compose.yaml:

version: '3.8'
services:
    admin_api:
      container_name: django_api
      build:
        context: .
        dockerfiile: Dockerfile
      volumes:
        - ./:app
      ports:
        - 8000:8000
      depends_on:
        - admin_db
        
  
        
   admin_db:
     container_name: django_admin_db
     image: mysql: 5.7.22
     restart: always
     environment:
       MYSQLDATABASE: django_admin
       MYSQL_USER: root
       MYSQL_PASSWOED: root
       MYSQL_ROOT_PASSWORD: root
     volumes:
       - .dbdata:/var/lib/mysql
       
       
5.- Change the settings.py file databases in line 76:

'ENGINE': 'django.db.backends.mysql',
NAME: 'django_admin',
user: 'root',
password: 'root',
host: 'admin_db,
port: '3306'


6.- create the container in terminal and the folder:
docker-compose up

7.- go to the url: (this is because 0.0.0.0 is inside our localcontainer)
localhost:8000

8.- In terminal go to inside docker container:
docker-compose exec admin_api sh

9.- create user insede the container using the last command:

python manage.py startapp users

10.- run migration of the data base:
python manage.py migrate

11.- create super user:
python manage.py createsuperuser--email a@a.com
password: a

12.- go to the side:
localhost:8000/admin

13.- enter the super user name just to see...

ch4:

14.- create the first api. in the users folder create the file views.py:

from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(['GET'])
def hello(request):
  return Response('hello')
  
  
15.- create a new file in the user folder named urls.py:

from django.urls import path
from .views import hello

urlpatterns = [
path('hello',hello),
]

16.- in the urls.py in the admin folder, add in the paths:

from django.urls import path, include
path('api/',include('users.urls'))

17.- in settings, line 38 in installed_apps, add:

'rest_framework',
'users',

18.- check localhost:8000/hello

19.- if we change get to post, we need to use postman to test it:
copy the localhost:8000/hello after change to post method in the decorator, and test it in postman.
