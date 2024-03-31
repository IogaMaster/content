# Introduction
NixOS is an amazing option for server deployments, however deploying many servers at a time can be quite difficult. If you have ever set one up yourself you may know how tedious it can be to plug in a keyboard and monitor and manually partition the drives and install.

This video will show you how to use kexec, nixos-anywhere, disko and deploy-rs to make installing and managing NixOS servers simple.
Let's go.
# What is kexec
kexec is a feature of the linux kernel that allows a new kernel to be loaded into memory and then booted into. It acts a software reboot into a new linux based operating system.
This means we can boot into a minimal install system, without using a flash drive.

That is how we can get into the NixOS installer environment using only the network.

nix-community has a kexec image for x86_64 systems, in the future you will be able to generate a custom kexec image using nixos-generators.
# nixos-anywhere
nixos-anywhere is a tool that will handle the install of our new nixos system by automatically formatting and partitioning drives, installing nixos, and copying additional files to the machine; such as sops related keys.

The partitions are created using disko, a tool that makes drives declarative.
Vimjoyer made a video about impermanence, in which he talks about using disko. I have linked it in the description.
# Deploy-rs
deploy-rs is a simple tool developed by numtide that allows automatic rollbacks and checks before deployment of servers. If you change an ssh config that will lock you out of your server, deploy-rs will not switch to the latest config, and warn you of your mistake.
# Installing
Let's go through an installation.
I have prepared a disko config for this system, and I have root ssh access.

We need to grab the kexec image using curl, and extract it to `/root`
Once I have done that, I use the run script to reboot into the image.

My script waits for a system with the hostname `nixos` to come online.
Then we copy the hardware config using this command:
`nixos-generate-config --show-hardware-config --root /mnt`
We pipe the output into a file.

Once that is done we are ready to install the system.

This command handles the rest:
`nix run github:nix-community/nixos-anywhere -- --flake .#<hostname> root@nixos`

Once that is done, nixos-anywhere will use the disko config to partition the drives and create filesystems.
Then it will build the system closure, and copy all of the store paths to the remote system.
Then it will copy additional files specified to the remote machine.

Then it reboots, and you are done! A freshly installed nixos system.

# Deploying
Deploy-rs requires some additional nix code to set up, you can copy the mkDeploy function from my dotfiles, linked below.

By default deploy-rs will deploy all systems it can reach.
If you specify a nixos configuration from a flake it will only build and deploy that one.
You can skip checks if you would like using the `--skip-checks` flag.

# Additional Tips
If you wish to run an automated test to see if your disko config can install use this command.
`nix build .#nixosConfigurations.<HOSTNAME>.config.system.build.installTest -L`

It will install a system in a vm and verify bootability,  `-L` will show the logs.
# Outro
Thanks to my supporters,
Hauskens, for joining the Voyager tier.
Kinzoku, for continued membership to the Voyager tier.
aksh1618 for continued membership to the Starfarer tier.

If you like what I do and would like to support me please check out my ko-fi page, link in the description.
If you have any questions, feel free to ask in the comments or join our Discord community.
Thanks for watching!
