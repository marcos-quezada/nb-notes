# Effortless dev environments with Nix and direnv (determinate.systems)

<https://determinate.systems/posts/nix-direnv/>

## Tags

#nix

## Content
# Effortless dev environments with Nix and direnv

Luc Perkins

------------------------------------------------------------------------

Like many of you, I work on a lot of different projects. Even when a
project is less serious—hey, I should check out this new JS framework!—I
strive to reduce the friction involved with setting up the project’s dev
environment to the absolute bare minimum possible. In this post, I’d
like to tell you how I do that using two tools that have become
indispensable to my daily workflows:
<a href="https://nixos.org/" rel="noopener noreferrer">Nix</a> and
<a href="https://direnv.net/" rel="noopener noreferrer">direnv</a>.

## [Prerequisites](#prerequisites)

If you’d like to follow along with this tutorial:

-   <a href="https://nixos.wiki/wiki/Flakes#Enable_flakes"
    rel="noopener noreferrer">Enable flakes on NixOS</a> or install [Nix
    with Flakes enabled on any other Linux and
    macOS](https://determinate.systems/posts/determinate-nix-installer).
-   Install
    <a href="https://direnv.net/" rel="noopener noreferrer">direnv</a>.
    I personally use <a
    
href="https://github.com/the-nix-way/nome/blob/main/home/programs.nix#L16-L21"
    rel="noopener noreferrer">Home Manager</a> to install this but of
    course feel free to use whichever method works for you. Optionally,
    you can also install
    <a href="https://github.com/nix-community/nix-direnv"
    rel="noopener noreferrer">nix-direnv</a>, which provides a faster
    implementation of the Nix-related functionality in direnv.

## [Nix shell environments](#nix-shell-environments)

Before I go deeper into the fearsome combo of Nix and direnv, I’d like
to say a bit about the <a
href="https://nix.dev/tutorials/declarative-and-reproducible-developer-environments"
rel="noopener noreferrer">Nix shell</a> for those who aren’t yet
familiar. <a href="https://nixos.org/" rel="noopener noreferrer">Nix</a>
enables you to create fully declarative and reproducible development
environments that are defined by:

-   Which Nix packages you want installed in the environment
-   Which shell commands you want to run whenever the environment is
    initialized

The standard function for creating shell environments is
<a href="https://nixos.org/manual/nixpkgs/stable/#sec-pkgs-mkShell"
rel="noopener noreferrer"><code>mkShell</code></a> in Nixpkgs, but other
functions are available in the ecosystem. Here’s a simple example:

<figure>
<pre data-language="nix"><code>1{2  devShells.default = pkgs.mkShell {3    
buildInputs = with pkgs; [go_1_19 terraform];4  };5}</code></pre>
</figure>

When you enter this shell by running `nix develop`, Go version 1.19 and
Terraform will both be available in the current directory (but not
globally). With definitions like this, you can include every package you
may need in the environment, which includes executables (like Go and
Terraform) as well as things like language servers and editor
configurations.

## [direnv and Nix environments](#direnv-and-nix-environments)

<a href="https://direnv.net/" rel="noopener noreferrer">direnv</a> is a
tool that, whenever you navigate to a directory with a `.envrc` file in
it, enacts the directives in that file to produce a directory-specific
environment *every time* you navigate there. direnv has a lot of great
features worth checking out, such as <a
href="https://direnv.net/man/direnv-stdlib.1.html#codedotenv-ltdotenvpathgtcode"
rel="noopener noreferrer">loading <code>.env</code> files</a> and
<a href="https://direnv.net/man/direnv-stdlib.1.html#codeuse-rbenvcode"
rel="noopener noreferrer">managing local Ruby versions</a>, but today I
want to focus on just one: the <a
href="https://github.com/direnv/direnv/blob/master/stdlib.sh#L1256-L1271"
rel="noopener noreferrer"><code>use flake</code></a> directive. If you
add this to your `.envrc` file, direnv activates the <a
href="https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-develop.html#flake-output-attributes"
rel="noopener noreferrer">Nix shell</a> defined in the local
`flake.nix`:

Practically, this means that all you need to do to enter your
Nix-defined development environment is to `cd` into the directory
(unless you run `direnv revoke`, which causes direnv to ignore the
`.envrc` until it’s reactivated using `direnv allow`). This is an
elegant solution to a problem that we’ve all encountered where we
navigate to a directory, run a setup command, and end up confused
because the desired executable is missing or some other aspect of the
environment isn’t what we want.

## [direnv with remote Nix environments](#direnv-with-remote-nix-environments)

The `use flake` directive works great when the `.` points to a local
flake defined by a `flake.nix` file, automatically loading the dev shell
defined under the `devShells.default` output. But there’s actually an
even faster way to use direnv for dev environments because `use flake`
can point to *any* dev shell defined in any flake. Take this <a
href="https://github.com/the-nix-way/dev-templates/blob/main/node/flake.nix#L27-L33"
rel="noopener noreferrer">Node.js dev environment</a> as an example:

<figure>
<pre data-language="nix"><code>1{2  # Inputs omitted for brevity34  outputs = { 
... }: {5    devShells.default = pkgs.mkShell {6      packages = with pkgs; 
[node2nix nodejs pnpm yarn];78      shellHook = &#39;&#39;9        echo 
&quot;node `${pkgs.nodejs}/bin/node --version`&quot;10      &#39;&#39;;11    
};12  };13}</code></pre>
</figure>

The URL for this flake is `github:the-nix-way/dev-templates?dir=node`,
so if we want to use this environment, we can add this to the `.envrc`
file instead:

<figure>
<pre data-language="bash"><code>use flake 
&quot;github:the-nix-way/dev-templates?dir=node&quot;</code></pre>
</figure>

