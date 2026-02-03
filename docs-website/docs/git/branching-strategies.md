## Branching Strategies (feature, release, hotfix)

### Why Branches Exist

Branches are used to protect stable code and allow safe development.

The main branch always be stable.

### feature branch
+ used to create a new features
+ created from main
+ merge after completion.
+ delete after merge

Example :
+ feature/login

### release branch
+ used to prepare code for production
+ only bug fixes allowed. no new feature.
+ created when features completed

Example :
+ Release/1.1.0
+ Release/2.0

### Hotfix Branch
+ used for urgent production bugs
+ directly created from main
+ merge quickly

Example :
+ hotfix/payment-fix

### Branching Rules
+ Never commit directly to main
+ use pull request


## Merge vs Rebase

### First: One Simple Truth
+ merge and rebase both do the same job.
+ they bring changes from one branch to another branch

Difference is **How the history looks**. not what code you get.

### Story Setup
+ you have main branch(stable)
+ you are working in **feature/login**

While you are working main got new commits.

Now you want to bring the changes of main into your branch.

Now you have two option :
+ **merge**
+ **rebase**


### Setup
```ruby
git init demo
cd demo
echo "v1" > app.txt
git add app.txt
git commit -m "initial commit"
```
now you have :
main => commit A

### Create Feature Branch

```ruby
git checkout -b feature/login
echo "login features">> app.txt
git add .
git commit -m "added login feature"
```
now you have :
main => commit A
feature/login => commit B

### Add New Commit to main

```ruby
git checkout main
echo "hot update" >> app.txt
git add .
git commit -m "hot update on main"
```
now you have :

+ main => commit A => commit C
+ feature/login => commit B

### Now you have 2 option :


## Merge (Bring branches together)

![Tux, the Linux mascot](/static/img/git-three-way-merging.png)

+ Combines branches using a merge commit.
+ safe for shared branches.
+ preserve full history.
+ can create messy logs

```ruby
git checkout feature/login
git merge main
```
Whats happens :
+ git creates a new merge commit
+ history keeps both paths

Use :
```ruby
git log --oneline --graph
```

you will see the merge.

## REBASE (Clean)
Only do this on feature branch
```
A → B   (main branch)
```
```
A → B   (main)
          \
           C → D   (feature)
```
```
A → B → E → F -> G   (main)
    \
     C → D     (your branch is outdated now)
```

```ruby
git checkout feature/login
git rebase main
```

```
A → B → E → F → G → C' → D'  (rebased feature branch)
```

+ Rewrites commit history

+ Replays commits on top of another branch

+ Creates clean, linear history

+ Should not be used on shared branches





