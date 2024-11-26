# Making a dev shell with nix flakes (fasterthanli.me)

<https://fasterthanli.me/series/building-a-rust-service-with-nix/part-10#a-flake-with-mkderivation>

## Description

In the previous chapter, we've made a nix 'dev shell' that contained the fly.io command-line utility, 'flyctl'. That said, that's not how I want us to define a dev shell. Our current solution has i...

## Tags

#nix

## Content

# Making a dev shell with nix flakes {#making-a-dev-shell-with-nix-flakes .reader-title}

Amos Wenger

------------------------------------------------------------------------

Mar 05, 2023
![clock icon](https://cdn.fasterthanli.me/content/img/clock~37781bbae6898349.svg){#clock-icon height="16" width="16"}
32 min
[#nix](https://fasterthanli.me/tags/nix)

In the previous chapter, we\'ve made a nix \"dev shell\" that contained the fly.io
command-line utility, \"flyctl\".

That said, that\'s not how I want us to define a dev shell.

Our current solution has issues. I don\'t like that it has `import <nixpkgs>`.
Which version of `nixpkgs` is that? The one you\'re on? Who knows what that is.

Also, we haven\'t really seen a mechanism to use `.nix` files from elsewhere.

As you may be beginning to suspect, building Rust code (or a Node.js app, etc.)
with nix is actually non-trivial, because we need to hash the whole input,
download everything in advance (did I mention builders don\'t have access to the
internet? to make double-plus-sure everything is reproducible?), use the right
toolchain, etc.

So we won\'t be using `mkDerivation` at all - we\'ll be using a third-party Nix
library. And we need a way to refer to it cleanly.

Those problems are solved with nix flakes - part of the new, experimental,
don\'t-cry-if-it-breaks stuff.

[](#a-flake-with-derivation){#a-flake-with-derivation}

## A flake with `derivation`

A nix flake is just a regular nix file! It needs to evaluate to a set with
`inputs` and `outputs`, and some others if we feel fancy.

So, let\'s remove `shell.nix`, and create a mostly-empty `flake.nix`:

This isn\'t enough, and any flakes-enabled commands (the new stuff, the `nix`
subcommands, not the `nix-dash-something` commands) should let us know, like,
`nix flake check`:

Shell session

``` {data-lang="shell"}
$ nix flake check
error: getting status of '/nix/store/zfl0kgix0a8kz9jny1sjbvwrgk8syyjb-source/flake.nix': No such file or directory
```

Wait, no, that\'s\... that\'s not the error I expected. Due to how flakes work, the
`flake.nix` file needs to be *staged* into our Git repository, so let\'s do that:

Shell session

``` {data-lang="shell"}
$ git add flake.nix

$ nix flake check
warning: Git tree '/home/amos/catscii' is dirty
error: flake 'git+file:///home/amos/catscii' lacks attribute 'outputs'
```

So, fine, let\'s add outputs:

nix

``` {data-lang="nix"}
# in flake.nix
{
  outputs = { };
}
```

Shell session

``` {data-lang="shell"}
$ nix flake check
warning: Git tree '/home/amos/catscii' is dirty
error: expected a function but got a set at /nix/store/8nwynxhiiyrg78rsfpqa61nfal8yzyrh-source/flake.nix:3:3
```

Ah, `outputs` should be a function. A function of its `inputs`, no doubt:

nix

``` {data-lang="nix"}
# in flake.nix
{
  outputs = inputs: { };
}
```

Shell session

``` {data-lang="shell"}
$ nix flake check
warning: Git tree '/home/amos/catscii' is dirty
```

And just like that, we\'ve made our first, completely useless flake.

Let\'s make it useful! One thing a flake can have as an output is\... an
executable! We can use `derivation` just like we did before!

The only difference is, now we specify *where* nixpkgs comes from:

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    # this is equivalent to `nixpkgs = { url = "..."; };`
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  };
  outputs = inputs: {
    packages."x86_64-linux".default = derivation {
      name = "simple";
      builder = "${inputs.nixpkgs.legacyPackages."x86_64-linux".bash}/bin/bash";
      args = [ "-c" "echo foo > $out" ];
      src = ./.;
      system = "x86_64-linux";
    };
  };
}
```

And if we do `nix build`, it builds just like before.

Note that this is `nix build` with a space, as in \"the build subcommand of nix\",
not \"the nix-build command\".

Shell session

``` {data-lang="shell"}
$ nix build
warning: Git tree '/home/amos/catscii' is dirty

$ cat result
foo
```

And in `flake.lock`, our inputs are locked:

JSON

``` {data-lang="json"}
{
  "nodes": {
    "nixpkgs": {
      "locked": {
        "lastModified": 1676481215,
        "narHash": "sha256-afma/1RU0EePRyrBPcjBdOt+dV8z1bJH9dtpTN/WXmY=",
        "owner": "NixOS",
        "repo": "nixpkgs",
        "rev": "28319deb5ab05458d9cd5c7d99e1a24ec2e8fc4b",
        "type": "github"
      },
      "original": {
        "owner": "NixOS",
        "ref": "nixos-unstable",
        "repo": "nixpkgs",
        "type": "github"
      }
    },
    "root": {
      "inputs": {
        "nixpkgs": "nixpkgs"
      }
    }
  },
  "root": "root",
  "version": 7
}
```

\...which we can update with `nix flake update`.

We can also show the outputs our flake has with `nix flake show`:

Shell session

``` {data-lang="shell"}
$ nix flake show
warning: Git tree '/home/amos/catscii' is dirty
git+file:///home/amos/catscii
└───packages
    └───x86_64-linux
        └───default: package 'simple'
