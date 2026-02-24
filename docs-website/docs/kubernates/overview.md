# Introduction

## Limitations of Running Applications Only With Docker

### A Single Server Reality

#### Current workflow with docker

```
Create a project --> Create a Dockerfile --> Runs Container
```

```ruby
docker run -d 3000:3000 myapp .
```

It only works in **development** setup. when you run your application in local **VM** or **VPS** server.

## What actually happens in real production

Imagine your helyx website becomes popular and 2000 users opens it at the same time.

now the probelm starts.

### Problem 1 — Container Crash

Node js app crash.

memory leak , unhandled exception , db connection anything.

What docker do 

```
stop container --> website down
```

Docker will not automatically recreate a container. You must manually start.

### Problem 2 — Server Crash

Your VPS provider reboots machine.

+ Docker stop
+ All Container gone
+ Website down

### Problem 3 — Traffic Increase

1 server = 100% CPU

So you start multiple container

```ruby
docker run -d -p 3000:3000 myapp .
docker run -d -p 3001:3000 myapp .
docker run -d -p 3002:3000 myapp .
```

### 3️⃣ The Load Balancing Problem

User do not choose 3000 , 3001 , 3002

So you use nginx for reverse proxy

```
user --> nginx -> Multiple Container
```

Now you must manually manage :

+ which container is alive
+ remove old container
+ restarts dead containers
+ which port is free
+ create new container

#### This is called **Container Orchestration Problem**.

### 4️⃣ Updating Application

You release a new version.

```
stop and remove old container
starts new container
// website down
```

#### So companies needed :

**Update application WITHOUT stopping it.**

## What Kubernetes Actually Is?

Kubernates is a distributed system that schedules and maintain containers across multiple machines automatically.

#### Kubernates = Automatic Devops Engineer

It :

+ Starts Containers
+ Restart Container automatically when crashes
+ Distributes Load
+ Replaces Dead server
+ Updates without downtime