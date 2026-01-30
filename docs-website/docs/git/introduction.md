### What is Git?

Git is a version control system that saves snapshots of a project so we can track history, go back in time, and work safely with others.

Git does NOT store changes line-by-line.
Git stores snapshots of the project.

### Git vs GitHub

Git is a tool that runs on my computer.

GitHub is a website that hosts Git repositories.

Git works without internet.

GitHub is used for collaboration.

### The .git Folder

When we run git init, Git creates a hidden folder called .git.

This folder stores:

+ project history
+ commits
+ branches

everything Git needs

### Git Objects (Internal Storage)

> [!NOTE]
> Git saves the project as objects.
> Everything in Git is built using only 4 objects.

**Blob — Think: “File Content”**
A Blob is the content of a file.

Example file:
**hello.txt**

Content :
hello world

Git stores :
+ not the file name
+ note the path
+ it stores only the content

__That stored content = Blob__

>[!NOTE]
> if 2 file have same content then git stores only 1 blob.
>This is why git is efficient.


**Tree — Think: “Folder Structure”**

A Tree represents :
+ a folder
+ file names
+ file structure

Example :
```
project/
 ├── hello.txt
 └── app.js
```

Tree says :
hello.txt => blob123
app.js => blob456

So :
+ Tree knows names
+ blobs knows content

**Commit — Think: “Save Button”**

Commit = Snapshot of the whole project at a moment in time

A commit contains :

+ reference to a tree
+ reference to a parent commit
+ message
+ author

Points to a tree

**Branch**

Just a name (pointer)

Points to a commit

**HEAD — Think: “Where am I now?”**

HEAD means current position.
HEAD points to the current branch.
The branch points to a commit.

Example:
HEAD → Branch (main) → commit -> Tree -> Blob

Meaning :
+ I am on a branch main
+ main points to commit A

You run:
```ruby
git add file.txt
git commit -m "first commit"
```

Git does internally :

+ Creates blob for file content
+ Creates tree for folder structure
+ Creates Commit pointing to a tree
+ Moves branch pointer
+ HEADS follows branch

### Important Truths

+ Git stores snapshots, not diffs
+ Branch is not a copy of code
+ Commit does not store files directly

Everything in Git is based on pointers