```

However, this isn\'t a very good flake, yet.

The code is redundant all over.

Instead of referring to `inputs.stuff`, we could destructure the arguments of
`outputs`:

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    # this is equivalent to `nixpkgs = { url = "..."; };`
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  };
  # now a set 👇
  outputs = { self, nixpkgs }: {
    packages."x86_64-linux".default = derivation {
      name = "simple";
      # was `inputs.nixpkgs`, now just:
      builder = "${nixpkgs.legacyPackages."x86_64-linux".bash}/bin/bash";
      args = [ "-c" "echo foo > $out" ];
      src = ./.;
      system = "x86_64-linux";
    };
  };
}
```

And then, we can use `let` to great effect:

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  };
  outputs = { self, nixpkgs }:
    let
      # define system once
      system = "x86_64-linux";
      # use it here, and bind platform-specific packages to `pkgs`
      pkgs = nixpkgs.legacyPackages.${system};
    in
    {
      # we can use `${system}` here!
      packages.${system}.default = derivation {
        name = "simple";
        # with `with`, we can just do `bash`
        builder = with pkgs; "${bash}/bin/bash";
        args = [ "-c" "echo foo > $out" ];
        src = ./.;
        system = system;
      };
    };
}
```

There. Isn\'t that much tidier?

But see that `system = system` there? That\'s not very [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

nix has a thing for this: `inherit`. In fact, we can inherit a bunch of things:

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  };
  outputs = { self, nixpkgs }:
    let
      system = "x86_64-linux";
      name = "simple";
      src = ./.;
      pkgs = nixpkgs.legacyPackages.${system};
    in
    {
      packages.${system}.default = derivation {
        inherit system name src;
        builder = with pkgs; "${bash}/bin/bash";
        args = [ "-c" "echo foo > $out" ];
      };
    };
}
```

This version of the flake does exactly the same, but it\'s clearer! (We didn\'t
really need to inherit `name` and `src`, but I wanted to show it off).

Finally, our flake doesn\'t *only* work on `x86_64-linux`. We could define it
manually for a bunch of other targets, or we could use a nix library to do it
for us!

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    # 👇 we have a new input!
    flake-utils.url = "github:numtide/flake-utils";
  };
  #      it's destructured here 👇
  outputs = { self, nixpkgs, flake-utils }:
    # calling a function from `flake-utils` that takes a lambda
    # that takes the system we're targetting
    flake-utils.lib.eachDefaultSystem (system:
      let
        # no need to define `system` anymore
        name = "simple";
        src = ./.;
        pkgs = nixpkgs.legacyPackages.${system};
      in
      {
        # `eachDefaultSystem` transforms the input, our output set
        # now simply has `packages.default` which gets turned into
        # `packages.${system}.default` (for each system)
        packages.default = derivation {
          inherit system name src;
          builder = with pkgs; "${bash}/bin/bash";
          args = [ "-c" "echo foo > $out" ];
        };
      }
    );
}
```

Any flake-enabled command updates `flake.lock` (since we have a new input),
`nix flake show` now shows that we can build for a bunch of common targets:

Shell session

``` {data-lang="shell"}
$ nix flake show
warning: Git tree '/home/amos/catscii' is dirty
git+file:///home/amos/catscii
└───packages
    ├───aarch64-darwin
    │   └───default: package 'simple'
    ├───aarch64-linux
    │   └───default: package 'simple'
    ├───x86_64-darwin
    │   └───default: package 'simple'
    └───x86_64-linux
        └───default: package 'simple'
