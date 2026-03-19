I need a way to automatically load a nix shell that doesn't accidentally recurse in an infinite loop. Most ways of doing this result in that. For example, trying to open a nix shell at the end of ```.bashrc``` results in an infinite loop, as ```.bashrc``` gets called by the nix shell. 

One way of handling this without being sold into more Nix features is ```direnv```, which is capable of loading and unloading environments as you navigate directories. First, the feature has to been hooked into bash, by putting this line into ```.bashrc```:
```sh
eval "$(direnv hook bash)"
export DIRENV_LOG_FORMAT='' # disable very verbose output
```

Any directory with a ```.envrc``` file in it, ```direnv``` will run that file, and whatever commands are in it. Thus, I set one up in my home directory like this:
```sh
use nix

# Set up a custom temp directory on the user level.
# This also bypasses direnv forgetting to create
# the directory for $TMPDIR after nix sets it up.
mkdir -p $TMPDIR
```

# Conflict with nix-shell

When running ```nix-shell``` manually, Nix will assign a temporary value to ```$TMPDIR```, create that directory, and then remove it when the shell closes. ```direnv``` by design doesn't open a new shell, and so anything Nix does behind the scenes during that process, which seems to include making ```$TMPDIR```, gets skipped. This seems to be more of an issue with ```direnv``` than Nix from what I can tell. Even manually creating the temporary directory still leaves the problem of now having to remove it when existing the shell or environment.

