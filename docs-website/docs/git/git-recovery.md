# Git Recovery Notes

## The most important truth about **git**

+ git does not store any file or code
+ git stores snapshot called commits.

+ a branch (main/feature) is a pointer to a commit.
+ so when my code disappears eventually :
```
it does not actually deleted. my branch pointer moved
```

## Reflog - git time machine 

git refog shows where my head and branches pointed in the past.

even if :
+ hard reset
+ delete branch
+ checkout old commit

### Command

```ruby
git reflog
```

### How i recover lost work

1. Run git reflog.
2. find the commit id before mistake.
3. Copy commit id then run

```ruby
git reset --hard <commit-id>
```

#### Important
git reflog is my first action when my code disappears.

#### Golden rule
if code is missing => do not panic => run git reflog.

## 3. Reset — Moving the Branch Pointer

Reset changes where the branch points.

### Soft reset

```ruby
git reset --soft HEAD~1
```

Effect :
+ Remove commit
+ keeps changes staged
+ Effective when **commit** too early but my code is correct.

### Mixed Reset (default)
```ruby
git reset HEAD~1
```

Effect :
+ Remove commit.
+ keep code in file but unstaged
+ Effective when **commit** too early and i want to edit and recommit.

### Hard Reset ⚠️
```ruby
git reset --hard HEAD~1
```

Effect :
+ Remove commit.
+ Remove file or code changes.

Danger:
+ Looks like data loss.

## 4. Revert — Safe Undo (For Team Work)

+ reset rewrites history.
+ revert reserves history.

#### Command 
```
git revert <commit-id>
```

#### When to use
+ code already pushed.
+ working in shared repository.
+ company project

#### Golden Rule
+ local repository mistake => reset
+ pushed mistake => revert

## 5. Cherry-pick — Copy a Commit

Used to bring a commit from another branch.

#### Command
```
git cheery-pick <commit-id>
```

#### What is does

Apply the same changes on the current branch.

#### Uses 
+ hotfix production
+ copy bug fix
+ move one feature only.