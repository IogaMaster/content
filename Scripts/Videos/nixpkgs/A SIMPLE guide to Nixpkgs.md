Nixpkgs is the central software repository for anything Nix related.
It's the largest and most fresh package set in the Linux space by FAR.

The contributing process is fairly straightforward, however there are a few idioms and guidelines to follow.

The video is aimed at giving you an overview of the process, some tips, and some tools to make your contributions higher quality.
# Creating the package
The normal workflow for creating a new package goes as follows.
Fork nixpkgs on github and clone it.

nixpkgs can take a long time to clone, use filter tree to minimize the file size.
`git clone git@github.com:<username>/nixpkgs --filter=tree:0`

If you're using a git ui like lazygit, loading all the commits can be quite slow. To fix this simply run this command:
`git commit-graph write`
This will save all commits without fetching. You will have to run this periodically.
To avoid this run `git maintainence start`. This will automatically write the commit graph hourly.

The next step is to create a branch. You never want to commit changes directly to the master branch, as this can prevent new contributions until the current one is merged into nixpkgs.

Then you create a package, the current preferred location for these packages is in `pkgs/by-name`.

In there you create a package file.
In a sub directory with the first two characters of the package name.
Then another sub directory that is the name of the package.
Then a package.nix file
Here is an example with a package i maintain.
`pkgs/by-name/ni/niri/package.nix`
## Creating the package build
At the top of the package you import all the helper functions and packages needed to build and run your package.

Then create the package build. The most common function for that is `stdenv.mkDerivation`. It can build any package, however different programming languages require different builder functions, Rust for example requires `rustPlatform.buildRustPackage`.

Onto the conventions, you want to use `pname` and `version` over `name`.

Package versions in nixpkgs cannot have `v` in them.
So if you pull a package source from a tag with `v` remove it from the version.

`src` contains the extracted source code of the project. There are a few helper functions in nixpkgs. Most often you will use `fetchFromGitHub`.  Usage is pretty simple, give it the owner of the repo, and the repo name, and the revision, this can be a commit hash or a tag;
and the hash of the source code.
You can leave this blank for now, when you build the package it will tell you what hash to use.

Now let's use those imported packages.
There are a few types of inputs.

`nativeBuildInputs`: These are the packages required to run the build commands. Like `cmake`, `meson`, `ninja` or `pkg-config`.

`buildInputs`: These are the libraries linked into the package during the build. Like `SDL2`, `openssl` or `zlib`.
### Phases
Onto the phases, these are bash snippets that are called by `stdenv.mkDerivation`.
Some important variables used in the snippets are:
`$src` this is the store path of the source code.
`$out` the final store path the final package will be copied to.

There are a few of these phases.

First we have the `configurePhase`: 
This will run the `./configure` script by default.  It's also used to run more complex `cmake` setup.

Then the `buildPhase`: By default this will run a makefile if it exists. This will run the actual compilation.

Then the `installPhase`:  By default it will create the store path under the `$out` variable and call make install. This is where you will copy the resulting output from the build and things like man pages and documentation to `$out`.
## meta
Lastly let's look at the meta block.
This contains all the nixpkgs specific metadata, like the description and homepage. These are just strings.

Then we have the more advanced attributes.

`license`: Nixpkgs uses a centralized list of licenses in `lib/licenses.nix`. This information is crucial for the nixpkgs build server. Periodically, the master branch transitions to the `nixpkgs-unstable` branch, triggering the hydra build server to compile and store every package in the cache at `cache.nixos.org`. To comply with legal requirements, it's essential to designate packages with an unfree license if they are built from precompiled packages or lack a specified license.

`mainProgram`: Just a string containing the filename of the main binary. Not required if you are adding a library.

`platforms`: This tells the build server what platforms to cache. These can include `linux`, `unix` which is MacOS and linux. And `all`. You can also specify a specific architechure.

