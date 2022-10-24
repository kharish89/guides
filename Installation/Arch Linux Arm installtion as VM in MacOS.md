#vm #arch_linux_arm #apple #M1 #M2 #KDE_Plasma #development

This guide will cover the step by step installtion and setup for arch linux arm on apple silicon mac as a Virtual Machine using [[Parallels]] or [[UTM]] as a VM tool. We will also cover desktop and some of the dev tools setup. Arch linux is a very customizable and comes as a bare bone image so the user can customize it to their personal taste. Hence we will use the [archboot](https://pkgbuild.com/~tpowa/archboot/web/archboot.html) project to get an ISO image that will be handy to install in a VM setting. ArchBoot also packages an easy to use isntalltion UI for arch linux and provides ISO for various machine architecture.

## Prerequisite
- [Parallels desktop](https://www.parallels.com) for apple silicon mac (or) 
- [UTM](https://docs.getutm.app) for MacOS a open source free virtual machine app
- archboot [aarch64 ISO](https://pkgbuild.com/~tpowa/archboot/iso/aarch64/latest/)
- 16 GB system memory, so we can allocate 8 GB for VM
- 8 core system, so we can allocate 4 cores
- 1 TB SSD / NVMe, so we can allocate 128 GB

> [!Info] 
> It is best to use Parallels desktop on apple silicon mac, at the time of writting UTM has **openGL** rendering as experimental and the linux for arm images I tried, namely
> 
> - Ubuntu 22.04 LTS daily build for arm64 desktop (ubuntu only offers arm64 server and not desktop in stable)
> - Ubuntu 22.10 daily build for arm64 desktop
> - ArchLinux arm
>
> With Parallels the graphics rendering was smooth and buttery

## Steps
1. Create VM with minimum desired specs
2. Boot & install arch linux from archboot installtion image
3. Reboot & create non root user with sudo access
4. Install KDE plasma desktop & system utils
5. Install Development tools

---
### Create VM with desired hardware specs

Select the VM type as Linux and choose the ISO downloaded from archboot images repository. I downloaded the image with just date and not the latest or local one. Which means we need an active internet to make the installtion.

Specs:
- OpenGL acceleration enabled
- 4 core CPU
- 8 GB memory
- 128 GB storage
- Clipboard sharing

---
### Boot & Install arch linux from archboot image

Once the VM is prepared we can now boot into the image by selecting the `Archboot Arch Linux AA64` option on the boot menu.

![[Pasted image 20221023170848.png]]

You should now see the arch boot options presented to you as below

![[Pasted image 20221023171113.png]]

Hit **ENTER** and continue to the install wizard

![[Pasted image 20221023171156.png]]

Once you hit **OK** you should see the 9 step setup options

![[Pasted image 20221023171344.png]]

Steps 0-3 are pretty straight forward and I would skip any screenshots for the to save some image space. Lets jump to step 4 to prepare storeage drive

![[Pasted image 20221023171631.png]]

Select Auto-Prepare for easy setup as its a fresh installtion on a VM

![](../Images/Arch%20linux%20installtion/Pasted%20image%2020221023171729.png)
![[Pasted image 20221023171851.png]]
![[Pasted image 20221023171918.png]]

Mostly accepting the default and just adjusted the swap to be 2 times the size of my RAM as a good rule of thumb. In my case the memory allocated for this VM was 8 GB so I did a swap of 16 GB (`16834` MB).

![[Pasted image 20221023171944.png]]
![[Pasted image 20221023172140.png]]
![[Pasted image 20221023172157.png]]
![[Pasted image 20221023172210.png]]
![[Pasted image 20221023172245.png]]

Now lets install some packages to do the basic configuration

![[Pasted image 20221023172540.png]]

After following several other network related prompts and accepting default values, we see

![[Pasted image 20221023172700.png]]

After the installtion is finished with the above minial package you are prompted with a success message and a prompt to accpet the GPG entropy, which I gave a **YES**.

Next we will move on to the Configure System step to add host name, /etc/host config and root password.

![[Pasted image 20221023172933.png]]

When prompted to use a text editor I chose - nano
1. Nano (easy)
2. VI (If you know what you are doing)

![[Pasted image 20221023173209.png]]

- Set the `/etc/hostname` to the name of the machine you want when accessing the shell after boot 
- Set the `/etc/hosts` to have local host config (this is not mandatory, I just did it)
```
127.0.0.1    localhost
::1          localhost
```
- Set the `Root-Password` to something that is secure as it will be the one used for inital login after isntallation.

Lets move on to Installting the boot loader

![[Pasted image 20221023173644.png]]
![[Pasted image 20221023173703.png]]

I chose `GRUB_UEFI` as my choice of boot loader type, as with some googling I found that to be most upto date.

![[Pasted image 20221023173819.png]]

Now in this step I literally did not change any settings and you will understand why when you hit there!! (I really didn't know what any of the config meant, LOL!!)

![[Pasted image 20221023174015.png]]

Hit **NO** for the prompt to install another bootloader

![[Pasted image 20221023174157.png]]

Then step 8 to exit the installer will directly take you to this succes and reboot message

![[Pasted image 20221023174336.png]]

Type the reboot command to exit the installtion image and boot into the arch linux installtion
```shell
reboot
```


---
### Reboot & create non-root user with sudo access

Once you have done reboot, eject the ISO image used to boot so it doesn't take that as prioity and put you back to the archboot installtion. If you already had your boot preference to HDD, then the new installtion would be propted as below for you to boot.

![[Pasted image 20221023174848.png]]

You are presented with the hostname selected and the login screen, we will use the user `root` and the root password you set in the previous step.

![[Pasted image 20221023175005.png]]

A successful login shoul look something like this below

![[Pasted image 20221023175055.png]]

Now our goal is to setup a new user account with sudo access that we can use to login and install the desktop and development tools. It is not safe to use the root user as installing some software as root user is dangerous.

Lets welcome `pacman` the default package manger for arch linux, it is similar to `brew` for mac and `apt-get` or `snap` for ubuntu. Pacman is a lovely package manager and the community publishes a hell loads of ports for arch linux and specially for arm64/aarch64 architecture which our apple silicon mac is based of. At the time of writting other package libraries (ofcourse except `brew` for macos) did not have many apps ported natively. So I decided to go with arch linux as the popular [Ashai linux project](https://asahilinux.org) was also based on arch linux for arm distro.

> [!info] 
> We don't need sudo command in fornt of all pacman installation since we are a root user at this point

Updating pacman packages with
```shell
pacman -Syu
```

Install `sudo` & `nano`
```shell
pacman -S sudo vi
```

Lets add our new user (dev in my case, plese substitute your username)
```shell
useradd -m -U -G power,wheel,storage dev-user
```

Validate the user by issuing the `groups <user>` command
```shell
groups dev-user
```

Next, lets add a password for our `dev-user`
```shell
passwd dev-user
```

Lets add the user to sudo by uncommeting the sudo file
```shell
visudo
```

![[Pasted image 20221023180541.png]]

We can now switch to our new user `dev-user` by the below command
```shell
su - dev-user
```

> *Note:* Going forward we need to issue sudo command to all pacman commands as we are not root any more


---
### Install KDE plasma desktop & system utils

First, make sure that your Arch Linux install is up-to-date by using the command:

```shell
sudo pacman -Syuu
```

Next, download the plasma-desktop package.

```shell
sudo pacman -S plasma-desktop
```

It may ask you to choose providers for some packages, just press enter to choose the default ones and continue with the installation. It will also install the xorg display server automatically so you don’t need to worry about that.

![[Pasted image 20221023181501.png]]

Once the plasma-desktop finishes installing, we can now proceed to install a display manager. A display manager is a GUI that allows us to login into our desktop environment. We will be using sddm here as it the default Display Manager for KDE Plasma.

To install sddm, type:

```shell
sudo pacman -S sddm 
```

You are now pretty much ready to go. But if you start sddm and log into your plasma session, what you will see is blank desktop with no web browser, no network manager, no audio controls, no file manager, not even a terminal app.

We will need to fix that and install some essential utilities. Here are the ones I have chosen:

**Web Browser**: For web browser we will be using good old firefox here.

**Network Manager**: Kde has a package named plasma-nm that we can install and use to connected to a network (Wifi/Ethernet).

**Audio**: For audio, we will install plasma-pa, which is PulseAudio integrating for Plasma desktop.

**File Manager**: Dolphin is the file manager that we are going to install.

**Terminal**: As for terminal, we will be installing Konsole. It is the default terminal app for KDE.

I have also chosen to install the kdeplasma-addons package. It provides some extra widgets for your status bar such as Caps lock indicator, Microphone indicator, Night color switcher, etc. 

**GTK Styling**: Some apps on KDE look a bit off (in terms of styling) without this. To configure this, after install, go to Settings>Appearance>Application Style> GNOME/GTK Application Style.

**kscreen**: This package will allow you to configure your monitor/s. If you want to change orientation, resolution, scaling, & refresh rate of your monitor then you will need this app. kscreen is also recommended for **setups with multiple monitors**.

**Alacritty:** OpenGL based terminal emulator which is light and fast

**kcalc:** A simple scientific calculator app

**ksysgaurd / plasma-systemmonitor:** A nice system monitor like mac activity monitor or windows task manager

**KDE Info Center:** A extension that provides system information on the settings app in plasma desktop

**Ark archive utility:** An archive utility used to compress and expand multiple formats

Again, these are the ones I have chosen to install as they work great with KDE Plasma, as long as you know what you are doing, you can replace them with other utilities of your liking.

Now to install them, use command:

```shell
sudo pacman -S firefox plasma-nm plasma-pa dolphin konsole kdeplasma-addons kde-gtk-config alacritty kcalc ksysgaurd kinfocenter plasma-systemmonitor kscreen ark
```

Once the installation finishes, use these two commands (case-sensitive) to start and enable network manager.

```shell
sudo systemctl enable NetworkManager
```

```shell
sudo systemctl start NetworkManager
```

**Optional Packages**: 

**powerdevil**: If you are installing kde plasma on a device like a laptop or notebook, you might want to install powerdevil. Powerdevil will show you device’s battery percentage in the system tray and also grant you controls to adjust screen brightness. It also provides settings like lid close action, screen timeout, sleep settings, low-battery action etc.

We can now go ahead and log into the KDE Plasma Desktop. First, lets enable sddm so it gets started on boot.

```shell
sudo systemctl enable sddm
```

Now enter the command below to start sddm

```shell
sudo systemctl start sddm
```

Once you type the above command, you should see a login screen, enter your details there and login.


---
### Install Development tools

Now that we have come this far installing all the necessary tools for the system. Let's see what are some of the development tools we can install and configure.
- **git:** Version control system (ofcourse you knew this!!) <span class='flair mod-pop' style='background-color:#F1C40F;color:#000;'>Team Gold</span>
- **base-devel:** Group of packages need when compiling software from [[AUR]] Archlinux User Repository, where we can find packages like VSCode & Obsidian at the time of writting.
- **unzip:** Archive utility might be a dependency for some scripts we install
- **neovim:** New Vim editro with lots of support for plugins and supports lua scripts along with vimscripts. Customizable to an IDE with preconfigured packages like [AstroVim](https://astronvim.github.io).
- **zsh:** Alternate shell environment that supports autocompelte, themes and plugin support
- **omzsh:** Easy way to manage your zsh shell environment with expanded configurations
- **Jerbrains mono font:** One of the Nerd fonts used by [starship](https://starship.rs) shell prompt and [AstroVim](https://astronvim.github.io)
- **starship:** Cross platform shell prompt with customizable support.
- **nvm:** Node version manger helps to maintain more than one version of nodeJS environment on you system, usefull when you need to switch between project with different nodeJS versions.
- **nodeJS:** Binary for NodeJS runtime.
- **ripgrep:** Advanced terminal based grep utility aimed as a replacement for `grep` util.
- **lazygit:** Terminal based git utility that is not a replacement but closely tries to be `magit` on emacs.
- **ncdu:** Terminal based disk usage utility.
- **htop:** Terminal based resource utilization monitor
- **Astrovim:** A currated collection of neovim plugins and configurations, that makes nvim an IDE for most usecases in an quick setup.
- **Rustup:** Rust language official installer and toolchain manager.
- **gdb:** Debug tool for binary
- **lldb:** Debug tool
- **valgrind:** Memory debug tools
- **VSCode:** Open source IDE based on electron, which is very extensible using official and community plugins.
- **Obsidian:** A nice markdown editor with excellect support for plugins and can become a second brain.