# Pull Request Workflow & Code Review Process

## What is a Pull Request
A pull request(PR) is a request to merge changes into a protected branch after review and validation.

PR exists because :
+ main must be stable.
+ no one should directly push changes into main.
+ changes must be reviewed.

## 🧩 WHO DOES WHAT (Very Important)

| Role | Responsibility |
|------|----------------|
|Developer| writes codes and open PR|
|Reviewer | Reviews logic and risk|
|Ci | Automatically checks|
|Maintainer | Approves and merge|


## COMPLETE PR WORKFLOW

### Step 1 – Create Feature Branch
```
git checkout -b feature/login
```

#### Rule
Never work directly on main

### Step 2 – Work & Commit (Small commits)
```
git add .
git commit -m "add login"
```
#### Rule
+ one logical change per commit
+ clear commit messages

### Step 3 – Push Branch
```
git push -u origin feature/login
```

Now github sees your branch

### Step 4 – Open Pull Request

PR usually targets:
```
feature/login  →  main
```

PR includes :
+ Title
+ Description
+ linked issue
+ screenshots/logs(if needed)

### Step 5 – CI Runs Automatically

Ci checks :
+ build
+ test
+ lint
+ security rules

if Ci fails PR cannot be merged.

### Step 6 – Code Review Happens

Reviewr checks :
+ Logic
+ naming
+ edge cases
+ securities
+ performance test

Reviewer may :
+ Approve
+ request changes
+ comment

### Step 7 – Fix & Update PR
You :
```
git commit -am "fix validation bug"
git push
```

PR updates automatically.

### Step 8 – Merge PR

Merge options :
+ squash and merge(clean history and most common)
+ Merge commit(full history).
+ Rebase commit(rewrite History).

After merege :
+ feature branch will be deleted.
+ main updated.

### 🧠 WHY SQUASH MERGE IS COMMON

instead of :
```
fix
fix again
opps
final fix
```

Squash gives :

```
add login feature
```

Clean history and Teams happy.

### Squash Merge — Example

#### Step 1 - Create feature branch
```
A → B   (main)
     \
      (feature/login)
```

#### ✍️ STEP 2: You work and commit MANY times (this is normal)
```
git commit -m "add login page"
git commit -m "fix validation bug"
git commit -m "remove debug logs"
```

Now history looks like:
```
A → B   (main)
     \
      C → D → E   (feature/login)
```

This is normal developer work.

#### STEP 4: Open Pull Request (feature/login → main)

On GitHub:

+ CI runs ✅
+ Reviewer approves ✅

Now comes the key decision 👇

#### 🔀 STEP 5: Choose SQUASH AND MERGE

You click:
```
“Squash and merge”
```
GitHub asks for one commit message:
```
Add login feature
```

#### ✅ FINAL RESULT (MOST IMPORTANT)

Main branch AFTER squash merge :
```
A → B → S   (main)
```

Where:

+ S = single new commit
+ Message: "Add login feature"

❌ Commits C, D, E are NOT in main history

+ ✅ Main history is clean
+ only feature branch have this commits.