```

Note that `nix build` still defaults to building.. the default package, for
our current platform.

We can tell, if we ask it for extra debugging output with `--debug`:

Shell session

``` {data-lang="shell"}
$ nix build --debug
evaluating derivation 'git+file:///home/amos/catscii#packages.x86_64-linux.default'...
warning: Git tree '/home/amos/catscii' is dirty
acquiring write lock on '/nix/var/nix/temproots/39176'
got tree '/nix/store/m0wdkb3rggsnr66z98xppxrq2crwc8ia-source' from 'git+file:///home/amos/catscii'
checking access to '/nix/store/m0wdkb3rggsnr66z98xppxrq2crwc8ia-source/flake.nix'
evaluating file '/nix/store/m0wdkb3rggsnr66z98xppxrq2crwc8ia-source/flake.nix'
old lock file: {
  (cut)
}
computing lock file node ''
computing input 'flake-utils'
keeping existing input 'flake-utils'
computing lock file node 'flake-utils'
computing input 'nixpkgs'
keeping existing input 'nixpkgs'
computing lock file node 'nixpkgs'
new lock file: {
  (cut)
}
trying flake output attribute 'packages.x86_64-linux.default'
using cached string attribute 'packages.x86_64-linux.default.type'
using cached string attribute 'packages.x86_64-linux.default.drvPath'
querying info about missing paths...
starting pool of 24 threads
building of '/nix/store/0g5ggy17bfvys0lxfw5wdyrsapfcpz9q-simple.drv!out' from .drv file: created
building of '/nix/store/0g5ggy17bfvys0lxfw5wdyrsapfcpz9q-simple.drv!out' from .drv file: woken up
querying info about missing paths...
starting pool of 24 threads
entered goal loop
building of '/nix/store/0g5ggy17bfvys0lxfw5wdyrsapfcpz9q-simple.drv!out' from .drv file: init
building of '/nix/store/0g5ggy17bfvys0lxfw5wdyrsapfcpz9q-simple.drv!out' from .drv file: loading derivation
building of '/nix/store/0g5ggy17bfvys0lxfw5wdyrsapfcpz9q-simple.drv!out' from .drv file: have derivation
building of '/nix/store/0g5ggy17bfvys0lxfw5wdyrsapfcpz9q-simple.drv!out' from .drv file: done
building of '/nix/store/0g5ggy17bfvys0lxfw5wdyrsapfcpz9q-simple.drv!out' from .drv file: goal destroyed
```

We can also build it more explicitly, by passing the path of the thing to build:

Shell session

``` {data-lang="shell"}
$ nix build .
```

Or we can be more specific:

Shell session

``` {data-lang="shell"}
$ nix build .#default
```

Or even *more* specific:

Shell session

``` {data-lang="shell"}
$ nix build .#packages.x86_64-linux.default
```

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

Wait, does that mean we can cross-compile?

It\'s, uhhh, complicated. I haven\'t cracked it at the time of this writing.

If you try the obvious thing, it fails somewhat gracefully, at least:

Shell session

``` {data-lang="shell"}
$ nix build .#packages.x86_64-darwin.default
warning: Git tree '/home/amos/catscii' is dirty
error: a 'x86_64-darwin' with features {} is required to build '/nix/store/3k8h1zqdm5l8c64l2i8bpxcmxdff4fi1-simple.drv', but I am a 'x86_64-linux' with features {benchmark, big-parallel, nixos-test, uid-range}
```

[](#a-flake-with-mkderivation){#a-flake-with-mkderivation}

## A flake with `mkDerivation`

Just for completeness\' sake, let\'s see how our `shaka-packager` package would
look like as flake.

Instead of removing our default package, we\'ll simply add it as another output:

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };
  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        name = "simple";
        src = ./.;
        pkgs = nixpkgs.legacyPackages.${system};
      in
      {
        packages.default = derivation {
          inherit system name src;
          builder = with pkgs; "${bash}/bin/bash";
          args = [ "-c" "echo foo > $out" ];
        };
        packages.shaka-packager =
          let
            version = "2.6.1";
            inherit (pkgs) stdenv lib;
          in
          stdenv.mkDerivation
            {
              name = "shaka-packager-${version}";

              # https://nixos.wiki/wiki/Packaging/Binaries
              src = pkgs.fetchurl {
                url =
                  "https://github.com/shaka-project/shaka-packager/releases/download/v${version}/packager-linux-x64";
                sha256 = "sha256-MoMX6PEtvPmloXJwRpnC2lHlT+tozsV4dmbCqweyyI0=";
              };

              dontUnpack = true;
              sourceRoot = ".";

              installPhase = ''
                install -m755 -D $src $out/bin/shaka-packager
              '';

              meta = with nixpkgs.lib; {
                homepage = "https://shaka-project.github.io/shaka-packager/html/";
                description =
                  "Media packaging framework for VOD and Live DASH and HLS applications";
                platforms = platforms.linux;
              };
            };
      }
    );
}
```

There! Isn\'t that neat?

Shell session

``` {data-lang="shell"}
$ nix flake show
warning: Git tree '/home/amos/catscii' is dirty
git+file:///home/amos/catscii
└───packages
    ├───aarch64-darwin
    │   ├───default: package 'simple'
    │   └───shaka-packager: package 'shaka-packager-2.6.1'
    ├───aarch64-linux
    │   ├───default: package 'simple'
    │   └───shaka-packager: package 'shaka-packager-2.6.1'
    ├───x86_64-darwin
    │   ├───default: package 'simple'
    │   └───shaka-packager: package 'shaka-packager-2.6.1'
    └───x86_64-linux
        ├───default: package 'simple'
        └───shaka-packager: package 'shaka-packager-2.6.1'
```

And we can build it with:

Shell session

``` {data-lang="shell"}
$ nix build .#shaka-packager
```

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

Wait.. isn\'t it downloading an `x86_64-linux` binary for `shaka-packager`?

Oh, you\'re right! This one should *not* be for all targets.

Well, what a good excuse to learn about the `//`! It returns the union of two
sets, preferring the right-hand-side if there\'s conflicts.

Note that this doesn\'t do a deep merge, so we need to expand things a little:

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };
  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem
      (system:
        let
          name = "simple";
          src = ./.;
          pkgs = nixpkgs.legacyPackages.${system};
        in
        {
          packages = {
            default = derivation {
              inherit system name src;
              builder = with pkgs; "${bash}/bin/bash";
              args = [ "-c" "echo foo > $out" ];
            };
          } // (if system == "x86_64-linux" then {
            shaka-packager =
              let
                version = "2.6.1";
                pkgs = nixpkgs.legacyPackages.x86_64-linux;
                inherit (pkgs) stdenv lib;
              in
              stdenv.mkDerivation
                {
                  name = "shaka-packager-${version}";

                  # https://nixos.wiki/wiki/Packaging/Binaries
                  src = pkgs.fetchurl {
                    url =
                      "https://github.com/shaka-project/shaka-packager/releases/download/v${version}/packager-linux-x64";
                    sha256 = "sha256-MoMX6PEtvPmloXJwRpnC2lHlT+tozsV4dmbCqweyyI0=";
                  };

                  dontUnpack = true;
                  sourceRoot = ".";

                  installPhase = ''
                    install -m755 -D $src $out/bin/shaka-packager
                  '';

                  meta = with nixpkgs.lib; {
                    homepage = "https://shaka-project.github.io/shaka-packager/html/";
                    description =
                      "Media packaging framework for VOD and Live DASH and HLS applications";
                    platforms = platforms.linux;
                  };
                };
          } else { });
        }
      );
}
```

And now we have the expected output:

Shell session

``` {data-lang="shell"}
$ nix flake show
warning: Git tree '/home/amos/catscii' is dirty
git+file:///home/amos/catscii
└───packages
    ├───aarch64-darwin
    │   └───default: package 'simple'
    ├───aarch64-linux
    │   └───default: package 'simple'
    ├───x86_64-darwin
    │   └───default: package 'simple'
    └───x86_64-linux
        ├───default: package 'simple'
        └───shaka-packager: package 'shaka-packager-2.6.1'
