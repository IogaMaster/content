# Introduction
NixOS is an amazing option for server deployments, however deploying many servers at a time can be quite difficult. If you have ever set one up yourself you may know how tedious it can be to plug in a keyboard and monitor and manually partition the drives and install.

This video will show you how to use kexec, nixos-anywhere, disko and deploy-rs to make installing and managing NixOS servers simple.
Let's go.

# What is kexec
kexec is a feature of the linux kernel that allows a new kernel to be loaded into memory and then booted into. It acts a software reboot into a new linux based operating system.
This means we can boot into the minimal install iso, without using a flash drive.

That is how we can get into the NixOS installer environment, let's take a look at automating the install.
# nixos-anywhere
nixos-anywhere is a tool that will handle the install of our new nixos system by, Automatically formatting and partitioning drives, installing nixos, and copying additional files to the machine; such as sops related keys.

Let's look at writing a declarative disk layout.
# disko
disko is tool that nixos-anywhere uses to set up the drives for the install.
We need to write the config declarativly in nix, 
Most likely you will want a bios/efi compatible disk layout.
```nix
# Example to create a bios compatible gpt partition
{lib, ...}: {
  disko.devices = {
    disk.disk1 = {
      device = lib.mkDefault "/dev/sda";
      type = "disk";
      content = {
        type = "gpt";
        partitions = {
          boot = {
            name = "boot";
            size = "1M";
            type = "EF02";
          };
          esp = {
            name = "ESP";
            size = "500M";
            type = "EF00";
            content = {
              type = "filesystem";
              format = "vfat";
              mountpoint = "/boot";
            };
          };
          root = {
            name = "root";
            size = "100%";
            content = {
              type = "lvm_pv";
              vg = "pool";
            };
          };
        };
      };
    };
    lvm_vg = {
      pool = {
        type = "lvm_vg";
        lvs = {
          root = {
            size = "100%FREE";
            content = {
              type = "filesystem";
              format = "ext4";
              mountpoint = "/";
              mountOptions = [
                "defaults"
              ];
            };
          };
        };
      };
    };
  };
}
```

The config is verbose by fairly simple to understand, just lay out the disks how you would with gparted.

These tools cover the install of servers, let's take a look at managing them with deploy-rs.
# Deploy-rs
deploy-rs is a simple tool developed by numtide that allows automatic rollbacks and checks before deployment of servers. If you change an ssh config that will lock you out of your server, deploy-rs will not switch to the latest config, and warn you of your mistake.