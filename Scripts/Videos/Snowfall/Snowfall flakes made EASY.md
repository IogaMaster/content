# Introduction
Snowfall is a framework for structuring  Nix flakes.
It handles all the tedious parts, leaving you to craft your configuration.

 This video is aimed to help you understand how it works and how to set up a NixOS configuration using my Snowfall template.
Lets dive in!
# Template
My template comes with some essentials, bios/efi boot options, Nix cli config, audio, networking, NVIDIA, and a module to extend battery life on nixos laptops.

It has just enough to get you going.
You will need to add and enable the window manager or desktop environment you want.
If you would like some fleshed out examples of a snowfall configuration, check out jakes config, and my dotfiles.

Lets build the basics on top of the template (although these concepts will apply to any snowfall configuration)
# The basics
Snowfall uses directories to declare modules, systems, library functions, packages, etc.

Let's create a system, you need to create a structure that looks like this:
```txt
systems/
 - x86_64-linux/
    - hostname/
      - default.nix
      - hardware-congfiguration.nix
```

Its pretty simple, the directory names define the architecture and the host name.
Keep in mind that these work with darwin systems as well.
## Modules

Now the real power of snowfall comes from the simplicity of creating modules:
```txt
modules/
 - nixos/
    - foo/bar/baz/
      - default.nix
```

Here you add them to the modules directory, then in nixos or home depending on whether you are creating a home-manager module or not. Most of the time you want the NixOS directory. 
You can create any number of nested directories for the module.

Modules are pretty simple to create (mkBoolOpt is an alias that is in the template).
Options are just globally accessible variables that are interpolated into the config block.
```nix
{
  options,
  config,
  lib,
  ...
}:
with lib;
with lib.custom; let
  cfg = config.module;
in {
  options.module = with types; {
    enable = mkBoolOpt false "Enable module";
  };

  config =
    mkIf cfg.enable {
    };
}
```

The module name must correspond with the directory structure, (modules require it in a way that snowfall doesn't abstract away)
# Packages and overlays
Snowfall will automatically include custom packages under a namespace you create.
They work exactly the same way as a package in nixpkgs.
```txt
packages/
 - sys/
    - default.nix
```

Overlays are also simple.

```txt
overlays/
 - obsidian/
    - default.nix
```

## Library functions
Sometimes you want some utilites for working with nix.
There are some in the template, mkOpt and mkBoolOpt for example.
# Nix Templates 
My template also includes some nix templates to make creating modules, overlays, libraries, etc; much easier.

Just run:
`nix init -t .#<template name>`
and move it into the directory you want
# A note on homes
I do not use homes myself as they are more for non-nixos systems. Although they can be used on nixos.

They work like systems but define home-manager modules instead of nixos modules.
# Additional Help
Some of my discord members use snowfall and would be happy to help, in addition I have started doing one hour mentoring calls where I can help you build or convert your config to snowfall, and with the link in the description you can get 50% off until February 1st.
# Outro
I would like to thank kinzoku for his membership of the voyager tier of Ko-fi, aksh1618 for the starfarer tier.
I would also like to thank Gaetan Lepage for his 3$ donation on Ko-fi

If you like what I do and would like to support me please check out my ko-fi page, link in the description.
If you have any questions, feel free to ask in the comments or join our Discord community.
Thanks for watching!
