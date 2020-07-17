---
layout: post
title: "Nix - A Bumpy Takeoff"
date: 2020-07-16
categories: nix, haskell
---

Today I tried playing around with [haskell](https://www.haskell.org/) to learn it a little bit better. It was not a smooth start. It seems most projects these days in the haskell ecosystem are moving towards using [nix](https://nixos.org/) as their package installer of choice.

I can see the motivations for this. A package manager that uses a purely functional deployment model fits nicely in line with the Haskell philosophy. It also makes some bold promises such as solving the problem of dependency hell, all the while producing packages that are more reliable, reproducable, and portable.

This all seems great.  But like most things in life, it seems that it doesn't come without its own set of problems.  One of the main ones being that for for users of MacOS Catatalina. 

With the arrival of Catalina, macOS now runs on a [read only system volume](https://support.apple.com/en-ca/HT210650) (Macintosh HD) that gets mounted on `/` .  A user writable second volume (Macintosh HD - Data) now also gets created and mounted at `/System/Volumes/Data`.  It then uses some magic combined with a new concept (brought in along with the arrival of Catalina) called *firmlinks* to combine these two volumes together to create what you see as your file system tree. You can read more about that (here)[https://derflounder.wordpress.com/2020/01/18/creating-root-level-directories-and-symbolic-links-on-macos-catalina/#:~:text=synthetic.conf%20is%20intended%20to,(8)%20during%20early%20sys%2D].

Why is this a problem?  Well, because Nix's architecture and support structures are centered around dropping packges into `/nix` directory that exists at that root level.  A root level that is now read only.  In fact, if you want to uninstall Nix (at least in the case of [single user installation](https://nixos.org/nix/manual/#sect-single-user-installation)), all you need to do is run `rm -rf /nix`.  Pretty nice that everything well compartmentalized like that.  Unfortunately, it just so happens that the location they choose to put everything is a bit complicated in this particular case.

Trying to install nix using the recommended command `sh <(curl -L https://nixos.org/nix/install) --darwin-use-unencrypted-nix-store-volume` fails with error:

    error: refusing to create Nix store volume because the boot volume is
           FileVault encrypted, but encryption-at-rest is not available.
           Manually create a volume for the store and re-run this script.
           See https://nixos.org/nix/manual/#sect-macos-installation

Naturally, Nix offers [a variety of solutions](https://nixos.org/nix/manual/#sect-macos-installation) to this, however they all seem to have their own sets of drawbacks (especially if you don't have a [T2 chip](https://support.apple.com/en-us/HT208862) installed in your machine).  

Initially, I found [this link](https://github.com/digitallyinduced/ihp/issues/93#issuecomment-639611313) (the third solution above) suggesting creating a directory and adding it to the `/etc/synthetic.conf` file.  Now what is the `/etc/synthetic.conf` file, you might be asking?  It's a new configuration file introduced along with Catalina.  What it does is basically allows you to tell macOS to create either directories on the root level that you can mount to or symlinks to other parts of the file system.  (You can see more information about this here.)[https://derflounder.wordpress.com/2020/01/18/creating-root-level-directories-and-symbolic-links-on-macos-catalina]  The solution in the aforementioned link is going the symlink route.

This seemed to work and allowed me to successfully install nix.  However, upon running certain commands, I was running into this error:

    checking XCode version... ./configure: line 5201: xcodebuild: command not found not found (too old?)
    checking for gcc... gcc
    checking whether the C compiler works... no
    configure: error: in `/System/Volumes/Data/opt/nix/store/a88vjfyx4y3grkkvcil3bmkxnmglib9r-configured-ghcjs-src/ghc': configure: error: C compiler cannot create executables See`config.log' for more details

After running find with -L argument to follow symbolic links, I found this 'config.log' file:

    find -L /nix "config.log"
    /nix/store/a88vjfyx4y3grkkvcil3bmkxnmglib9r-configured-ghcjs-src/ghc/config.log

But it didn't provide very helpful either information unfortunately.  Upon reviewing the [above posted link to the nix manual]((https://nixos.org/nix/manual/#sect-macos-installation), I discovered that his option is actually not recommended for reasons that I was seeing.  Builds were likely to fail.  

And while the instructions provided other solutions, it didn't go very into detail about how to implement them.  I decided to go with the second solution and luckily I found [this post](https://www.gitmemory.com/issue/NixOS/nix/2925/535680573) which more or less walked through the process of manually adding a disk volume and preparing it for a nix installation.  It was written in ruby, but most of the instructions were just simple wrappers around shell commands.

First step was to add a disk volume.  Here is the command for doing that:

    diskutil apfs addVolume disk1 APFSX Nix -mountpoint /nix

What is going on here exactly?  Let's break it down:

 - apfs - this is the verb for the diskutil command (i.e. do something involving the APFS - the Apple File system) 
 - addVolume - this is the sub-verb for the apfs command - here we are adding a volume of type APFS
 - disk1 - this is the container where we are adding this volume
 - filesystem - APFSX in this case
 - Nix - Name of the volume being created
 - mountpoint - where we are mounting it (in this case at the root location `/nix`)

 You might be wonder what a container is.  This is a concept of APFS that basically serves a digital space for housing one or more volumes.  The volumes inside of a container can share the space of that container.  Containers have a fixed size but volumes can expand as necessary (and can even expand outside of the container appearantly).  More information on that [here](https://www.lifewire.com/volume-vs-partition-2260237).

You can find more info on the `addVolume` command [here](https://www.dssw.co.uk/reference/diskutil.html#verbs:~:text=unlocked.-,addVolume,Ownership%20of%20the%20affected%20disks%20is%20required.).  It is also avaiable on the man page for `diskutil`.

You can check on the information of that created volume like so:

    diskutil info /nix

Some instructions seemed to be saying you need to run `diskutil enableOwnership /nix` but in my case it seemed to be enabled already (`Owners` entry being set to `Enabled` in the above `info` command).

Next it was necessary to change ownership of the mounted volume:

    sudo chown -R <your user> /nix/

And finally, you optionally enable filevault (encryption) for this volume you just created:

    diskutil apfs enableFileVault /nix -user disk -stdinpassphrase

This will require a password and unfortunately it cannot just use your filevault password for you boot volume.  That appears to be the main crux of this solution (and that this password requirement can complicate services that start on boot if it is first required to enter your password).  The nix community seems to be hard at work on this and we'll see if they manage to solve it some time in the near future.

And that seemed to do it nix now installs correctly as demonstrated by the following output:

    $ sh <(curl -L https://nixos.org/nix/install) --darwin-use-unencrypted-nix-store-volume
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
      0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
    100  2490  100  2490    0     0   2615      0 --:--:-- --:--:-- --:--:--  2615
    downloading Nix 2.3.7 binary tarball for x86_64-darwin from 'https://releases.nixos.org/nix/nix-2.3.7/nix-2.3.7-x86_64-darwin.tar.xz' to '/var/folders/b_/4vbpfsbn0yl0tjzp_96nd1y80000gn/T/nix-binary-tarball-unpack.XXXXXXXXXX.8WRBd5B9'...
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100 26.4M  100 26.4M    0     0  5890k      0  0:00:04  0:00:04 --:--:-- 6079k
    Note: a multi-user installation is possible. See https://nixos.org/nix/manual/#sect-multi-user-installation
    Creating volume and mountpoint /nix.
    
         ------------------------------------------------------------------ 
        | This installer will create a volume for the nix store and        |
        | configure it to mount at /nix.  Follow these steps to uninstall. |
         ------------------------------------------------------------------ 
    
      1. Remove the entry from fstab using 'sudo vifs'
      2. Destroy the data volume using 'diskutil apfs deleteVolume'
      3. Remove the 'nix' line from /etc/synthetic.conf or the file
    
    Using existing 'Nix' volume
    Configuring /etc/fstab...
    Password:
    123
    155
    performing a single-user installation of Nix...
    copying Nix to /nix/store.............................................
    installing 'nix-2.3.7'
    building '/nix/store/hqqr5jaxy2kaxij5jzz3kd8gd63qfdkm-user-environment.drv'...
    created 7 symlinks in user environment
    installing 'nss-cacert-3.49.2'
    building '/nix/store/055gdwqrwg839skhjcp65r2inyzdxs1p-user-environment.drv'...
    created 9 symlinks in user environment
    unpacking channels...
    created 1 symlinks in user environment
    modifying /Users/alex/.bash_profile...
    
    Installation finished!  To ensure that the necessary environment
    variables are set, either log in again, or type
    
      . /Users/alex/.nix-profile/etc/profile.d/nix.sh
    
    in your shell.

And with that, all of nix commands seemed to be working correctly.  The situation at the moment is not great for certain mac users but hopefully a better solution comes around shortly.  Nix has a ton of potential and seems to have a lot of advantages over other package managers out there.