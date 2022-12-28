# Snippets

Useful snippets for working with Git.



## Restoring lost commits

```bash
# List where HEAD was three steps ago
git reflog HEAD@{3}
# List where HEAD was three steps ago (as log-entry)
git log HEAD@{3}
```

Even more


## Find largest files in Git repository

List all files ever committed sorted by size

Run in git-bash:

```bash
git rev-list --objects --all \
| git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
| sed -n 's/^blob //p' \
| sort --numeric-sort --key=2 \
| cut -c 1-12,41- \
| $(command -v gnumfmt || echo numfmt) --field=2 --to=iec-i --suffix=B --padding=7 --round=nearest
```
*NOTE*: Add `| tail -n10` at the end to only list the 10 biggest files.


## Diff stats by filetype

List difference to *main* branch per file type.

Run in git-bash:

```bash
git diff --numstat main | python -c"from sys import stdin
import re
stats={}
re_int = re.compile(r'^\d+$', re.MULTILINE)
for l in stdin.readlines():
    insert, delete, fn = l[:-1].split('\t')
    ext = fn.split('.')[-1].replace('}','')
    if ext not in stats:
        stats[ext] = [0,0,0]
    stats[ext][0]+=1
    if re_int.match(insert):
        stats[ext][1] += int(insert)
    if re_int.match(delete):
        stats[ext][2] += int(delete)
for ext in sorted(stats):
    print(f'{ext:<10s} {stats[ext][1]:4d} inserts and {stats[ext][2]:2d} deletes in {stats[ext][0]:2d} files')"
```


## Set user name and e-mail for all git projects in a folder

NOTE: Change *Your Name* to your name and *your_email* to your e-mail before running this :)

```bash
for d in `ls`; do                                                     
    if [ -d "$d/.git" ]; then
        echo "Updating user name and e-mail for $d"
        cd $d
        git config --local user.name "Your Name"                 
        git config --local user.email "your_email@volpi-group.com"
        cd ..
    fi
done
```


## Extract subdirectory as separate branch

Create a new branch containing only the files (and commit history) of the specified PATH.
This can then be converted in a new git repo and added as submodule ot the original one.

```bash
git subtree split -P <PATH> -b <BRANCH>
```
NOTE Make sure path does not start/end with '/'. Eg. instead of `./folder/subfolder/` use `folder/subfolder`.


### Example

Extract folder B of RepoA into separate git repo

Before

```
RepoA
	.git
		RepoA git files
	A
		A_file.txt
	B
		B_file.txt
```

After

```
RepoA
	.git
		RepoA git files
	A
		A_file.txt
	B
		.git
			RepoB git files
		B_file.txt
```

Steps  
NOTE: This assumes an empty branch with url repo_b was created remotely.
```bash
git clone <repo_a> RepoA
cd RepoA
git subtree split -P B -b repo_b_branch
git checkout repo_b_branch
git remote set-url <repo_b>
git remote --set-upstream-to=origin/main repo_b_branch
git push origin HEAD:main
git checkout main
git branch -D repo_b_branch
git rm -r B
git submodule add <repo_b> B
```


