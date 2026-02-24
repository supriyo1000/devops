# Understanding Pod in Kubernetes

## 1️⃣ First — The Core Idea

+ Kubernates does not deploy **Container**
+ Kubernates deploy **Pods**.

A **Pod** is a smallest **deployable unit** in Kubernates that wraps one or more containers and provide them same **Network** and **Storage**.

## 2️⃣ Why Kubernetes Did NOT Use Containers Directly

In real production one application may need many helper processes.

#### Suppose :

+ Main Application (Node js)
+ Logging agent
+ Monitoring agent
+ file sync process

#### These process must :

+ run together
+ start together
+ stop together
+ communicate via localhost

#### So Kubernetes created a wrapper:

```
Pod
 ├── Container 1 (Main App)
 ├── Container 2 (Logger)
 └── Container 3 (Monitor)
```

## 3️⃣ The Most Important Pod Feature (You must remember)

inside a Pod:

#### All containers share:

+ same ip address
+ same network
+ shared storage volumes

Meaning:

**Container A** can talk to **Container B** using:
```
localhost:5000
```

No external networking needed.

This is called:
#### Shared Network Namespace

## 4️⃣ Pod IP Behavior

Pod ip is temporary
```
If a Pod dies --> new Pod --> new Ip
```

|Pod|Ip|
|---|---|
|pod abc| 10.29.40.9|
|(restart)| ❌ gone|
|pod-xyz|10.244.0.9|

This is why you need **Service**.

## 5️⃣ Pod Lifecycle (How Kubernetes Thinks)

If a Pod Crashed It cannot be repaired. Kubernates destroy it and recreate. It is called **Immutable Infrastructure**.

## 6️⃣ What a Pod Actually Contains

A pod is not a container.

It includes :

+ Containers 
+ volumes (storage)
+ Network identity (Ip and hostname)

## Pod YAML Structure

```ruby
# Kubernates API Version
apiVersion : v1 
# What object we create(pod)
kind : pod
# Identity (name and labels)
metadata : 
  name : helyx-app
  labels :
    app : myapp

# What should run inside
spec : 
  containers : 
    - name : container-name
      image : nginx
      ports : 
        - containerPort : 80
```