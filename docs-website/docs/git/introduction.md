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

**Blob**

+ Stores file content
+ Does not store file name

**Tree**

+ Represents folders
+ Connects file names to blobs

**Commit**

A saved snapshot of the project

Points to a tree

Has message, author, time

**Branch**

Just a name (pointer)

Points to a commit

**What is HEAD?**

HEAD means current position.
HEAD points to the current branch.
The branch points to a commit.

Example:
HEAD → main → commit

### Important Truths

+ Git stores snapshots, not diffs
+ Branch is not a copy of code
+ Commit does not store files directly

Everything in Git is based on pointers