If you run `direnv allow` in the directory, direnv gets to work using
Nix to install the `packages` defined in this remote flake (Node.js,
npm, etc.). That can take a while time the first time you enter the
directory because Nix needs to install the environment’s dependency tree
into the Nix store, but when you `cd` in the future it should be quite
speedy.

But the potential fun doesn’t stop there. I’ve created, for example, my
own little helper script for creating `.envrc` files that I call `dvd`:

<figure>
<pre data-language="bash"><code>echo &quot;use flake 
\&quot;github:the-nix-way/dev-templates?dir=$1\&quot;&quot; &gt;&gt; 
.envrcdirenv allow</code></pre>
</figure>

So if I run .dvd go., for example, my shell immediately starts loading
<a
href="https://github.com/the-nix-way/dev-templates/blob/main/go/flake.nix#L22-L37"
rel="noopener noreferrer">this environment</a>, which includes Go
version 1.19 and some commonly used Go tools (like
<a href="https://pkg.go.dev/golang.org/x/tools/cmd/goimports"
rel="noopener noreferrer">goimports</a>).

No Dockerfile, no Makefile, no local Nix logic, just a single line in a
`.envrc` file can get you a specific, arbitrarily complex dev
environment.

## [Layering environments](#layering-environments)

Because `.envrc` files are scripts, you can add as many `use flake`
statements as you want, which enables you to *layer* Nix environments.
Here’s an example `.envrc`:

<figure>
<pre data-language="bash"><code>use flake 
&quot;github:the-nix-way/dev-templates?dir=elixir&quot;use flake 
&quot;github:the-nix-way/dev-templates?dir=gleam&quot;</code></pre>
</figure>

In this case, both an <a
href="https://github.com/the-nix-way/dev-templates/blob/main/elixir/flake.nix"
rel="noopener noreferrer">Elixir</a> and a <a
href="https://github.com/the-nix-way/dev-templates/blob/main/gleam/flake.nix"
rel="noopener noreferrer">Gleam</a> environment would be applied to the
current directory. This may not *always* be the best idea, as Nix honors
the flake listed last in case of a clash, for example in the case of two
packages named `cargo` that refer to different versions of
<a href="https://doc.rust-lang.org/cargo/"
rel="noopener noreferrer">Cargo</a>. But if you apply due care, you can
reap the benefits of layered environments without being stung by
ambiguity.

## [Drawback](#drawback)

The major pro of using remote flakes for dev environments is that it
enables you to initialize an isolated, pure dev environment about as
quickly as you can hope for in the software world. For weekend projects
and experiments it’s perfect. The downside is that relying on a dev
environment defined in a remote flake doesn’t provide exactly what you
need. Layering environments can add some specificity but for projects
that require a fully custom environment, the approach in this post
breaks down pretty quickly.

But never fear! There’s another great Nix feature that can help here: <a
href="https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake-init.html"
rel="noopener noreferrer">Nix flake templates</a>. With templates, you
can use the `nix flake init` command to initialize a Nix-defined
template in the current directory. Personally, I use a script called
`dvt` to initialize specific templates from my
<a href="https://github.com/the-nix-way/dev-templates"
rel="noopener noreferrer">dev-templates</a> project:

<figure>
<pre data-language="bash"><code>nix flake init -t 
&quot;github:the-nix-way/dev-templates#$1&quot;direnv allow</code></pre>
</figure>

So if I run dvt I get a specific `.envrc`, `flake.nix`, and
`flake.lock`. These files provide a great basis for further
customization. Let’s say that I need to start up a Python project but I
also want to install <a href="https://github.com/watchexec/watchexec"
rel="noopener noreferrer">watchexec</a>. For that I can run
`dvt python`, wait for direnv to load the environment, and then add
`watchexec` to the `packages` in `flake.nix`.

## Find out more about how Determinate Systems is transforming the developer 
experience around Nix

## [Security warning](#security-warning)

One thing that you should always bear in mind when using remote flakes
for dev environments is that you can add *anything* to `packages` and
that `shellHook` allows for arbitrary code execution. If you add
`use flake "github:bitcoin-scam/gonna-mine-btc"` to your `.envrc` you
are going to be in for a rough time (and you could still be in for a
rough time if you use a more innocuous-seeming flake). So there are some
precautions you should always take:

1.  Examine the actual contents of the flake to make sure nothing funny
    is going on.
2.  Less obviously, if you’re using a version-controlled remote flake,
    make sure to “pin” to a specific reference. So instead of
    `use flake "github:the-nix-way/dev-templates?dir=rust"` you can use
    `use flake 
"github:the-nix-way/dev-templates/b2377c684479f6bf7d222a858e1eafc8a2ca3499?dir=rust"`
    to make the env truly declarative and overcome the ambiguity of the
    first `use flake` directive.

## [Broader implications](#broader-implications)

Using techniques like this has helped me to not just get up and going in
dev projects more quickly than ever before—seriously, it almost feels
like cheating—but also to declutter my home environment. I talked about
this a little bit in a [previous
post](https://determinate.systems/posts/nix-home-env), where I laid out
an imperative that I’ve been following recently where I keep my laptop’s
global environment as minimal as possible—just Git, jq, wget, and a few
other executables that I truly need available everywhere on my
system—while making everything else project specific.

As devs, we should spoil ourselves more. We should get used to dropping
into projects willy-nilly and having them Just Work, and the processes
that make them Just Work should fade into the background. The approach I
laid out here charts what I take to be a reasonable course between
convention and configuration. I certainly don’t expect you to make the
same choices, but I do hope that the formidable combination of Nix
flakes and direnv becomes an extra-sharp arrow in your daily dev quiver.
