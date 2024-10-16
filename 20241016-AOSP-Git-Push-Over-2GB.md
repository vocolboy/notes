# AOSP Git Push Over 2GB

# Issue
* https://docs.github.com/en/get-started/using-git/troubleshooting-the-2-gb-push-limit

# Reappear Issue
Push AOSP full commit

# Fix Step
`vi ~/.gitconfig`
```
#add partial-push command
[alias]
    partial-push = "!sh -c 'REMOTE=$0;BRANCH=$1;BATCH_SIZE=100; if git show-ref --quiet --verify refs/remotes/$REMOTE/$BRANCH; then range=$REMOTE/$BRANCH..HEAD; else range=HEAD; fi; n=$(git log --first-parent --format=format:x $range | wc -l); echo "Have to push $n packages in range of $range"; for i in $(seq $n -$BATCH_SIZE 1); do h=$(git log --first-parent --reverse --format=format:%H --skip $i -n1);  echo "Pushing $h..."; git push $REMOTE ${h}:refs/heads/$BRANCH; done; git push $REMOTE HEAD:refs/heads/$BRANCH'"
```

# Usage
`git partial-push origin master`