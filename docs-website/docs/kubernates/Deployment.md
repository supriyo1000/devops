# Deployment — Managing Pods Automatically

## 1️⃣ First Understand the Problem

In stage-2 you created a pod.

But we discovered a serious issue now :

```
if pod dies --> application will die.
```

Kubernates does not recreate a pod like you created.

when :
+ server memory spike
+ node reebot
+ container crash

So kubernates intoduces **Controller**.

**Deployment** is a type of Controller.

## 2️⃣ What Deployment Actually Is

Deployment is a Kubernates controller that ensures a specific number of identical pods are always running.

#### Desired State

you dont need to start applications.

You specified a rule :
```
i want 3 copies of pod always running for my application
```

Kubernetes continuously checks and maintains it.

## 3️⃣ What Happens Internally

You created a deployment

Kubernates automatically does

```
Deployment --> Replica set --> Pods --> Containers
```

|Object | Responsibility|
|-------|---------------|
|Deployment | defines application rule |
|Replica set| keeps correct number of Pods|
|Pod| runs container|

## 4️⃣ The Self-Healing Magic

Kubernates automatically replace the failed pods for system availablity. So User never notice.

## 5️⃣ Horizontal Scaling

Now traffic increases.

Now you change :

```
replicas : 3 --> replicas : 6
```

Kubernates automatically creates 3 more pods.

+ no ssh
+ no docker commands
+ no manual start

## 6️⃣ Updating Application Without Downtime

you release a new version of your app.

#### in docker deployment :

+ stop old container
+ start new container
    ➡ downtime

#### In Kubernetes Deployment:

It performs:

### Rolling Update

+ 1 new pod created => health check
+ remove old pod
+ repeat

+ user cant experience outage.

## 7️⃣ Rollback (Life Saver Feature)

suppose Your new version have a bug.

You run

```ruby
kubectl rollout undo deployment myapp
```

System instantly returns to previous version.

## 8️⃣ Deployment YAML

```ruby
apiVersion : apps/v1
kind : Deployment

metadata : 
    name : myapp-deployment

spec :
    replicas : 3  // number of copies
    selectors :  // which pods belogs to this deployment
        matchLabels :
            app : myapp

    template :  // blueprint of pod
        metadata :
            labels :
                app : myapp

        spec :
            containers :  // what image to run
                - name : myapp-container
                image : nginx
                ports :
                   - containerPort : 80
```

+ **Deployment** does not run containers.
+ it **creates pods** using the template.

## 9️⃣ Why Selector & Labels Exist (Hidden Key Concept)

Kubernates tracks pods using **label**.

Think

#### label = tag

```
labels :
    app : myapp
```

Deployment only control pods with matching labels.

This is how Kubernetes connects everything later (especially Services).

