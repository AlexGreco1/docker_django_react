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

# 4 Rest API

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

# 5 MODELS:

20. In the folder Users, files models.py:

class User(models.Model):
    first_name = models.Charfield(max_lenght=200)
    last_name = models.Charfield(max_lenght=200)
    first_name = models.Charfield(max_lenght=200, unique=True)
    email = models.Charfield(max_lenght=200)
    
21. In the users folder, in the views.py file, add:

from .models import User

@api_views([GET])
def user(request):
    users = User.objects.all()
    return Response(users)
    
    
22. in users folder in the urls.py modified:
    from .views import users
    
    path('users',users),
 
 
23. Now, to exist, we need to migrate using terminal. command:
python manage.py makemigratetions
puthon manage.py migrate

# 6 SERIALIZE:

24. In the docker-compose.yaml file, add at the end (localhost:docker container):
    ports:
    - 33066:3306

25.  Restar container (stop) and them in terminal:
docker-compose up

26. Connect the database in jetbrains:
host:localhost, port: 33066, password:root, database: django_admin

27. Register function in the views.py file in the users folder add:
@api_view(['POST']):
def register(request):
    pass
    
28. Create a new file called serializer.py in the users folder to manipulate json data and validate:
form rest_framework import serializers
from .models import user

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = '__all__'

29. In the views.py in the users folder change the pass to:

    from .serializers import UserSerializer

    data = request.data
    
    if data['password'] != data['password_confirm']:
        raise exceptions.APIException('Password do not match')
        
    serializer = UserSerializer(data=data)
    serializer.is_valid(raise_exception=True)
    serializer.save()
    return Response(serializer.data)

30. add in urls.py in the user folder:
    from .views import users, register
    path('register',register)
    
31. Test wrong passwor and them wright password, open postman open register page, mathod post, body. If is successfull we will see it in postman and in the user table:
{
"first_name":"a",
"last_name":"b",
"email":"a@a.com",
"password":1,
"password_confirm":2
}

# 7 Write Only:

32. Not show password in the respond. Go to serializers.py in users and change fields __all__ to:

['id','first_name','last_name','email','password']
extra_kwargs= {
'password': {'write_only': True}

33. Add to views.py in the users folder inside the users method change to:
    def users(rquests):
        serializer = UserSerializer(User.objetcs.all(), many=True)
        return Response(serializer.data)

# 8 Hashing password:

34. hashing password extending the models of django. In the models in the users change to:

from django.contrib.auth.models import AbstractUser

class User(AbstractUser):

    usarname = None
    
    USERNAME_FIELD = 'email'
    REQUIERED_FIELDS=[]


35. Settings.py add the end:
    AUTH_USER_MODEL = 'users.User'

36. Run migration inside admin folder in terminal:
    docker-compose exec admin_api sh
    
    python manage.py makemigrations
    
    python manage.py migrate
   
37. hashing password in serilizer.py in the users folder create a method:

def create(self, validated_data):
    password = validated_data.pop('password', None)
    instance = self.Meta.model(**validated_data)
    if password is not None:
        instance.set_password(password)
    intances.save()
    return instance 


38. create a new user to test like step 31:
{
"first_name":"a",
"last_name":"b",
"email":"a@a.com",
"password":1,
"password_confirm":2
}

# 9 Loging

39. 

 
