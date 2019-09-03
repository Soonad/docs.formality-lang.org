# Installation

Right now, Formality can only be installed through npm. Install npm following
[this guide](https://www.npmjs.com/get-npm). Then, go to the command-line and
type:

```
npm i -g formality-lang
```

Using [`node2nix`](https://github.com/svanderburg/node2nix#installation), we can
also install Formality using the Nix package manager:

```
$ nix-env -f '<nixpkgs>' -iA nodePackages.node2nix
$ node2nix --nodejs-12
$ nix-env -f default.nix formality-lang
```

If nodejs-12 is not in your nixpkgs channel, then build from unstable with:

```
$ node2nix --nodejs-12
$ nix-channel add https://nixos.org/channels/nixpkgs-unstable unstable
$ sed -i 's/nixpkgs/unstable/g' default.nix
$ nix-env -f default.nix formality-lang
```

This should be all you need. In order to test if it worked, type `fm` on the
terminal. If you see Formality's command-line options, then it has been
successfully installed in your system. If you have any problem during this
process, please [open an issue](https://github.com/moonad/Formality/issues).