`maintainers`: This contains a list of maintainers for the package from the `maintainers/maintainer-list.nix` file. It is not required to add a maintainer, however it is good practice. When someone updates your package ofborg will request a review from you.
Let's take a look at adding yourself as a maintainer.
## Becoming a maintainer.
Navigate to the `maintainers/maintainer-list.nix` file.
Then add a block in alphabetical order with your username.

Here is mine.
```nix
iogamaster = {                                                    email = "iogamastercode+nixpkgs@gmail.com";
   name = "IogaMaster";
   github = "iogamaster";
   githubId = 67164465;
};
```

You need to add your email, name (can be your pseudonym), github username, and your github ID. 
You can get your id by running this command:
`curl https://api.github.com/users/<USERNAME> | grep "id":`

Commit this with the message.
`maintainers: add <name>`, name being the title of the block you added.
Once this is added to nixpkgs you won't need to do it again.
# Commiting
Once that is done you can commit your package.
nixpkgs commits follow a very specific format;
`(pkg-name): (from -> to | init at version | refactor | etc)`

Here are some examples,
Adding the package: `niri: init at 0.1.0`
Updating the package `niri: 0.1.0 -> 0.1.2`

If you have a bunch of extra commits you need to squash them.

Then push.
# Creating the PR
Let's create a pull request, just point your change to master and submit the pr.
If your change causes a mass rebuild, like updating gnupg for example, which 5000+ packages depend on, you need to point it to the staging branch instead.

Then you need to check a few boxes, you need to specify what architecture you built your package on, that you verified it's functionality, and that it follows the contributing guidelines.

Make the title of your pr the commit of the changed package.

If you needed to change or add multiple packages in one pr. Just add all the commits in the title and separate them with a comma like so:
`hypridle: init at 0.1.0, hyprlang: 0.3.2 -> 0.4.0`

Github actions will run some basic tests, like checking the formatting, and if you followed the `pkgs/by-name` conventions properly.
Then ofborg will build your changed packages for every platform specified, this can take a while. Especially on macos, it can sometimes take days to finish the evaluation.
## Getting reviewed
When people review your pr you need to remember that harsh changes may be needed, please respect other contributors. Be positive, clear and concise with your responses, and handle  disagreements professionally.

If you would like a review ping me at `@IogaMaster` in a comment, I will take a look at your changes.
# Additional Tips and tools
Some nice tools to keep your nixpkgs contributions clean include,
`statix`: This will show some lints for your packages.
`deadnix`: This will remove dead code.
`nixpkgs-hammering`: This will run your package code and provide lints and fixes that insure you keep to the nixpkgs standards and conventions.
`nixpkgs-fmt`: This will format your package code to follow the nixpkgs standard.
### nix-init
If all of this sounds a little complicated and full of boilerplate you would be right.
The best tool by far is `nix-init`
Give it a link to the source code of your package, and it will create the package file, set up the boilerplate, fetch the source and pin the hashes. It will even detect the license and add it. It can also add you as a maintainer and commit the changes, following the commit convention.

This makes things really simple, however don't rely on this to build a completely working package. In most cases you still need to add the phases and make a proper build.
So knowing how to do this manually is still required.
### nix-update
Give `nix-update` the name of the package, and it will fetch the new source revision and hash, it can also make an update commit.
It doesn't work all the time however.

So if you need to manually update this you can use `nurl`.
### nixpkgs-review
If you give `nixpkgs-review` a pr number, it will build every package affected by the pr.
You need to be in a nixpkgs directory. It can take a ridiculously long time, so it's not a requirement. 
# Outro
Thanks to my supporters,
Hauskens, for continued membership to the Voyager tier.
Kinzoku, for continued membership to the Voyager tier.
aksh1618 for continued membership to the Starfarer tier.

If you like what I do and would like to support me please check out my ko-fi page, link in the description.
If you have any questions, feel free to ask in the comments or join our Discord community.
Thanks for watching!