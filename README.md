# Hidden secrets

Repo used to showcase bugs:

* [gitleaks/gitleaks#1028](https://github.com/gitleaks/gitleaks/issues/1028)
* [betterleaks/betterleaks#386](https://github.com/betterleaks/betterleaks/issues/386)

There is a commit in here [badc0de](../../commit/badc0de) which introduces a secret during a merge commit.  It's unable to be detected by both gitleaks and betterleaks.

## Create your own

This repo was created with the following bash script.

```bash
#!/bin/bash

# generate a random secret
export AWS_SECRET_ACCESS_KEY=$(python3 -c "import secrets; print(secrets.token_urlsafe()[:40])")

git init -b main hidden-secrets
cd hidden-secrets

# Create 3 commits so we have something to create a merge with
echo "# Hidden secrets" > README.md
git add . && git commit -m "Initial commit"
initial_commit=$(git rev-parse HEAD)

echo "spam" > foo.txt
git add . && git commit -m "Add foo file"
second_commit=$(git rev-parse HEAD)

echo "eggs" >> foo.txt
git add . && git commit -m "Update foo file"
third_commit=$(git rev-parse HEAD)

# create a branch to do some bad stuff on
git checkout -b bad $initial_commit
# create a trivial merge commit which does nothing
git merge --no-ff $second_commit -m "nothing to see here"
# ammend the merge commit to contain secrets
env | grep AWS_SECRET_ACCESS_KEY > SECRETS
git add . && git commit --amend --no-edit

# merge the bad branch into main
git checkout main
git merge bad -m "merge bad stuff"
git branch -D bad
```
