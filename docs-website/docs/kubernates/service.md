# SERVICE (Networking & Load Balancing)

1. Every time pod restarts --> new ip created.
2. if browser connects directly to pod ip. Immediately broken.
3. So kubernates needs something permanent ip , port etc.

## 2️⃣ What a Service Is

A service is a stable network endpoint that provides a fixed ip and DNS name to access a dynamic sets of pod.

## 3️⃣ What Service Actually Does (Very Important)

A Service provides 3 major things.

+ Stable ip
+ Load balancing
+ Pod discovery

It automatically distributes traffic across pods:

```ruby
User Request
     ↓
Service
 ↓   ↓   ↓
Pod1 Pod2 Pod3
```

## 4️⃣ How Service Knows Which Pods to Send Traffic

Service does not connect to a Deployment. Service connects to Pods using labels.

### Labels & Selectors

Your Pod has :

```
labels :
    app : myapp
```

Service searches :

```
selector :
    app : myapp
```

Match → traffic routed.

## 5️⃣ Types of Services (Very Important Section)

Kubernates have 3 main service type

### ① ClusterIP (Default)

Internal only.

Accessible only inside cluster.

Used for :

+ backend Api
+ microservice
+ database

Users cannot access from internet.

### ② NodePort

Opens a port for machine.

Example:
```
ServerIP:30007
```

Now browser can connect

### ③ LoadBalancer

Used in cloud (AWS , GCP , AZURE)

Cloud provider automatically creates :

+ external ip
+ public load balancer

## 6️⃣ Service YAML

```ruby
apiVersion : v1
kind : service
metadata :
    name : myapp-service

spec :
    type : NodePort

    selector :
        app : myapp
        
    ports : 
        - port : 80
          targetPort : 80
          nodePort : 30007
```

## 7️⃣ What Happens After You Create Service

Now flow becomes:
```
Browser → NodeIP:NodePort → Service → Pod → Container
```

Finally your application opens in browser 🎉

## 8️⃣ Built-in Load Balancing (Very Important)

If you have:

replicas = 3

Users connect:

Requests automatically distributed.

You just got :

```
load balancer

service discovery

failover
```

Without installing HAProxy or Nginx manually.

## 9️⃣ Hidden Feature — Service Discovery (Powerful Concept)

Inside Kubernetes cluster:

Other services can call your app using :
```
http://myapp-service
```

No IP needed.

Kubernetes has internal DNS.

This is how microservices communicate.
