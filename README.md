# Setting up Django with PostgreSQL in Docker

This guide will walk you through setting up a Django project with PostgreSQL as the database engine, all running inside Docker containers.

## Prerequisites

Before you begin, ensure you have the following installed on your system:

- Docker
- Docker Compose

## Steps

1. **Clone the Repository**: Clone the repository containing your Django project or create a new one.

   ```bash
   git clone <repository_url>
   cd <repository_directory>
   ```

2. **Update Django Settings**: Update your Django project's settings (`settings.py`) to use PostgreSQL as the database engine.

   ```python
   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.postgresql',
           'NAME': 'postgres',
           'USER': 'postgres',
           'PASSWORD': 'password',
           'HOST': 'db',  # This should match the service name in your Docker Compose file
           'PORT': 5432,
       }
   }
   ```

3. **Create a Dockerfile**: Create a `Dockerfile` in your project directory with the provided content.

   ```Dockerfile
   FROM python:3.9

   # Set environment variables
   ENV PYTHONDONTWRITEBYTECODE 1
   ENV PYTHONUNBUFFERED 1

   # Set work directory
   WORKDIR /code

   # Install dependencies
   COPY requirements.txt /code/
   RUN pip install -r requirements.txt

   # Copy project
   COPY . /code/
   ```

4. **Create a Docker Compose File**: Create a `docker-compose.yml` file in your project directory with the provided content.

   ```yaml
   version: '3'

   services:
     db:
       image: postgres
       environment:
         POSTGRES_DB: ${NAME}
         POSTGRES_USER: ${USR}
         POSTGRES_PASSWORD: ${PASSWORD}

     web:
       build: .
       command: python manage.py runserver 0.0.0.0:8000
       volumes:
         - .:/code
       ports:
         - "8000:8000"
       depends_on:
         - db
   ```

5. **Build and Run Docker Containers**: Build and run your Docker containers using the following command:

   ```bash
   docker-compose up --build
   ```

6. **Apply Migrations**: Open another terminal and access the shell of the Django container and apply migrations.

   ```bash
   docker exec -it <django_container_name> bash
   python manage.py migrate
   exit
   ```

7. **Access Django App**: Your Django app should now be accessible at `http://localhost:8000`.


## Building and Pushing Docker Image to Docker Hub

You only need to build the Docker image if you want to push it to a container registry like Docker Hub. If you're just running the containers locally for development or testing purposes, then you can start them using docker-compose up without explicitly building the image using docker build.
To push your Docker image to Docker Hub, first create a repo in dockerhub after signin and then follow these steps:

1. **Build your Docker image**: Before you can push your image to Docker Hub, you need to build it locally using the `docker build` command. Make sure you are in the directory where your Dockerfile is located.

   ```bash
   docker build -t username/repository_name .
   ```

   This command will build your Docker image with the tag `username/repository_name`.

2. **Tag your Docker image**: Once the image is built, you can tag it with the repository name on Docker Hub.

   ```bash
   docker tag username/repository_name username/repository_name:1.0
   ```

   This command ensures that your local image is tagged correctly for pushing to Docker Hub.

3. **Push your Docker image**: Now you can push the tagged image to Docker Hub.

   ```bash
   docker push username/repository_name:1.0
   ```

   This command will upload your local Docker image to the `username/repository_name` repository on Docker Hub.


