
Version control, and managing copies and work progress is all facilitated with git.

# The home project

Host the home setup publicly, so it is easily retrievable, and so people can see and critique or contribute to it. The home project is whatever custom OS configuration I do.  This is actually annoying, because git doesn't like cloning into an existing repo. So I have to close into a directory, then move the files out of that directory. This home setup, however, is only meaningful for WSL. Trying to view a copy in the Windows home directory isn't actually useful, there is no reason for me to access these on Windows. If anything the Windows home directory is more useful for various Windows tools like Obsidian.

```sh
git remote add origin https://github.com/me/winhome.git
```
# Setting up a new project

I want to host remotes on Windows, with the ability to develop on WSL, and still see all the files in Windows. 

Start with a WSL development branch in the correct location ```~/projects```

```sh
$:~/projects git init <language>_projects/<project-name>
```

# Setting up the remote
```setupremote``` will call ```~/projects/.setup_remote.sh```:

```sh
# ASSERT: must be at correct project directory dept (right now it is 6)
depth=$(pwd | awk -F/ '{print NF}') # split by / and count number of fields
required_depth=6
if [ "$depth" != "$required_depth" ]; then
    echo "need to be in /home/projects/<language>/<project-name>/ to setup remote"
    exit 1
fi

# ASSERT: must have made first commit
if ! git diff-index --quiet HEAD --; then
    echo "need to make first commit to setup remote"
    exit 1
fi

PROJECT_DIR="$(basename "$(dirname "$PWD")")/$(basename "$PWD")"
REMOTE_DIR="$WINHOME_DIR/remotes/$PROJECT_DIR"
WIN_WORKING_DIR="$WINHOME_DIR/projects/$PROJECT_DIR"
WORKING_DIR="$HOME/projects/$PROJECT_DIR"

# setup remote repo on windows...
git init --bare "$REMOTE_DIR.git" # remote, not a repo

# add that repo to the working project
git remote add winrepo "$REMOTE_DIR.git"

# first push needs to set upstream (good reason to require first commit)
git push --set-upstream winrepo master

# make a windows clone so we can view stuff on windows...
git clone "$REMOTE_DIR.git" "$WIN_WORKING_DIR"

exit 0
```

If you need to update the remote, you will need to call this first...

```sh
git remote remove winrepo
```

After this ```pushandupdate``` will handle pushing to the remote and updating the windows copy of the directory.

