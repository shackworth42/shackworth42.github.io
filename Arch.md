# Arch Linux install

## VM Setup

1. Download the Arch ISO from the Arch website
2. Create a new VM with that ISO
    1. Select 'Linux under guest operating system
    2. Select Other Linux 5.x and later kernel 64-bit
3. Increase the disk size and memory allocation to a reasonable level (at least 2 GB or more of memory)
4. Change the boot mode of your VM to UEFI
    1. Right click on your VM and select settings at the top
    2. Under advanced, change the firmware type to UEFI
5. Start your VM
6. Verify your VM is setup correctly by launching the command; 'ls /sys/firmware/efi/efivars'
    1. This should generate a list of files if you have followed the above steps
7. Confirm internet connectivity with the command: 'ping -c 4 www.linuxconfig.org'
8. Set your system clock to sync with your host machine with the command: 'timedatectl set-ntp true'



## Partition and Disk Setup

1. View the current disk setup with the command: 'lsblk'
2. Enter the following command: 'cfdisk /dev/sda' and select GPT for the label type
3. Press Enter to create a new partition and allocate 500MiB by typing '500M' 
    1. Press the right arrow to change 'Type' to 'EFI File System'
4. Press the down arrow to select 'Free Space' then press New again, and allocate approximatley 70-80% of your remaining disk space
5. Create a third and final disk partition with the remainder of your disk space, and change the Type to 'Linux Swap'
6. Use the arrow keys to select 'Write' and press enter. TYPE OUT YES (I missed this instruction about 12 times and was infuriating myself because the changes weren't writing)
7. Select 'Quit' and press enter to exit cfdisk
8. Double check your partitions by entering the command 'lsblk' again. You should have sda1, sda2, and sda3
9. Activate your swap partition with the following commands:
    1. mkswap /dev/sda3
    2. swapon /dev/sda3
10. Create the root file system on your large disk partition with the command: 'mkfs.ext4 /dev/sda2'
11. Now, create your EFI filesystem on sda1 with the command: 'mkfs.fat -F32 /dev/sda1'
12. Now to proceed with the install, you must mount your filesystems with the command: 'mount /dev/sda2 /mnt'
13. Designate a boot directory with the command: 'mkdir /mnt/boot'
14. Next, mount your EFI filesystem to that boot directory with 'mount /dev/sda1 /mnt/boot'


## Install Arch base

1. Install the base Arch files with 'pacstrap /mnt base linux linux-firmware'
2. Generate a file to tell the disk where to mount partitions on boot with 'genfstab -U /mnt >> /mnt/etc/fstab'
3. Now chroot into the base filesystem with 'arch-chroot /mnt'
4. Change the timezone of your system with 'ln -sf /usr/share/zoneinfo/US/Central /etc/localtime'
    1. If in another timezone, select the appropriate combination of Region and City
5. Install a text editor of your choice with pacman using 'pacman -S vim'
6. Edit the /etc/locale.gen and uncomment 'en_US.UTF-8 UTF-8' or whichever CharacterSet is appropriate
7. Generate the locales with the command 'locale-gen'
8. 



## config

1. Create a locale.conf file using the command 'vim /etc/locale.conf'
    1. Add 'LANG=en_US.UTF-8' to the file
2. Edit /etc/hostname and add your chosen hostname and save the file
3. Edit /etc/hosts with your chose hostname. It should look like this when complete:
    1. 127.0.0.1   localhost
    2. ::1         localhost
    3. 127.0.1.1   HOSTNAME.localdomain  HOSTNAME
4. Enable your network tools to continue working after a reboot with: 
    1. systemctl enable systemd-networkd
    2. systemctl enable systemd-resolved
5. Determine your IP interface name by using the command: 'ip addr'
6. Pay attention to the 'ens33' interface and the attatched name. Note this 'Name' for the next step
7. Edit /etc/systemd/network/20-wired.network and enter the following:
    1. [Match]
    2. Name=ens33

    3. [Network]
    4. DHCP=yes
8. Set the password for your root user with: 'passwd'
9. Install the Intel Microcode if using an Intel processor: 'pacman -S intel-ucode'

## Bootloader setup (GRUB on UEFI)

1. Install GRUB and the EFI helper tools with:
    1. 'pacman -S grub efibootmgr'
2. Install GRUB to your EFI system partition (we mounted it at /boot earlier) with:
    1. 'grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB'
3. Generate a GRUB configuration file so it actually knows what to boot:
    1. 'grub-mkconfig -o /boot/grub/grub.cfg'


## Finish base install and reboot into Arch

1. Double check that everything looks right:
    1. 'lsblk' (make sure /dev/sda2 is mounted at / and /dev/sda1 is mounted at /boot)
    2. 'cat /etc/fstab' (make sure you see entries for /, /boot, and swap)
2. Exit the chroot environment with the command: 'exit'
3. Unmount all the filesystems you mounted under /mnt with:
    1. 'umount -R /mnt'
4. Reboot your VM with the command: 'reboot'
5. After the VM powers off, go into your VM settings and disconnect the Arch ISO so the VM boots from the virtual disk and not the installer
6. When the VM boots again, you should land on a text login prompt. Log in as 'root' with the password you created earlier


## First boot checks

1. Confirm that your network still works after reboot:
    1. 'ping -c 4 www.linuxconfig.org'
2. Make sure systemd-networkd and systemd-resolved are actually running:
    1. 'systemctl status systemd-networkd'
    2. 'systemctl status systemd-resolved'
3. Sync and update your system packages (this may take a minute): 
    1. 'pacman -Syu'


## Install a different shell (zsh) and make it the default

1. Install zsh with:
    1. 'pacman -S zsh'
2. (Optional but nice) Install some helpful extras for zsh like completions:
    1. 'pacman -S zsh-completions'
3. You can leave root as bash if you want, but for normal users we will set zsh as the default shell in a later section when we create the user accounts


## Create users and configure sudo

1. Install sudo with the command:'pacman -S sudo'
2. Create a main user account with the commands:
    1. 'useradd -m -G wheel,audio,video,storage -s /usr/bin/zsh YOURNAME'
    2. Set the password for your user with: 'passwd YOURNAME'
3. Create a user account for codi with the same permissions:
    1. 'useradd -m -G wheel,audio,video,storage -s /usr/bin/zsh codi'
    2. Set the password for codi with: 'passwd codi'
4. Enable sudo for users in the wheel group by editing the sudoers file:
    1. Run: 'visudo'
    2. Inside the file, find this line: '# %wheel ALL=(ALL) ALL'
    3. Uncomment it so it looks like: '%wheel ALL=(ALL) ALL'
    4. Save and exit (visudo will check for syntax errors before saving)
5. Log out of root and test sudo with your new user:
    1. At the login prompt, log in as 'YOURNAME'
    2. Run: 'sudo pacman -Syu'
    3. Make sure sudo asks for your password and then runs the command successfully


## Install SSH 

1. Log in as root or use sudo from your user, then install OpenSSH with the command:'pacman -S openssh'
2. Enable the SSH daemon so it starts on boot:'systemctl enable sshd'
3. Start SSH immediately without rebooting:'systemctl start sshd'


## Add color coding and aliases to your shell (zsh)

1. Turn on color for pacman output so it looks more like the Arch installer:
    1. As root (or with sudo), edit the pacman config: 'vim /etc/pacman.conf'
    2. Find the line that says '#Color' and remove the '#'
    3. (Optional, but recommended) Also uncomment 'VerbosePkgLists' for nicer package lists
    4. Save and exit
2. Now give yourself a nicer colored terminal and some useful aliases in zsh:
    1. Log in as your normal user (YOURNAME), not root
    2. Open your zsh configuration file: 'vim ~/.zshrc'
    3. Add the following lines somewhere near the top:
        1. 'autoload -U colors && colors'
        2. 'alias ls="ls --color=auto"'
        3. 'alias grep="grep --color=auto"'
        4. 'alias ll="ls -lah"'
        5. 'alias update="sudo pacman -Syu"'
        6. 'alias cls="clear"'
    4. Save and exit the file
    5. Reload your config with: 'source ~/.zshrc'
3. Copy the same configuration over to codi so they get all the colors and aliases too:
    1. As YOURNAME (or root), run: 'cp ~/.zshrc /home/codi/.zshrc'
    2. Fix ownership so codi actually owns their config: 'chown codi:codi /home/codi/.zshrc'
4. Log in as codi and verify:
    1. Log out and log back in as 'codi'
    2. Run 'ls' and 'update' and make sure colors/aliases are working
5. At this point you have:
    1. A different shell installed (zsh instead of just bash)
    2. Color coded terminal output similar to the Arch installer
    3. A handful of useful aliases in your shell configuration file


## Install Xorg (graphics) and LXQt desktop environment

1. Log in as root (or use sudo from your user) and install the Xorg display server and some basic graphics packages:
    1. 'pacman -S xorg-server xorg-xinit mesa xf86-video-vmware'
2. Install LXQt (light-weight desktop environment) plus a terminal and icon theme:
    1. 'pacman -S lxqt qterminal breeze-icons'
3. (Optional but recommended for VMware) Install open-vm-tools so things like clipboard and resizing work better:
    1. 'pacman -S open-vm-tools'
    2. Enable the main services:
        1. 'systemctl enable vmtoolsd.service'
        2. 'systemctl enable vmware-vmblock-fuse.service'
4. Reboot or log out/log back in later to let open-vm-tools fully kick in once the GUI is working


## Install and enable a display manager (SDDM) and boot to GUI

1. Install SDDM, the display manager (login screen) recommended for LXQt:
    1. 'pacman -S sddm sddm-kcm'
2. Enable SDDM so it starts automatically at boot:
    1. 'systemctl enable sddm'
3. Set the system to boot into a graphical target by default (instead of a text console):
    1. 'systemctl set-default graphical.target'
4. Reboot your VM one more time:
    1. 'reboot'
5. After reboot, you should see the SDDM graphical login screen instead of the plain text login
    1. Log in as your normal user (YOURNAME)
    2. On first login, if prompted for session type, choose 'LXQt Session'
6. Once logged in, you should be sitting in an LXQt desktop with:
    1. A working panel and menu
    2. qterminal as your terminal emulator (open it up and make sure your zsh colors and aliases are still working)
    3. Better integration inside VMware if you installed open-vm-tools (clipboard, window resizing, etc.)

