I want to do any and all of my linux based development from WSL. Most common versions of Linux are derived from Debian, which comes with ```apt```, but as soon as possible after booting the OS, 

# Managing all software and package versions

I want to use Nix package manager where I can, instead of ```apt```. However, I need to use ```apt``` to install ```curl``` to download and install ```nix```. I also need to use ```apt``` to install ```wget``` to download and install VSCode-Server. 

## Apt package versions

I am currently using a script like this in ```.bashrc``` to output all apt package versions to an easily accessible txt file in the home directory:
```sh
## APT PACKAGE TRACKING
echo "Manually installed apt packages:" > ~/apt.txt # Let's start the file at the top
grep -Po '(apt|apt-get) install \K.*' '/var/log/apt/history.log' > ~/apt.tmp # command history
sed -i 's/$/ /' ~/apt.tmp # add a space for better grepping
dpkg-query -W -f='${binary:Package} ${Version}\n' | grep -F -f ~/apt.tmp >> ~/apt.txt # use tmp for grepping
echo "" >> ~/apt.txt # add a newline for better formatting
echo "All installed packages:" >> ~/apt.txt
dpkg-query -W -f='${binary:Package} ${Version}\n' >> ~/apt.txt
echo "" >> ~/apt.txt # add a newline for better formatting
echo "App History:" >> ~/apt.txt
grep -Po '(Install|Upgrade|Remove): .*' /var/log/apt/history.log | perl -pe 's/, (?![^\(]*\))/\n\t/g' >> ~/apt.txt # perl replacement of commas not enclosed in parenthesis
rm ~/apt.tmp # remove the temporary file
```
```~/apt.txt```:
```txt
Manually installed apt packages:
curl 8.14.1-2+deb13u2
direnv 2.32.1-2+b16
git 1:2.47.3-0+deb13u1
wget 1.25.0-2
xz-utils 5.8.1-1

All installed packages:
adduser 3.152
apt 3.0.3
apt-utils 3.0.3
base-files 13.8+deb13u2
...
```

# Manually installed software versions

VSCode / VSCode-Server needs to be installed by ```wget``` which does a manual installation in ```.vscode-server/```.  Curl also installs Nix somewhere. I can get the code version from ```.vscode-server/bin/*/package.json``` , and the Nix version is simply got with ```nix --version```. I use that to make an easily accessible file for describing my manually installed software and versions.

```txt
Non-apt installed programs:
1.111.0
ce099c1ed25d9eb3076c11e4a280f3eb52b4fbeb
x64
{
  "name": "Code",
  "version": "1.111.0",
  "private": true,
  "overrides": {
    "node-gyp-build": "4.8.1",
    "kerberos@2.1.1": {
      "node-addon-api": "7.1.0"
    }
  },
  "type": "module"
}
nix-env (Nix) 2.34.0
```

# Nix managed software

There are Nix packages I want available on an OS User level, but I _do_ _not_ want to manage them  globally or even in the User's nix-profile. I want to contain Nix more carefully by simply using a ```nix-shell``` in ```~/``` that is automatically opened and closed using ```direnv```. I want to avoid having to use User-Profiles, Channels, Flakes,  or Home-Manager.

# nixpkgs version

All software installed through nix comes from the nixpkgs repository, hosted on github: https://github.com/NixOS/nixpkgs This repository is updated several times a day, and so anything built from it can potentially change if you don't specifically maintain a precise version. I do this with ```nixpkgs-version.txt```:
```nix
let
  downloadFromGithub = { version, name, owner, sha256 }:
    builtins.fetchTarball {
      name = "${name}-${version}";
      url = "https://github.com/${owner}/${name}/archive/${version}.tar.gz";
      inherit sha256;
    };

in
{
  # Use the date you started using it
  march_16_2026 = downloadFromGithub {
    version = "917fec9";
    owner = "NixOS";
    name = "nixpkgs";
    sha256 = "1x3hmj6vbza01cl5yf9d0plnmipw3ap6y0k5rl9bl11fw7gydvva";
  };
}
```

I plan to save all versions, and simply comment out or not use them when switching to new ones. Any version can then be imported into the home nix shell:
```nix
let
  nixpkgs = (import ./nixpkgs-versions.nix).march_16_2026;

in
  { pkgs ? import nixpkgs {} }:
```

# Nix packages versions

Put a Nix shell script in home that will control any OS User level packages I want available.

```nix
let
  nixpkgs = (import ./nixpkgs-versions.nix).march_16_2026;

in
  { pkgs ? import nixpkgs {} }:

    let
      packages = [
        # Ruby language
        pkgs.ruby
        pkgs.bundix # nix style gemset management

        # Rails support
        pkgs.nodejs # for js asset bundling
        pkgs.sqlite # database

        # Python language
        pkgs.python315
        pkgs.uv # python package and environment manager

        # General language support
        pkgs.gcc        # c-compiling
        pkgs.openssl    # networking
        pkgs.zlib       # compression
        pkgs.libffi     # for foreign function interfaces
        pkgs.libyaml    # for faster YAML parsing
        pkgs.pkg-config # for building native extensions
        pkgs.libxml2
        pkgs.libxslt
        pkgs.yarn
      ];

    in pkgs.mkShell {
      inherit packages; 

      shellHook = ''
        # bundix fails when it tries to write to /tmp, changing it to $HOME/tmp works fine
        export TMPDIR=$HOME/tmp
        mkdir -p $TMPDIR

        # Print out the packages in the nix shell to a file for easy reference
        rm nix.txt
        ${builtins.concatStringsSep "\n"
          (map (p: "echo ${p.pname or p.name} ${p.version or ""} >> ~/nix.txt" ) packages)}
          
        ~/.welcome.sh
      '';
    }
```

The ```shellHook``` portion will run when the shell finishes loading. At this point, Nix knows the specific versions of the packages, so we can write it to a file called ```nix.txt```.
