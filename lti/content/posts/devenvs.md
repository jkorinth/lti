+++
date = '2026-06-20T23:35:17+02:00'
draft = true
title = 'What I love about Nix devShells'
tags = [ 'nix', 'devenvs', 'qol', 'devlife' ]
+++

IMO one of _the_ best features of Nix flakes are `devShells`. Why? They let me leave a
project for any amount of time and let me jump right back in later. Key to this is
`direnv`, which allows you to ephemerally modify your environment when you `cd` into a
directory, and will take care of cleaning up after. `nix develop` is nice, but if you
live in the terminal, you don't want to lose your customized shell just to get your
tools back.

So, here's how I setup almost every new project these days:

```sh
git init p
cd p
echo use flake https://github.com/jkorinth/devenvs/dev#zephyr > .envrc
direnv allow .
```

Whenever I cd into `p` now, it will automatically set me up with a shell where I have
all the tools I need for Zephyr RTOS development. In the beginning I used to have a
local `.nix/` folder, which would contain a `flake.nix` (and its `flake.lock`). Nowadays
I move most of my modifications directly to my `devenvs` repo[^1]. But Nix also makes it
pretty simple to extend one of the `devShells`, if need be:

A `flake.nix` for a devShell always starts with the usual stuff:

```nix
{
  inputs = {
    nixpkgs.url = "nixpkgs/nixos-unstable";         # <- or whatever version you'd like
    flake-utils.url = "github:numtide/flake-utils"; # removes a lot of boiler plate
  };

  outputs = { nixpkgs, flake-utils, ... }:
    flake-utils.lib.eachDefaultSystem(system:       # <- picks your current system automatically
      let
        pkgs = import nixpkgs { inherit system; };  # <- take nix packages for your system
      in {
        devShell = pkgs.mkShell {                   # <- make a shell
           ...
        };
      });
}
```

Once you've gotten used to the weirdness of the Nix language, it's actually pretty
straightforward to reference an external project:

```nix
{
  inputs = {
    ...
    devenvs.url = "github:jkorinth/devenvs/dev";     # <- fetch my own
  };

  outputs = { nixpkgs, flake-utils, devenvs, ... }:
    flake-utils.lib.eachDefaultSystem(system:       # <- picks your current system automatically
      let
        pkgs = import nixpkgs { inherit system; };  # <- take nix packages for your system

      in {
        devShell = pkgs.mkShell {                   # <- make a shell...
          inputsFrom =
            [ devenvs.devsShells.${system}.zephyr ];# <- ...based on one of mine
          ...                                       # <- customize!
        };
      });
}
```

This allows you to do other tricks, like using a specific version of `nixpkgs`,
regardless of what the `devenvs` repo declares:

```nix
{
  inputs = {
    nixpkgs.url = "nixpkgs/nixos-25.11";        # <- use 25.11
    ...
    devenvs = {
      url = "github:jkorinth/devenvs/dev";
      inputs.nixpkgs.follows = "nixpkgs";       # <- follow the nixpkgs from here
    };
  };
  ...
}
```

Nix and NixOS were/are the greatest gamechangers in my #devlife for the past decade.
:rocket: :raised_hands:


[^1]: [https://github.com/jkorinth/devenvs](https://github.com/jkorinth/devenvs)
