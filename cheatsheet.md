# How to get source code for Linux binaries
$ pkgctl repo clone --protocol=https zsh  
or  
$ makepkg --nobuild --skippgpcheck --skipchecksums  
if one or more files can't pass the validity check. Then run  
$ cd zsh  
$ makepkg --nobuild --skippgpcheck  
  
or install asp from the AUR and run  
$ asp export zsh  
$ cd zsh  
$ makepkg --nobuild --skippgpcheck  
  
# GDB  
save breakpoints <filename>  
  Save all current breakpoint definitions to a file suitable for use  
  in a later debugging session.  To read the saved breakpoint  
  definitions, use the "source" command.  
Reference: https://sourceware.org/gdb/current/onlinedocs/gdb.html/Save-Breakpoints.html  
  
  ## Building a program written in c that can be debugged with gdb without ignoring static functions and stuff like that  
  $ ./configure CFLAGS="-g3 -O0 -fno-inline -fno-lto" LDFLAGS="-g"  
  $ make  
  $ objdump --syms Src/zsh | grep debug  
  $ file Src/zsh <!--- should say "with debug_info". -->  
  $ cd Src  
  $ gdb ./zsh  
  (gdb) break input.c:inputline <!--- inputline is a static function, but the breakpoint is set successfully. -->  
  Breakpoint 1 at 0x6b5a7: file input.c, line 370.  
  
# GIT  
$ git init  
$ git branch -m main <!--- to rename master to main. -->  
$ git status  
$ git remote add origin https://github.com/username/app.git  
$ git remote -v <!--- list all the configured remote repositories. -->  
$ git add wsh.c  
$ git commit  
$ git push origin main  
or  
$ git push -u origin main  <!--- Pushes main and sets tracking so next time you can just run git push  
                                 without specifying remote and branch. -->  
$ git mv file_from file_to <!--- to truly rename a file. -->  
$ git remote set-url origin NEW_URL <!--- If you renamed your repository on github, run this to update the local clone  
                                        to point to the new repository url. -->  
git blame <!--- to find out why certain code exists. --->  
  
  
# pacman errors  
  
  ## exists on filesystem  
  This worked for one update, then stuff broke again and using this command didn't work.  
  $ sudo pacman --overwrite "*" -Syu  
    
  This works:  
  $ sudo pacman -S <files>  
  where <files> is all the files on your system. Or run  
  $ sudo pacman -S --overwrite '*' $(< pkglist.txt)  
    
  I think also just deleting everything that exists on filesystem and then running sudo pacman -Syu has worked for me.  
  
  ## error: failed to prepare transaction (could not satisfy dependencies)  
  [harrol3@kold ~]$ sudo pacman -Syu  
  ...  
  :: Starting full system upgrade...  
  resolving dependencies...  
  looking for conflicting packages...  
  error: failed to prepare transaction (could not satisfy dependencies)  
  :: installing icu (76.1-1) breaks dependency 'libicui18n.so=75-64' required by electron28  
  :: installing icu (76.1-1) breaks dependency 'libicuuc.so=75-64' required by electron28  
  :: installing flac (1.5.0-1) breaks dependency 'libFLAC.so=12-64' required by electron28  
  :: installing icu (76.1-1) breaks dependency 'libicui18n.so=75-64' required by electron30  
  :: installing icu (76.1-1) breaks dependency 'libicuuc.so=75-64' required by electron30  
  :: installing flac (1.5.0-1) breaks dependency 'libFLAC.so=12-64' required by electron30  
    
  [harrol3@kold ~]$ pacman -Qi electron28  
  Name            : electron28  
  Version         : 28.3.3-3  
  ...  
  Required By     : None  
  ...  
  Поэтому можно просто   
  [harrol3@kold ~]$ sudo pacman -Rs electron28 electron30  
  
  ## WARNING: Deprecated option 'ALL_microcode' found. Update '/etc/mkinitcpio.d/linux-lts.preset' to use the 'microcode' hook instead.  
    
  fix:  
    
  1. /etc/mkinitcpio.conf  --- added in HOOKS section microcode  
    
  HOOKS=(base udev keyboard autodetect microcode modconf kms keymap consolefont block encrypt filesystems fsck)  
    
  2. hashed ALL_microcode  
    
  #/etc/mkinitcpio.d/linux.preset --- hashed line ALL_microcode  
    
  mkinitcpio preset file for the 'linux' package  
    
  #ALL_config="/etc/mkinitcpio.conf"  
  ALL_kver="/boot/vmlinuz-linux"  
  #ALL_microcode=(/boot/*-ucode.img)  
    
  PRESETS=('default' 'fallback')  
    
  #default_config="/etc/mkinitcpio.conf"  
  default_image="/boot/initramfs-linux.img"  
  #default_uki="/efi/EFI/Linux/arch-linux.efi"  
  #default_options="--splash /usr/share/systemd/bootctl/splash-arch.bmp"  
    
  #fallback_config="/etc/mkinitcpio.conf"  
  fallback_image="/boot/initramfs-linux-fallback.img"  
  #fallback_uki="/efi/EFI/Linux/arch-linux-fallback.efi"  
  fallback_options="-S autodetect"  
    
  3. rebuilding mkinitcpio by command:  
    
  mkinitcpio -p linux  
    
  4. restart sytems (Linux Arch + BTFRS + encyption + systemd-boot + timeshift)  
    
  Reference: https://bbs.archlinux.org/viewtopic.php?id=293437  
  

