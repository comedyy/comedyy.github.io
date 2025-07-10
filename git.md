# GIT 相关

- [git submodule](./git/submodule.md)

merge的时候忽略lineending
git merge feature/TestLockStep --squash -s recursive -Xignore-space-at-eol


[alias]
	s = !git status
	x = !git clean -df && git reset --hard
	r = !git pull --rebase && git push

