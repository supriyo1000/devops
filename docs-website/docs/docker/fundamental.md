# Lesson 1: What is Docker?

### 1. Definition

Docker is a platform that packages an application and its dependencies into a container so that it can run consistently across different environments.

For example, your Spring Boot application might work on your Windows computer but fail on a Linux server because of differences in Java versions or installed dependencies.

Docker helps solve this problem by packaging the application with its required runtime and configuration.

### 2. Why do we need Docker?

Imagine you have developed a Spring Boot application using Java 17.
- Your computer has Java 17 installed.
- The server has Java 11 installed.
- Your application requires Java 17, so it fails to start on the server.
Without Docker, you need to configure the server's environment correctly.

With Docker, you can package your application with a compatible Java runtime and run it in a container.

__Remember__: Docker makes application environments more consistent, but it does not automatically eliminate every environment-related issue.

### 3. What is a container?
A container is an isolated environment in which an application runs with its required dependencies.

For example, you can run:
- A React frontend in one container.
- A Spring Boot backend in another container.
- A PostgreSQL database in another container.

One host can run multiple isolated containers.

### 4. Benefits of Docker

1. __Portability:__ Run the same image on different compatible hosts.
2. __Consistency:__ Reduce differences between development, testing, and production.
3. __Isolation:__ Keep application processes and dependencies separated.
4. __Scalability:__ Run multiple instances of an application when needed.
5. __Deployment:__ Package and deploy applications in a repeatable way.


### 5. What is a Docker Image?

A Docker image is a read-only template used to create containers.

It contains the application code, runtime, dependencies, and filesystem required to run an application.

Example: An image can contain your Spring Boot JAR and a compatible Java runtime.

One image can be used to create multiple independent containers.

###6. What happens when you delete a container?
```
docker rm -f web1
```

The container is removed, but the Nginx image remains. You can confirm this with docker images.

### 6. What happens when you delete an image?
```
docker rmi nginx
```

Docker removes the image if it is not being used by existing containers and other removal constraints are satisfied.


### 7. Does deleting a container delete all its data?

Not necessarily.
- Data in the container's writable layer is lost when the container is removed.
- Data stored in a separate Docker volume can persist.
- Bind-mounted data remains in its host location.
This distinction is particularly important when running databases.


### 8. What is the difference between a Docker image and a container?

A Docker image is a read-only template containing an application's filesystem, runtime, and dependencies. A container is an instance created from that image, with its own writable layer and isolated runtime environment. Multiple containers can be created from the same image, which makes application deployment and scaling easier.