# Packages that I installed in weird ways  
  ## vimplug  
  $ sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \  
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'  
  

# Laptop  
  
  ## my lsblk setup  
  NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS  
  nvme0n1     259:0    0 476.9G  0 disk   
  ├─nvme0n1p1 259:1    0     1G  0 part /boot/efi  
  ├─nvme0n1p2 259:2    0   1.5G  0 part [SWAP]  
  ├─nvme0n1p3 259:3    0  97.7G  0 part /  
  └─nvme0n1p4 259:4    0 295.3G  0 part /home  
  
  ## Backing up home  
  $ sudo tar -cvf /run/media/harrol3/home-backup/home-backup.tar /home/harrol3  
  
  ## Disable beep  
  $ setterm --blength=0  
  
  ## If stuck in sleep mode  
  Always start with removing/disconnecting the battery as well as the power cord and then holding   
  the power button down for around 20 seconds to discharge the capacitors. then put the battery back in and   
  see if the issue is fixed.  
  
  ## Drive formatting  
  To format the entire `sda` drive as `exfat`, run the following:  
  $ sudo mkfs.exfat /dev/sda  
  You can optionally use the `-n` tag to name your filesystem like so:  
  $ sudo mkfs.exfat -n kingston /dev/sda  
  
  ## Syncthing  
  To disable syncthing, run:  
  $ sudo systemctl stop syncthing@username  
  
  ## KDE-specific  
  Disable and re-enable your Global theme and kwin scripts after major updates to avoid `dolphin` slowing down.  
  This will probably require you to re-enable custom icons as well. Also if your konsole colors change,  
  press "Ctrl + Shift + ," go to Profiles > Appearance and edit the colors there.  
  The color 5 colors the directories in zsh.  
  
  ## Printer drivers  
  Used this website to find printer driver https://www.openprinting.org/printers  
  installed this and chose the relevant foo2hp driver  
  https://aur.archlinux.org/packages/foo2zjs-nightly  
  also installed hplip which didn't help  
  
  
# Network connection  
  
If you can't connect to the internet all of a sudden:  
1. turn off any network-related services you have running using systemctl  
2. networkmanager restart  
3. reboot.  
  
# Arch installation  
  ## Grub configuration  
  After using arch-chroot (step 3.2) and installing grub, you need to configure it with the following commands:  
  $ grub-install /dev/_nameofyourdrive_  
    
  Make sure that the specified path doesn't point to a partition. It should point to the whole drive you're configuring grub on.  
  Now generate the grub configuration file:  
  $ grub-mkconfig -o /boot/grub/grub.cfg  
    
  If you encounter the warning "Warning: os-prober will not be executed to detect other bootable partitions", run  
  $ nano /etc/default/grub  
    
  There you need to look for the setting "GRUB_DISABLE_OS_PROBER=false" which is probably commented out. You need to uncomment it.  
    
  ## Post-installation  
  Use https://wiki.archlinux.org/title/NetworkManager#nmcli_examples nmcli commands to connect to wifi.  
  
