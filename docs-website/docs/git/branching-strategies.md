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

### Example 1 :

**Situation** :
```ruby
A → B   (main)
     \
      C → D   (feature)
```
+ feature is created from B.
+ main branch is still point to B.

**Command** :
```ruby
git checkout feature
git merge main
```

**Output** :
```
A → B → C → D   (feature)
```

No merege commit needed(fast forward).

**Why?**
Because git just moved the pointer forward.


### Example 2 :

**Situation** :
```ruby
A → B → E   (main)
     \
      C → D   (feature)
```

**Command** :
```
git checkout feature
git merge main
```

**Output** :
```
A → B → E
     \     \
      C → D → M
```

+ M = merge commit.
+ Now M has two parents D and E.

### Example 3 :

**Situation** :
```
A → B → E → F → G   (main)
     \
      C → D        (feature – outdated)
```

**Command** :
```
git checkout feature
git merge main
```

**Output** :
```
A → B → E → F → G
     \           \
      C → D → → → M
```
+ git creates a merge commit M.
+ now feature includes all main changes.

### 🧠 IMPORTANT RULE

+ When main has new commits, merge creates a new commit.
+ When main does not have new commit, git moved the pointer to final commit.
+ preserve full history.

**Check** :
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
A → B → E → F → G → C → D  (rebased feature branch)
```

+ Rewrites commit history

+ Replays commits on top of another branch

+ Creates clean, linear history

+ Should not be used on shared branches

## Interactive Rebase

interactive rebase is used to :
+ squash commits
+ edit commit messages
+ reorder commits 

#### Example :

You have 5 messy commits:
```ruby
fix
fix again
oops
final fix
```

#### Run :
```
git rebase -i HEAD~4
```

#### You can :
+ squash them into 1 clean commit
+ rename commit message


## 🔥 MERGE CONFLICT

### When does conflict happen?

it happens when :
+ same file
+ same line
+ modified in both branches

### what to do :
+ open file
+ decide correct content
+ remove conflict marker
+ save then run 

```
git add app.txt
git commit
```

### Q1. What happens internally during merge?
Git creates a new merge commit with two parent commit and preserve full history.

### Q2. When is fast-forward merge possible?
when the base branch has no new commit. git only moves the pointer to new commit when merge.

### Q3. What causes merge conflicts?
when same files and same lines are modified in both branch.

### Q4. How do you resolve merge conflicts?
By manually editing conflict files , resolve conflict marker then stage then commit.

### Q5. Merge vs Rebase — when do you use merge?
when we are working in shared branch. because it does not rewrites history. 