```

Note that of course, `shaka-packager` has builds for all these platforms, so we
could have our flake translate `system` to the proper download URL, with the
proper sha256.

But this is not at all what we\'re trying to do here, so let\'s not.

[](#a-flake-with-a-dev-shell){#a-flake-with-a-dev-shell}

## A flake with a dev shell

Let\'s start our `flake.nix` over, since we got distracted with learning nix and
stuff.

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };
  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem
      (system:
        { }
      );
}
```

There.

The most pressing thing we need in our dev shell is `cargo` / `rustc` - remember
that? Way at the beginning of the article?

That\'s still what we\'re trying to do.

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

But at least, now we know enough of nix to do it easily, right?

Well\..... almost. We haven\'t talked about overlays yet. Remember how we said the
result of an `import <nixpkgs>` is a function that can be called with a set?

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

Yes?

Well, that\'s how we do overlays. Which provide packages that aren\'t in nixpkgs
by default.

For example, `github:oxalica/rust-overlay` provides various versions of the Rust
toolchain.

We can do, for example:

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
    rust-overlay.url = "github:oxalica/rust-overlay";
  };
  outputs = { self, nixpkgs, flake-utils, rust-overlay }:
    flake-utils.lib.eachDefaultSystem
      (system:
        let
          overlays = [ (import rust-overlay) ];
          pkgs = import nixpkgs {
            inherit system overlays;
          };
        in
        with pkgs;
        {
          devShells.default = mkShell {
            buildInputs = [ rust-bin.stable.latest.default ];
          };
        }
      );
}
```

And then:

Shell session

``` {data-lang="shell"}
$ nix develop
warning: Git tree '/home/amos/catscii' is dirty
warning: updating lock file '/home/amos/catscii/flake.lock':
• Added input 'rust-overlay':
    'github:oxalica/rust-overlay/3bab7ae4a80de02377005d611dc4b0a13082aa7c' (2023-02-16)
• Added input 'rust-overlay/flake-utils':
    'github:numtide/flake-utils/c0e246b9b83f637f4681389ecabcb2681b4f3af0' (2022-08-07)
• Added input 'rust-overlay/nixpkgs':
    'github:NixOS/nixpkgs/14ccaaedd95a488dd7ae142757884d8e125b3363' (2022-10-09)
warning: Git tree '/home/amos/catscii' is dirty

$ cargo --version
cargo 1.67.1 (8ecd4f20a 2023-01-10)

$
(pressing Ctrl-D)
exit
```

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

Ah, finally!

Wait no no no, hold on a minute.

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

What. What? It does the thing!

It certainly does, but did you notice how it added inputs:

``` {data-lang=""}
• Added input 'rust-overlay/flake-utils':
• Added input 'rust-overlay/nixpkgs':
```

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

Well?

We already have those! And now we have some duplicates!

Shell session

``` {data-lang="shell"}
$ nix flake metadata
warning: Git tree '/home/amos/catscii' is dirty
Resolved URL:  git+file:///home/amos/catscii
Locked URL:    git+file:///home/amos/catscii
Path:          /nix/store/1wfcnmimbrhkckp95k7nm94zdimrqr08-source
Last modified: 2023-02-17 00:18:04
Inputs:
├───flake-utils: github:numtide/flake-utils/3db36a8b464d0c4532ba1c7dda728f4576d6d073
├───nixpkgs: github:NixOS/nixpkgs/28319deb5ab05458d9cd5c7d99e1a24ec2e8fc4b
└───rust-overlay: github:oxalica/rust-overlay/3bab7ae4a80de02377005d611dc4b0a13082aa7c
    ├───flake-utils: github:numtide/flake-utils/c0e246b9b83f637f4681389ecabcb2681b4f3af0
    └───nixpkgs: github:NixOS/nixpkgs/14ccaaedd95a488dd7ae142757884d8e125b3363
```

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

Ah, well. Isn\'t that kinda inherent to the flake model?

Mhhhhhsorta but we can fix it:

nix

``` {data-lang="nix"}
  # in flake.nix (only inputs shown)
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
    rust-overlay = {
      url = "github:oxalica/rust-overlay";
      inputs = {
        nixpkgs.follows = "nixpkgs";
        flake-utils.follows = "flake-utils";
      };
    };
  };