# How do install from AUR without yay  
$ git clone https://aur.archlinux.org/_repositoryname_.git  
$ cd _repositoryname_  
$ makepkg  
  
Now you should have a _repositoryname_.pkg.tar.zst file in your working directory.  
Use `ls` to display it's full name, copy it and then run  
$ sudo pacman -U _nameofthefileyoujustcopied_.pkg.tar.zst  
  
# nvim  
I didn't like the way matching parentheses were formatted so I went into   
.local/share/nvim/site/pack/packer/start/nord.nvim/lua/nord/theme.lua and edited line 91 to look like this:  
MatchParen = { fg = nord.nord3_gui_bright, bg = nord.nord4_gui, style = "reverse"},  
Reference: https://github.com/shaunsingh/nord.nvim  
  
# tmux basics  
Whenever you open a new terminal, run tmux as your first command.  
When you’re done, before you close the terminal press ‘Ctrl+B’ followed by ‘d’ for ‘detatch’. This will basically keep your session running in the background after the terminal closes.  
Then, when you open your next terminal session, run tmux a for ‘attach’ and you’ll be back where you left off.  
A huge bonus of this is that you can leave stuff running while the terminal is closed, like a long update or install command.  
  
  
# Shortcuts  
  ## Text shortcuts  
  1. ctrl/cmd + arrow = move the cursor to the start/end of a line.   
     If pressed when the cursor is already at the end of a line, the cursor will move to the end of the paragraph;  
  2. cmd + Backspace = deletes the line before cursor;  
  3. alt/option + arrow = move to the start of next/previous word;  
  4. alt/option + Backspace = deletes the word before cursor;  
  5. shift + arrow = selects the text. Becomes very powerful when combined with the shortcuts 1 and 3.  
  ## Browser shortcuts  
  1. ctrl/cmd + T = open new tab;  
  2. ctrl/cmd + W = close current tab;  
  3. ctrl/cmd + Tab = switch to the next tab;  
  4. ctrl/cmd + Shift + Tab = switch to the previous tab;  
  5. ctrl/cmd + F = find in page;  
  6. ctrl/cmd + mousewheel up/ mousewheel down = zoom in/ zoom out;  
  7. cmd + arrow = switch between tabs; ^2a6ef0  
  8. ctrl/cmd + Shift + T = open closed tab;  
  ## Windows shortcuts  
  1. Win + D = minimize all windows;  
  2. Win + downarrow = minimize top window;  
  3. F11 = fullscreen mode;  
  4. Win + ; = emoji keyboard;  
  5. ctrl + End = go to bottom of page;  
  ## MacOS shortcuts  
  1. cmd + Space = change language;  
  2. cmd + H = hide window;  
  3. cmd + Tab = reveals an open or hidden window.  
     To close it - hold down cmd and press Tab to get to the app the window of which you want to reveal  
     and then release your finger off of cmd;  
  4. cmd + M = minimize the front window to the Dock   
  5. cmd + Tab - cmd+ alt/option = to select application the window of which you want to un-minimize by using cmd and Tab,  
     and then, while still holding cmd, press alt/option to un-minimize the window;  
  <!--- So, basically, you have to choose between hiding your windows and minimizing them to the dock.  
  Hiding happens instantly, without the animation that minimizing has and you can hide multiple windows   
  without clicking on each one, you just need to press cmd + H a few times until every window is hidden.  
  To open a hidden window you just need to cmd + Tab to the application.  
  Minimizing doesn't happen as fast as hiding does, to minimize multiple windows you have to click on each one.  
  To bring up a minimized window you have to press cmd + Tab and then alt/option while still holding down cmd,  
  which is manageable, but isn't as fast as cmd + Tab. -->  
  6. ctrl + arrow = switch between windows;  
  7. cmd + Backspace = erase the line before cursor;  
  8. cmd + ctrl + F = fullscreen mode;  
