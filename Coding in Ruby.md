Ruby is mostly straightforward, but rails can be tricky.

# Managing the Ruby version

```nixpkgs``` hosts most common versions of ruby. Nix community user Bob Vander Linden manages a separate [repo](https://github.com/bobvanderlinden/nixpkgs-ruby) of every existing ruby version, which can be used if ```nixpkgs``` ceases maintaining some specific version that is needed.

```nix
{ pkgs ? import nixpkgs {} }:

let
  packages = [
    pkgs.ruby
    pkgs.bundix
  ]
```

I currently do this in my home directory, so that ruby is available on a user level. 

# Starting a new project

Create a project directory with a Gemfile:

```sh
bundle init # creates Gemfile
```

As soon as the base project has _some_ files, you can setup version control [Using Git](Using%20Git.md).

# Managing Gems

Ruby refers to their libraries or packages as Gems. They can be managed by ```gem```, ```bundle```, and in Nix: ```bundix```.

```gem``` manages system/user level Gems (try not to use this)
```bundle``` manages project level Gems via ```Gemfile``` and ```Gemfile.lock``` 
```bundix``` takes what is in ```Gemfile.lock``` creates ```gemset.nix```

Install gems like this...

```sh
bundle add <gem-name> # adds gem to Gemfile, Gemfile.lock, and downloads

bundix # adds to gemset.nix, and compiles
```

# Setting up Ruby on Rails

Ruby on rails, even on nix, is got as a Gem. Once got, use ```rails new``` to set up a new project, and install any and all gems rails depends on. Check that rails works by running a server (```bin/rails``` is the safest reference to use). As soon as everything worked, run bundix again to validate all the new gems rails installed by adding them to ```gemset.nix``` and compiling nix store versions of them.

```sh
bundle add rails # get the rails gem

bundix # I guess run bundix now, to validate the rails version

bundle exec rails new . # start new project with all default settings

bin/rails server # make sure template server can run

bundix # add all new installed gems to gemset.nix and compile them
```

When running ```bundix``` at this point, it's very likely to give errors like:
```sh
'Bundix::Nixer#serialize': Cannot convert to nix: nil (RuntimeError)
```

Apparently, the solution I found is to go into Gemfile and and manually remove ```windows``` from ```platforms``` for both ```tzinfo-data``` and ```debug```:
```rb
gem "tzinfo-data", platforms: %i[ windows jruby ]
gem "debug", platforms: %i[ mri windows ], require: "debug/prelude"
```

That should resolve the error and allow ```bundix``` to complete ```gemset.nix``` for the rails project initialization.

# Using Git

I guess it's good to add these to ```.gitignore``` , which rails probably already generated.

```sh
/vendor
/tmp
/log
```

See [Using Git](Using%20Git.md).