```

Let\'s check again:

Shell session

``` {data-lang="shell"}
$ nix flake metadata
warning: Git tree '/home/amos/catscii' is dirty
Resolved URL:  git+file:///home/amos/catscii
Locked URL:    git+file:///home/amos/catscii
Path:          /nix/store/b2lxpiqs6g58gzrxir2q20nisvjz0kks-source
Last modified: 2023-02-17 00:18:04
Inputs:
├───flake-utils: github:numtide/flake-utils/3db36a8b464d0c4532ba1c7dda728f4576d6d073
├───nixpkgs: github:NixOS/nixpkgs/28319deb5ab05458d9cd5c7d99e1a24ec2e8fc4b
└───rust-overlay: github:oxalica/rust-overlay/3bab7ae4a80de02377005d611dc4b0a13082aa7c
    ├───flake-utils follows input 'flake-utils'
    └───nixpkgs follows input 'nixpkgs'
```

Ah, much better!

![Amos](https://cdn.fasterthanli.me/content/img/characters/classic-amos~d3ec363ae5682c68.svg){height="42" width="42"}

Making sure you have the same inputs, *down to the commit* is important to avoid
\"rebuild the world\" situations, as Pyrox explains [on GitHub](https://github.com/fasterthanlime/feedback/issues/242).

So, `nix-shell` is the old stuff, `nix develop` is the new, flake-enabled stuff.
Notably, it\'s much faster than `nix-shell`, so it\'s not *just* novelty for
novelty\'s sake.

(Also, for me at least, it doesn\'t change the shell prompt, which can be a bit
confusing - it\'s hard to tell when you\'re in the dev shell or not).

The next thing I\'m unhappy with, is that it gave us the latest stable:

Shell session

``` {data-lang="shell"}
$ nix develop --command rustc --version
warning: Git tree '/home/amos/catscii' is dirty
rustc 1.67.1 (d5a82bbd2 2023-02-07)
```

\...when we have a `rust-toolchain.toml` file.

We can fix that too!

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
    rust-overlay = {
      url = "github:oxalica/rust-overlay";
      inputs = {
        nixpkgs.follows = "nixpkgs";
        flake-utils.follows = "flake-utils";
      };
    };
  };
  outputs = { self, nixpkgs, flake-utils, rust-overlay }:
    flake-utils.lib.eachDefaultSystem
      (system:
        let
          overlays = [ (import rust-overlay) ];
          pkgs = import nixpkgs {
            inherit system overlays;
          };
          # 👇 new! note that it refers to the path ./rust-toolchain.toml
          rustToolchain = pkgs.pkgsBuildHost.rust-bin.fromRustupToolchainFile ./rust-toolchain.toml;
        in
        with pkgs;
        {
          devShells.default = mkShell {
            # 👇 we can just use `rustToolchain` here:
            buildInputs = [ rustToolchain ];
          };
        }
      );
}
```

And now, we have the proper version:

Shell session

``` {data-lang="shell"}
$ nix develop --command rustc --version
warning: Git tree '/home/amos/catscii' is dirty
rustc 1.68.0-nightly (af3e06f1b 2022-12-23)
```

Yay! Finally!

