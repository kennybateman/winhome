Use Nix to control Python version. Use UV to manage python packages. 
# Managing the version of Python and UV

```~/shell.nix```:
```nix
{ pkgs ? import <nixpkgs> {} }:

let
  packages = [
    pkgs.python315
    pkgs.uv
  ]
```

```~/nix.txt```:
```txt
python3 3.15.0a6
uv 0.10.6
```

# Adding packages

```sh
uv add numpy
```

# Run project

```sh
uv run project.py
```