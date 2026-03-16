
Use UV to manage python packages. It also runs an virtual environment for you.

```sh
uv add numpy
uv run project.py
```

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