[](#setting-up-direnv-and-nix-direnv){#setting-up-direnv-and-nix-direnv}

## Setting up direnv and nix-direnv

We\'re not quite done yet, though. There\'s still a couple things missing, but
before we move forward, let\'s set up a thing that \"enters the dev environment\"
anytime we cd into the directory.

We\'ll use the [direnv](https://direnv.net/) tool for that, just like we did in part 4.

We could install it with apt, or\... we could use `nix profile`! Let\'s try the
latter:

Shell session

``` {data-lang="shell"}
$ nix profile install nixpkgs#direnv
(cut)
```

Just like last time, we\'ll set up the bash hook for it (it works similarly for
zsh and others, if you couldn\'t resist switching to them already):

Shell session

``` {data-lang="shell"}
$ echo 'eval "$(direnv hook bash)"' >> ~/.bashrc
```

We already have an `.envrc` file in there, and it\'s already encrypted by
`git-crypt`, but\... we haven\'t decrypted it yet.

Our `.envrc` is a little hard to read right now:

Shell session

``` {data-lang="shell"}
$ xxd .envrc | head -3
00000000: 0047 4954 4352 5950 5400 39b8 4abe 420d  .GITCRYPT.9.J.B.
00000010: f7a1 1fde 1ba2 9136 eda4 0fa1 0550 ee0f  .......6.....P..
00000020: e3aa d4db d004 3e0c e387 2003 e23e 3af9  ......>... ..>:.
```

So, remember when I told you to save your `git-crypt-key` file? Time to copy it
somewhere into your VM once again, and then install git-crypt.

If you *haven\'t* saved your `git-crypt-key` file, bummer, but you can find or
regenerate honeycomb tokens etc., and re-write the file from scratch, generate
a new `git-crypt-key`, and lock it again with git-crypt.

Shell session

``` {data-lang="shell"}
$ nix profile install nixpkgs#git-crypt
(cut)
```

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

Wait, wouldn\'t it make sense to have it as part of the dev shell?

Mhhh not really: the instructions to activate the dev shell when cd-ing into
the directory are in the .envrc, so it needs to be already decrypted by that
time.

So \"having git-crypt already installed before you start hacking on catscii\"
is going to be a hard requirement for us.

Let\'s proceed:

Shell session

``` {data-lang="shell"}
$ amos@eggman:~/catscii$ git-crypt unlock ~/catscii-git-crypt-key
direnv: error /home/amos/catscii/.envrc is blocked. Run `direnv allow` to approve its content
```

That looks better:

Shell session

``` {data-lang="shell"}
$ head .envrc | grep --only-matching --extended-regexp 'export [^=]+'
export SENTRY_DSN
export HONEYCOMB_API_KEY
export CARGO_REGISTRIES_CATSCII_TOKEN
export GEOLITE2_COUNTRY_DB
```

Before we approve it, let\'s just add one thing to the beginning of our
`.envrc` file:

Shell session

``` {data-lang="shell"}
#!/bin/bash

if ! has nix_direnv_version || ! nix_direnv_version 2.2.1; then
  source_url "https://raw.githubusercontent.com/nix-community/nix-direnv/2.2.1/direnvrc" "sha256-zelF0vLbEl5uaqrfIzbgNzJWGmLzCmYAkInj/LNxvKs="
fi

nix_direnv_watch_file rust-toolchain.toml
use flake
```

This sets up [nix-direnv](https://github.com/nix-community/nix-direnv), which
may have a newer version by the time you read this.

You also probably want to add `/.direnv` to your `.gitignore`, because the next
steps will create this directory and put some symlinks in there.

Now we can run `direnv allow` and let it do its thing:

Shell session

``` {data-lang="shell"}
$ direnv allow
direnv: loading ~/catscii/.envrc
direnv: loading https://raw.githubusercontent.com/nix-community/nix-direnv/2.2.1/direnvrc (sha256-zelF0vLbEl5uaqrfIzbgNzJWGmLzCmYAkInj/LNxvKs=)
direnv: using flake
warning: Git tree '/home/amos/catscii' is dirty
evaluating derivation 'git+file:///home/amos/catscii#devShells.x86_64-linux.default'direnv: ([/nix/store/jyil3ym64dc3hrsqncn9k4naa1mm3w4z-direnv-2.32.2/bin/direnv export bash]) is taking a while to execute. Use CTRL-C to give up.
warning: Git tree '/home/amos/catscii' is dirty
direnv: nix-direnv: renewed cache
direnv: export +AR +AR_FOR_TARGET +AS +AS_FOR_TARGET +CARGO_REGISTRIES_CATSCII_TOKEN +CC +CC_FOR_TARGET +CONFIG_SHELL +CXX +CXX_FOR_TARGET +GEOLITE2_COUNTRY_DB +HONEYCOMB_API_KEY +HOST_PATH +IN_NIX_SHELL +LD +LD_FOR_TARGET +NIX_BINTOOLS +NIX_BINTOOLS_FOR_TARGET +NIX_BINTOOLS_WRAPPER_TARGET_HOST_x86_64_unknown_linux_gnu +NIX_BINTOOLS_WRAPPER_TARGET_TARGET_x86_64_unknown_linux_gnu +NIX_BUILD_CORES +NIX_CC +NIX_CC_FOR_TARGET +NIX_CC_WRAPPER_TARGET_HOST_x86_64_unknown_linux_gnu +NIX_CC_WRAPPER_TARGET_TARGET_x86_64_unknown_linux_gnu +NIX_CFLAGS_COMPILE +NIX_ENFORCE_NO_NATIVE +NIX_HARDENING_ENABLE +NIX_LDFLAGS +NIX_LDFLAGS_FOR_TARGET +NIX_STORE +NM +NM_FOR_TARGET +OBJCOPY +OBJCOPY_FOR_TARGET +OBJDUMP +OBJDUMP_FOR_TARGET +RANLIB +RANLIB_FOR_TARGET +READELF +READELF_FOR_TARGET +SENTRY_DSN +SIZE +SIZE_FOR_TARGET +SOURCE_DATE_EPOCH +STRINGS +STRINGS_FOR_TARGET +STRIP +STRIP_FOR_TARGET +__structuredAttrs +buildInputs +buildPhase +builder +cmakeFlags +configureFlags +depsBuildBuild +depsBuildBuildPropagated +depsBuildTarget +depsBuildTargetPropagated +depsHostHost +depsHostHostPropagated +depsTargetTarget +depsTargetTargetPropagated +doCheck +doInstallCheck +dontAddDisableDepTrack +mesonFlags +name +nativeBuildInputs +out +outputs +patches +phases +propagatedBuildInputs +propagatedNativeBuildInputs +shell +shellHook +stdenv +strictDeps +system ~PATH ~XDG_DATA_DIRS
```

How interesting! We can see everything it exports here: we recognize some of the
things from our `.envrc` file, like `GEOLITE2_COUNTRY` and `HONEYCOMB_API_KEY`, but
also some definitely-nix-related things like `NIX_BINTOOLS`, `AR`, `NM`, `STRIP`,
which are all part of the toolchain pulled in by Rust:

Shell session

``` {data-lang="shell"}
$ which $NM
/nix/store/l0fvy72hpfdpjjs3dk4112f57x7r3dlm-gcc-wrapper-12.2.0/bin/nm
```

[](#setting-up-direnv-in-vscode){#setting-up-direnv-in-vscode}

## Setting up direnv in VSCode

That\'s not quite enough though - we do want to be able to develop catscii in VS
Code, with rust-analyzer, which invokes cargo check, so it must have access to
a Rust toolchain, which we\'re getting from nix.

So how do we get VS Code extensions to find our nix-installed Rust toolchain?
Well, there\'s a
[direnv](https://marketplace.visualstudio.com/items?itemName=mkhl.direnv)
extension! That works with the SSH Remote extension we\'re already using. And
that we\'ve also talked about in Part 4.

After installing it, it\'ll tell us the environment was updated, and ask if
we want to restart extensions, which we do (so we can click \"Restart\").

[](#accessing-our-private-registry){#accessing-our-private-registry}

## Accessing our private registry

If you now try to run `cargo check`, you may run into a cargo error:

Shell session

``` {data-lang="shell"}
$ cargo check
(cut)
Caused by:
  failed to load source for dependency `opentelemetry-honeycomb`

Caused by:
  Unable to update registry `catscii`

Caused by:
  failed to fetch `ssh://git@ssh.shipyard.rs/catscii/crate-index.git`
```

And here, you have two options: either you *have* backed up your SSH key way
back in part 7, or you haven\'t, and then you can make a new SSH key and add it
to Shipyard.

[](#adding-missing-dependencies){#adding-missing-dependencies}

## Adding missing dependencies

After that, `cargo check` gets a lot further, because it manages to clone/update
the private repository.

But it wants `libssl-dev`! Very well then, let\'s add it to our `flake.nix`:

Remember, you can use the [NixOS package search](https://search.nixos.org/) site
to find what you\'re looking for. It\'s pretty good.

nix

``` {data-lang="nix"}
# in flake.nix
{
  inputs = {
    # (cut)
  };
  outputs = { self, nixpkgs, flake-utils, rust-overlay }:
    flake-utils.lib.eachDefaultSystem
      (system:
        let
          overlays = [ (import rust-overlay) ];
          pkgs = import nixpkgs {
            inherit system overlays;
          };
          rustToolchain = pkgs.pkgsBuildHost.rust-bin.fromRustupToolchainFile ./rust-toolchain.toml;
          # new! 👇
          nativeBuildInputs = with pkgs; [ rustToolchain pkg-config ];
          # also new! 👇
          buildInputs = with pkgs; [ openssl ];
        in
        with pkgs;
        {
          devShells.default = mkShell {
            # 👇 and now we can just inherit them
            inherit buildInputs nativeBuildInputs;
          };
        }
      );
}
```

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

Say amos, what\'s the difference between `buildInputs` and `nativeBuildInputs`?

Well, for dev shells, there\'s none. But `buildInputs` are things you\'ll also
need at run-time, whereas `nativeBuildInputs` are things you only need at
compile-time.

It\'s easier to remember if you think of cross-compiling: if you\'re building from x86_64 for aarch64, you might be using a \"native\" GCC for x86_64, that generates
aarch64 binaries, hence: native build inputs.

After saving `flake.nix`, simply hitting Enter will re-evaluate our `flake.nix`:

Shell session

``` {data-lang="shell"}
$ # (hitting enter)
direnv: loading ~/catscii/.envrc
direnv: loading https://raw.githubusercontent.com/nix-community/nix-direnv/2.2.1/direnvrc (sha256-zelF0vLbEl5uaqrfIzbgNzJWGmLzCmYAkInj/LNxvKs=)
direnv: using flake
warning: Git tree '/home/amos/catscii' is dirty
warning: Git tree '/home/amos/catscii' is dirty
direnv: nix-direnv: renewed cache
direnv: export +AR +AR_FOR_TARGET +AS +AS_FOR_TARGET +CARGO_REGISTRIES_CATSCII_TOKEN +CC +CC_FOR_TARGET +CONFIG_SHELL +CXX +CXX_FOR_TARGET +GEOLITE2_COUNTRY_DB +HONEYCOMB_API_KEY +HOST_PATH +IN_NIX_SHELL +LD +LD_FOR_TARGET +NIX_BINTOOLS +NIX_BINTOOLS_FOR_TARGET +NIX_BINTOOLS_WRAPPER_TARGET_HOST_x86_64_unknown_linux_gnu +NIX_BINTOOLS_WRAPPER_TARGET_TARGET_x86_64_unknown_linux_gnu +NIX_BUILD_CORES +NIX_CC +NIX_CC_FOR_TARGET +NIX_CC_WRAPPER_TARGET_HOST_x86_64_unknown_linux_gnu +NIX_CC_WRAPPER_TARGET_TARGET_x86_64_unknown_linux_gnu +NIX_CFLAGS_COMPILE +NIX_CFLAGS_COMPILE_FOR_TARGET +NIX_ENFORCE_NO_NATIVE +NIX_HARDENING_ENABLE +NIX_LDFLAGS +NIX_LDFLAGS_FOR_TARGET +NIX_PKG_CONFIG_WRAPPER_TARGET_HOST_x86_64_unknown_linux_gnu +NIX_STORE +NM +NM_FOR_TARGET +OBJCOPY +OBJCOPY_FOR_TARGET +OBJDUMP +OBJDUMP_FOR_TARGET +PKG_CONFIG +PKG_CONFIG_PATH +RANLIB +RANLIB_FOR_TARGET +READELF +READELF_FOR_TARGET +SENTRY_DSN +SIZE +SIZE_FOR_TARGET +SOURCE_DATE_EPOCH +STRINGS +STRINGS_FOR_TARGET +STRIP +STRIP_FOR_TARGET +__structuredAttrs +buildInputs +buildPhase +builder +cmakeFlags +configureFlags +depsBuildBuild +depsBuildBuildPropagated +depsBuildTarget +depsBuildTargetPropagated +depsHostHost +depsHostHostPropagated +depsTargetTarget +depsTargetTargetPropagated +doCheck +doInstallCheck +dontAddDisableDepTrack +mesonFlags +name +nativeBuildInputs +out +outputs +patches +phases +propagatedBuildInputs +propagatedNativeBuildInputs +shell +shellHook +stdenv +strictDeps +system ~PATH ~XDG_DATA_DIRS
```

\...and we should have `pkg-config` in `$PATH`, which should be able to find openssl:

Shell session

``` {data-lang="shell"}
$ which pkg-config
/nix/store/lrb01sby9jg4hfqhr32s28igg77bhcwr-pkg-config-wrapper-0.29.2/bin/pkg-config

$ pkg-config --modversion openssl
3.0.8
```

And now, it\'s a game of cat and mouse! Let\'s keep going:

Shell session

``` {data-lang="shell"}
$ cargo check
(cut)
    Finished dev [unoptimized + debuginfo] target(s) in 15.41s
```

Huh. I could\'ve sworn\...

![Bear](https://cdn.fasterthanli.me/content/img/characters/classic-bear-glasses~21fda37a9e307cc1.svg){height="42" width="42"}

Yeah, where\'s sqlite?

Mhhh. Mhhh!

Shell session

``` {data-lang="shell"}
$ cargo build
(cut)
  = note: /nix/store/qmmd097h4rwh2pwgz9l9i0byxb0x8q8l-binutils-2.40/bin/ld: cannot find -lsqlite3: No such file or directory
          collect2: error: ld returned 1 exit status

error: could not compile `catscii` due to previous error
```

Ahh! I thought so.

I would\'ve expected it to fail earlier with a nicer message, because, after all,
`pkg-config` cannot find it:

Shell session

``` {data-lang="shell"}
$ pkg-config --modversion sqlite3
Package sqlite3 was not found in the pkg-config search path.
Perhaps you should add the directory containing `sqlite3.pc'
to the PKG_CONFIG_PATH environment variable
No package 'sqlite3' found
```

But it seems the `libsqlite3-sys` maintainers [chose to spray and
pray](https://github.com/rusqlite/rusqlite/blob/9880cdef12c1b624e351ed077dd71a2f77978f77/libsqlite3-sys/build.rs#L436-L443)
in that scenario. Ah well.

Anyway, a [quick
search](https://search.nixos.org/packages?channel=22.11&from=0&size=50&sort=relevance&type=packages&query=sqlite)
reveals that in nixpkgs, sqlite 3 is just named \"sqlite\" (I have yet to encounter
a project that uses sqlite 1 or 2, after all), so we can add it to our `buildInputs`
in `flake.nix`:

nix

``` {data-lang="nix"}
          buildInputs = with pkgs; [ openssl sqlite ];
```

Press Enter in our shell to re-evaluate the flake, and then `cargo build`
succeeds!

Shell session

``` {data-lang="shell"}
$ cargo build
(cut)
   Compiling catscii v0.1.0 (/home/amos/catscii)
    Finished dev [unoptimized + debuginfo] target(s) in 18.74s
```

It is, however, missing the MaxMindDB database:

Shell session

``` {data-lang="shell"}
$ ./target/debug/catscii
{"timestamp":"2023-02-19T18:11:30.946044Z","level":"INFO","fields":{"message":"Creating honey client","log.target":"libhoney::client","log.module_path":"libhoney::client","log.file":"/home/amos/.cargo/git/checkouts/libhoney-rust-3b5a30a6076b74c0/9871051/src/client.rs","log.line":78},"target":"libhoney::client"}
{"timestamp":"2023-02-19T18:11:30.946265Z","level":"INFO","fields":{"message":"transmission starting","log.target":"libhoney::transmission","log.module_path":"libhoney::transmission","log.file":"/home/amos/.cargo/git/checkouts/libhoney-rust-3b5a30a6076b74c0/9871051/src/transmission.rs","log.line":124},"target":"libhoney::transmission"}
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: MaxMindDb(IoError("No such file or directory (os error 2)"))', src/main.rs:59:75
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

\...so we need to copy it over again. I used `scp`:

Shell session

``` {data-lang="shell"}
# from the guest
$ cd ~/catscii/
$ mkdir db

# from the host
$ scp GeoLite2-Country.mmdb eggman:~/catscii/db/
GeoLite2-Country.mmdb                                         100% 5543KB  48.7MB/s   00:00
```

And with that, it runs!

And we\'re *almost done* with this adventure. All we have to do now is rip out
that `Dockerfile`. It\'s going to be glorious. I can smell it from here.

[Comment on /r/fasterthanlime](https://fasterthanli.me/api/comments?url=https%3A%2F%2Ffasterthanli.me%2Fseries%2Fbuilding-a-rust-service-with-nix%2Fpart-10&title=Making+a+dev+shell+with+nix+flakes)

(JavaScript is required to see this. Or maybe my stuff broke)

Here\'s another article just for you:

[Understanding Rust futures by going way too deep](https://fasterthanli.me/articles/understanding-rust-futures-by-going-way-too-deep)

So! Rust futures! Easy peasy lemon squeezy. Until it\'s not. So let\'s do the easy
thing, and then instead of waiting for the hard thing to *sneak up on us*, we\'ll
go for it intentionally.

That\'s all-around solid life advice.

Choo choo here comes the easy part 🚂💨

We make a new project:

Shell session

``` {data-lang="shell"}
$ cargo new waytoodeep
     Created binary (application) `waytoodeep` package
```
