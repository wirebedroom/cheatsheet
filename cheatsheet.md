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
  Save all current breakpoint definitions to a file suitable for use in a later debugging session. To read the saved breakpoint definitions, use the "source" command.  
Reference: https://sourceware.org/gdb/current/onlinedocs/gdb.html/Save-Breakpoints.html  
  
  ## Building a program written in c that can be debugged with gdb without ignoring static functions and stuff like that  
  $ ./configure CFLAGS="-g3 -O0 -fno-inline -fno-lto" LDFLAGS="-g"  
  $ make  
  $ objdump --syms Src/zsh | grep debug  
  $ file Src/zsh // should say "with debug_info".  
  $ cd Src  
  $ gdb ./zsh  
  (gdb) break input.c:inputline // inputline is a static function, but the breakpoint is set successfully.  
  Breakpoint 1 at 0x6b5a7: file input.c, line 370.  
  
# GIT  
$ git init  
$ git branch -m main // to rename master to main.  
$ git status  
$ git remote add origin https://github.com/username/app.git  
$ git remote -v // list all the configured remote repositories.  
$ git add wsh.c  
$ git commit  
$ git push origin main  
or  
$ git push -u origin main  // Pushes main and sets tracking so next time you can just run git push without specifying remote and branch.  
$ git mv file_from file_to // to truly rename a file.   
$ git remote set-url origin NEW_URL // If you renamed your repository on github, run this to update the local clone to point to the new repository url.  
git blame // to find out why certain code exists.  
  
  
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
  Therefore we can just
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

  ## Mounting wisdom
  - EFI can be in /boot/efi. Some people prefer /boot instead, but I prefer /boot/efi because kernels are stored in /boot, and may conflict if you dual-boot multiple Linux distros, and I don't put my EFI partition that large anyway 
  - BIOS boot only exists if you run MBR and legacy BIOS IIRC, I don't think legacy BIOS supports GPT, and the boot records are stored in the partition table, not in a partition file system. 
  - Swap doesn't get mounted, you run swapon on the file. To format a partition or file as swap, run mkswap on it. I usually choose a file on a file system instead of a separate partition, to do that, run fallocate -l <size> <file> e.g fallocate -l 8G /swap, then mkswap /swap and make the permissions secure with chown root: /swap && chmod 600 /swap 
  - Root is mounted at /. In /etc/fstab, set the "pass" option (the last number) to 1. Root should be 1, other file systems should be 2 to be checked for errors (fsck), and other things should be 0 like swap which can't/won't be checked for errors. 
  - Home partition depends. If you want to use the partition for all users, have the user directories in the partition and mount it at /home. If you want it just for one user (e.g user) mount it at /home/user (make sure to chown the mount directory after mounting it). Although your home directory doesn't have to be in /home, it's generally recommended to have it there 
  - For other partitions, like files, I usually just mount them in some directory in the root directory, like /files, but usually you can mount them in /mnt or in a subdirectory of /mnt. Removable media support should come with your DE, if you don't have one, you can use something like udiskie, they will be mounted in /run/media/username, I like to create a symlink /media → /run/media.
  
  Reference: https://www.reddit.com/r/archlinux/comments/15av7rm/where_do_i_mount_my_partitions/

  ## Backing up home  
  $ sudo tar -cvf /run/media/harrol3/home-backup/home-backup.tar /home/harrol3  
  
  ## Disable beep  
  $ setterm --blength=0  
  
  ## If stuck in sleep mode  
  Always start with removing/disconnecting the battery as well as the power cord and then holding the power button down for around 20 seconds to discharge the capacitors. Then put the battery back in and see if the issue is fixed.  
  
  ## Drive formatting  
  To format the entire `sda` drive as `exfat`, run the following:  
  $ sudo mkfs.exfat /dev/sda  
  You can optionally use the `-n` tag to name your filesystem like so:  
  $ sudo mkfs.exfat -n kingston /dev/sda  
  
  ## Syncthing  
  To disable syncthing, run:  
  $ sudo systemctl stop syncthing@username.service
  It should output
  Removed '/etc/systemd/system/multi-user.target.wants/syncthing@username.service'
  
  ## KDE-specific  
  Disable and re-enable your Global theme and kwin scripts after major updates to avoid `dolphin` slowing down. This will probably require you to re-enable custom icons as well. Also if your konsole colors change, press "Ctrl + Shift + ," go to Profiles > Appearance and edit the colors there. The color 5 colors the directories in zsh.  
  
  ## Printer drivers  
  Used this website to find printer driver https://www.openprinting.org/printers nstalled this and chose the relevant foo2hp driver https://aur.archlinux.org/packages/foo2zjs-nightly also installed hplip which didn't help.  
  
  ## List systemd services
  $ systemctl --type=service

  ## Reflector mirrors
  $ sudo reflector --save /etc/pacman.d/mirrorlist --protocol https --country Russia,Belarus,Latvia,Estonia,Lithuania,Poland,Finland,Sweden,Moldova --latest 30 --sort age --fastest 30

# Network connection  
  
If you can't connect to the internet all of a sudden:  
1. turn off any network-related services you have running using systemctl  
2. networkmanager restart  
3. reboot.  

  ## Can't connect to dreamwidth.org when using VPN, 100% packet loss.  
  The issue ended up being with HTTP/2. Browsers default to it and so does curl.  
  Here's the troubleshooting process:  
  1. Check for DNS inconsistency by running  
    $ ping dreamwidth.org  
    multiple times or running  
    $ ping dreamwidth.org  
    $ traceroute dreamwidth.org  
    The latter command will tell you where the packet loss is. In my case it was the AWS servers.
    In my case ping and traceroute returned different IPs, which was inconsistent. So I ran  
    $ nslookup dreamwidth.org  
    And then ran  
    $ dig dreamwidth.org @1.1.1.1 +short  
    to test if the Cloudflare DNS (which has IP 1.1.1.1) resolved to the same IPs as my computer's DNS server,
    which it did, therefore DNS wasn't the problem.   

  2. Then I did  
    $ curl -v dreamwidth.org  
    It succeeded, which I could tell by  
    - Connected to dreamwidth.org (54.146.86.95) port 443  
    - SSL connection using TLSv1.3 [...] verify ok  
    - < HTTP/2 302  
    while ping failed, which meant that AWS was blocking ICMP (ping) but allowing HTTPS traffic.
    Then I tested the redirect manually:  
    $ curl -vL "https://dreamwidth.org"  # -L follows redirects  
    If this works (returns 200 OK), then your browser is the issue: it is failing to handle the redirect.  
    In my case the first request (dreamwidth.org) went fine,
    then the program began requesting www.dreamwidth.org and froze during TLS handshake. Then I ran  
    $ curl -v --http1.1 https://dreamwidth.org  
    which returned 302 without hanging, which meant that HTTP/2 was broken over my VPN for dreamwidth.org.  
    I confirmed this by running  
    $ curl -v --http2 https://www.dreamwidth.org  
    which hung.  

  3. To disable HTTP/2 for all websites in Firefox, type about:config in address bar, 
    then search for network.http.http2.enabled and set it to false.  

  ## Change DNS server until reboot  
  Backup /etc/resolv.conf, then  
  $ nvim /etc/resolv.conf  
  and put, for example, these two lines inside:  
  nameserver 1.1.1.1 # Cloudflare DNS  
  nameserver 8.8.8.8 # Google DNS  

  ## Outline  
  On startup the resolv.conf is generated by NetworkManager, also Outline autoopens after every reboot. If you turn on Outline VPN after rebooting, it will make the /etc/resolv.conf the following:  
  nameserver 9.9.9.9  
  options use-vc  

  ## If you hotspot from android phone that's connected to VPN, that VPN doesn't get transferred to PC.  

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

# KVM Windows 10 setup on Arch  
$ sudo pacman -S qemu-desktop  
$ yay -S quickemu  
$ quickget windows 10  
$ quickemu --vm windows-10.conf  
  
For file sharing you need to install samba, create a file /etc/samba/smb.conf and paste all text from https://git.samba.org/samba.git/?p=samba.git;a=blob_plain;f=examples/smb.conf.default;hb=HEAD into it. Then run  
$ sudo systemctl enable smb.service  
$ quickemu --vm windows-10.conf  
By default samba shares ~/Public.  
  
If you get Display 'sdl' is not available, you need to install qemu-desktop.  
If you get ERROR! QEMU 6.0.0 or newer is required, detected 10.0.0., then you need to open /usr/bin/quickemu, go to line 1941, which should read  
QEMU_VER_SHORT=$(echo "${QEMU_VER_LONG//./}" | cut -c1-2)  
and replace it with  
QEMU_VER_SHORT=$(echo "${QEMU_VER_LONG//./}" | sed 's/.$//')  
  
References:  
1. https://forum.endeavouros.com/t/quickemu-currently-broken/71612/6  
2. https://github.com/quickemu-project/quickemu/issues/1647  
And archwiki, of course.  

  ## Clipboard sharing to Windows guest
  Download https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-latest.exe  

# GParted glitch  
After I tried to boot into a flashdrive with GParted, my arch installation disappeared from BIOS. To fix it, I booted into a flashdrive with archiso, mounted everything with  
$ mkdir /mnt/boot/efi  
$ mount /dev/nvme0n1p1 /mnt/boot/efi  
then did  
$ arch-chroot /mnt  
$ mount /dev/nvme0n1p4 /home   
Then I reinstalled the kernel with   
$ sudo pacman -S linux   
and ran  
$ grub-install /dev/nvme0n1  
$ exit  
$ reboot  
  
I think reinstalling grub was what fixed the issue, reinstalling the kernel was probably unnecessary.  
  
# How to install from AUR without yay  
$ git clone https://aur.archlinux.org/_repositoryname_.git  
$ cd _repositoryname_  
$ makepkg  
Now you should have a _repositoryname_.pkg.tar.zst file in your working directory.  
Use `ls` to display it's full name, copy it and then run  
$ sudo pacman -U _nameofthefileyoujustcopied_.pkg.tar.zst  
  
# neovim  
I didn't like the way matching parentheses were formatted so I went into   
.local/share/nvim/site/pack/packer/start/nord.nvim/lua/nord/theme.lua and edited line 91 to look like this:  
MatchParen = { fg = nord.nord3_gui_bright, bg = nord.nord4_gui, style = "reverse"},  
Reference: https://github.com/shaunsingh/nord.nvim  
  ## Interactive find and replace
  :%s/thick/\\orbitThickness/gc
  The c flag means confirmation.
  ## Installing neovim on Windows  
  Run this with admin privileges:  
  $ winget install neovim  

# Installing winget on Windows  
Go to https://github.com/microsoft/winget-cli, then Releases, download the .msix bundle and run it.  
  
# tmux basics  
Whenever you open a new terminal, run tmux as your first command.  
When you’re done, before you close the terminal press ‘Ctrl+B’ followed by ‘d’ for ‘detatch’. This will basically keep your session running in the background after the terminal closes.  
Then, when you open your next terminal session, run tmux a for ‘attach’ and you’ll be back where you left off.  
A huge bonus of this is that you can leave stuff running while the terminal is closed, like a long update or install command.  

# Krita cropping  
The crop is meant to crop the entire image. If you want to remove redundant content of a layer try to make a selection, then Ctrl+Shift+I to invert the selection and then Del to remove content.  
  
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
  8. ctrl/cmd + Shift + T = open closed tab.

  ## MacOS shortcuts  
  1. cmd + Space = change language;  
  2. cmd + H = hide window;  // better than minimizing.  
  3. cmd + Tab = reveals an open or hidden window.  
     To close it - hold down cmd and press Tab to get to the app the window of which you want to reveal  
     and then release your finger off of cmd;  
  4. cmd + M = minimize the front window to the Dock   
  5. cmd + Tab - cmd+ alt/option = to select application the window of which you want to un-minimize by using cmd and Tab,  
     and then, while still holding cmd, press alt/option to un-minimize the window;  
  6. ctrl + arrow = switch between windows;  
  7. cmd + Backspace = erase the line before cursor;  
  8. cmd + ctrl + F = fullscreen mode.  

# Windows  
  ## Windows shortcuts  
  1. Win + D = minimize all windows;  
  2. Win + downarrow = minimize top window;  
  3. F11 = fullscreen mode;  
  4. Win + ; = emoji keyboard;  
  5. ctrl + End = go to bottom of page.  
  ### Multiple desktops  
  1. Ctrl + Win + D = open a new desktop;  
  2. Ctrl + Win + F4 = close the current one;  
  3. Ctrl + Win + Left/Right arrow = cycle through the desktops.  

  ## Things you cannot do on Windows:  
  - make Windows Explorer open a specific folder on launch.  

  ## MS Word  
  Turning formatting symbols on can help.
  ### Page numbering  
  Say you want to start page numbering from page 3. When you insert page numbering there, it will appear on pages 1 and 2 as well. After that go to page 3, Insert > Header and footer > special formatting for first page (or smth like that). This will make the numbering on pages 1 and 2 disappear.  
  ### How to remove a footer that won't go away  
  Make a section break from the next page, then disable the "same as in the previous section" button, then double-click on the ruler and select the value of the upper and lower margins.  
  ### How to add numbers to equations  
  You can create a template that can be used to automatically generate the table and equation with number to the right:  
    1. Insert → Table → 3x1.  
    2. Right click table → Table Properties.  
    3. In Table Tab, Check Preferred Width → Percent → 100.  
    4. In Column Tab, set preferred width to 7%, 86% and 7% for 1st 2nd and 3rd column respectively. These values work well for Times New Roman 12pt equation numbers. (Other percentages will work provided they add up to 100%.)  
    5. Click Ok.  
    6. Insert → Equation into center column (type in current equation or placeholder).  
    7. Click References → Insert Caption. Select Label: Equation Position: Above or Below  
    8.  Adjust numbering as desired.  
    9.  Cut and past number from above location to right column of equation table.  
    10. Right-align text in right column.  
    11. Center equation column.  
    12. Highlight the entire table.  
    13. Turn off borders.  
    14. Re-highlight the entire table.  
    15. Select Insert → Equation → Save Selection to Equation Gallery.  
  Now if you want to insert an equation with automatic numbering in standard journal/conference paper format, just select the template you have made from the equation gallery and it will insert it into the document as desired.  
  Equation numbers will be automatically updated and references can be made to them using the References → Cross Reference option for equations.  
  NOTE: If you'd like to save this newly formatted equation as a keyboard shortcut (like pressing the Alt and + keys simultaneously in order to create a new equation), you can do so by going to File → Options → Customize Ribbon → Customize Shortcuts and then selecting "Building Blocks". Search for your newly created equation template in the right list, then assign a keyboard shortcut to it.  
  Reference: https://superuser.com/questions/594559/how-do-you-easily-add-equation-numbers-to-microsoft-word-2010-equations  

  ## MathCAD 15  
  ### Export plots in high quality
  Save your file as html. The images will be saved in highest quality.  
  ### Cannot add normal Delta and italic i as captions to graph because turning on italic makes all text italic  
  First turn off italic, type the Delta and drag it onto the graph. Now turn on italic -- the Delta will not change. Type i, drag it to the graph.  

# Latex  
  ## Saving tikz images to png   
  \documentclass[tikz,border=10pt,convert]{standalone} % convert makes it png.  

  ## Manual font installation  
  
### Zeroth  
  
Before you start make sure, that there is really no possibility for an installation with the package manager of your TeX distribution. For TeX Live see also the script getnonfreefonts (here on TeX.SX user dcmt wrote a nice answer as well) and this question and answer: Error in TeX Live – Font ... not loadable: Metric (TFM) file not found .  
  
In wide parts already described in short in TeX Users Group: Installing TeX fonts.  
  
Siep Kroonenberg describes a different method in Font installation the shallow way (PDF, website of Dutch TeX users group NTG; the same as TUGboat article: tb86kroonenberg-fonts.pdf.)  
  
### First  
  
You have to install the font files into a local texmf tree.  
  
    For MiKTeX you perhaps have to create such a local texmf tree.  
  
    In TeX Live you somewhere have already a directory texmf-local for a system-wide local installation (system variable TEXMFLOCAL), but you can also create a user specific path (TEXMFHOME). To find out the actual paths or whether they already exist you can input kpsewhich --var-value=TEXMFLOCAL or with TEXMFHOME respectively.  
    Installing of fonts in TEXMFHOME is not recommended, because afterwards you must do a manual update every time any Type1 font (see below) related stuff is updated! See Why shouldn't I use getnonfreefonts to install additional fonts? Why shouldn't I use updmap when installing or removing fonts? for all the gory details (and what to do if you read this warning too late).  
  
If the font package comes in a packed file, you can in most cases safely extract with subdirectories into this local texmf tree, but you should control this before doing so by comparing with the following tree structure. For instance perhaps all needed subdirectories are packed under a needless texmf directory.  
Note: Sometimes map files are twice included there. In recent TeX distributions you only need the ones, which are found as mentioned below in <local-texmf>/fonts/map/<engine-name>/..., not these in <local-texmf>/<engine-name>/... – mostly <engine-name> is dvips, but sometimes also pdftex, and rarely just another one.  
  
If there is no subdirectory structure in the packed file, you have to create it under <local-texmf> copying the structure of the according main texmf tree (known as TDS, the TeX Directory Structure) – here an almost full structure is shown (in the main texmf tree[s] it is actually more complex), for a single font package very likely only parts are necessary:  
  
<local-texmf>  
      +--fonts  
          |--afm  
          |   +--<font-package-name>  
          |           +--<*.afm files>  
          |--enc  
          |   +--<engine-name>  
          |           +--<font-package-name>  
          |                   +--<*.enc files>  
          |--map  
          |   +--<engine-name>  
          |           +--<font-package-name>  
          |                   +--<*.map files>  
          |--ofm  
          |   +--<font-package-name>  
          |           +--<*.ofm files>  
          |--opentype  
          |     +--<font-package-name>  
          |             +--<*.otf files>  
          |--ovf  
          |   +--<font-package-name>  
          |           +--<*.ovf files>  
          |--ovp  
          |   +--<font-package-name>  
          |           +--<*.ovp files>  
          |--pfm  
          |   +--<font-package-name>  
          |           +--<*.pfm files>  
          |--pk  
          |   +--<printer-type>  
          |           +--<font-package-name>  
          |                   +--<font-resolution>  
          |                          +--<*.pk files>  
          |--sfd  
          |   +--<font-package-name>  
          |           +--<*.sfd files>  
          |--source  
          |    +--<font-package-name>  
          |             +--<*.mf files>  
          |--tfm  
          |   +--<font-package-name>  
          |           +--<*.tfm files>  
          |--truetype  
          |     +--<font-package-name>  
          |             +--<*.ttf files>  
          |--type1  
          |    +--<font-package-name>  
          |            +--<*.pfa/*.pfb files>  
          +--vf  
              +--<font-package-name>  
                        +--<*.vf files>  
  
In the main texmf fonts tree(s) instead of <font-package-name> actually in almost all directories comes an additional <font-vendor> or public in between.  
  
Also most fonts nowadays are shipped out as packages with documentation and style files. These come in  
  
<local-texmf>  
      |--doc  
      |   +--fonts  
      |        +--<font-package-name>  
      |                |--<document files and their *.tex sources>  
      |                +--<example files>  
      +tex  
        |--latex  
        |    +--<font-package-name>  
        |             |--<*.fd files>  
        |             |--<*.sty files>  
        |             +--<*.tex files [no document sources and examples]>  
        +--plain  
             +--<font-package-name>  
                      |--<*.def files>  
                      +--<*.tex files [no document sources and examples]>  
  
Be careful that fonts and tex are at the same level  
  
<local-texmf>  
     |  
     +fonts  
     |  
     +tex  
  
### Second  
  
After copying the files you must update the file name data base of your TeX distribution:  
  
    In MiKTeX (note also Difference between administrative and user mode of MiKTeX )  
  
        Using the GUI: In the Start Menu go to the MiKTeX entry and open the settings – if you act as admin respectively “Settings (Admin)”, of course. The “MiKTeX Options” window will open. Go to the “General” tab and click there on “Refresh FNDB” (FNDB = File Name Data Base).  
  
        Using the CLI (command prompt): Execute initexmf --update-fndb (or shorter initexmf -u, as admin add the switch --admin).  
  
    In TeX Live  
        Linux and MacOSX: sudo mktexlsr (or, perhaps better to remember, with alias [symlink to mktexlsr]: sudo texhash.  
  
        Windows: mktexlsr (or the alias: texhash)  
  
For MetaFont files (*.mf) this is enough.  
  
For Open-/TrueType font files (*.otf/*.ttf, the directory names are misleading here, as the file extension does not say anything about OpenType abilities) you should additionally run fc-cache (MiKTeX and TeX Live Windows) or sudo fc-cache (TeX Live Linux and MacOSX) on the command prompt, but then you are ready as well. Nothing more needed for these fonts.  
  
But note, that as long as you do not work with a portable TeX distribution it is quite probably better to install Open- or TrueType fonts into the system wide font directory since this is anyway searched and the fonts are added to the font cache with fc-cache.  
  
MiKTeX portable users read also External font with portable MiKTeX, Version 2.9, please.  
  
### Third  
  
For Type1 fonts (see Wikipedia, PostScript fonts) further steps are required:  
  
    In MiKTeX (for MiKTeX portable read also External font with portable MiKTeX, Version 2.9!)  
  
        On command prompt (here no GUI possible) execute initexmf --edit-config-file=updmap.cfg (or initexmf --edit-config-file updmap). This will open your default text editor with the file updmap.cfg in your user profile under %AppData%\MiKTeX\<version>\miktex\config\. If it did not exist yet, it will be created.  
  
        If you add the switch --admin, the file will be created/opened in %AllUsersProfile%\MiKTeX\<version>\miktex\config\ (since Windows Vista) or %AllUsersProfile%\<Application Data>\MiKTeX\<version>\miktex\config\ (until Windows XP, the string <Application Data> is language dependent) and will be valid for all users without an own file name data base.  
  
        Add there at least the following (line with # is a comment, we assume here, the Map file has the name fontname.map).  
  
        # <font name> or <package name> or what fits better for you  
        Map fontname.map  
  
        Save and exit. If there is more to do, this should to be read in a readme file. For information you could also open the online help page updmap.cfg or the updmap.cfg in your main MiKTeX\miktex\config installation tree and read the comments in it (but do not edit, as every edit will be lost on a later update!).  
  
        Execute initexmf -u, this updates the font name data base only for the active user (you can leave this out, if you are sure, your local updmap.cfg already existed).  
  
        Execute initexmf --mkmaps (or shorter updmap).  
  
    In TeX Live (assuming the according map file name is fontname.map)  
  
        In Linux and MacOSX execute sudo updmap-sys --enable Map=fontname.map, in Windows updmap-sys --enable Map=fontname.map.  
  
        Make again an update of the file name data base, see second step above.  
  
    If you still have problems, you could try the approach given in this answer: Problems installing MathTime Professional 2 font on TexLive.  
  
  Reference: https://tex.stackexchange.com/questions/88423/manual-font-installation  

  ## Installing Times New Roman and Cambria Math  
  ### Times New Roman  
  $ yay -S ttf-times-new-roman  
  $ sudo pacman -S texlive-xetex  
  You might need to run $ sudo fmtutil-sys --all # To regenerate format files, including xelatex.fmt.  
  ### Cambria Math  
  Download cambria-math.ttf from this website: https://freefontdownload.org/en/cambria-math.font and put it inside /usr/local/share/fonts/ or smth. Verify with  
  $ fc-list | grep "Cambria Math"  
  /usr/local/share/fonts/cambria-math.ttf: Cambria Math:style=Regular  
  $ file /usr/local/share/fonts/cambria-math.ttf  
  /usr/local/share/fonts/cambria-math.ttf: TrueType Font data, digitally signed, 20 tables, 1st "DSIG", name offset 0x125b4c  
  $ ls -l /usr/local/share/fonts/cambria-math.ttf  
  -rwxr-xr-x 1 root root 1408700 Aug 14 15:30 /usr/local/share/fonts/cambria-math.ttf  

  ## Using Times New Roman and Cambria Math for a LaTeX Word-like document  
\ProvidesPackage{preamble-word}  
  
% Define Unicode character support first  
\providecommand\DeclareUnicodeCharacter[2]{}  
  
\usepackage{fontspec}  
\usepackage{unicode-math}  
  
% Load Times New Roman explicitly with fallbacks  
\setmainfont[  
  Path = /usr/share/fonts/TTF/, % Adjust if Times New Roman is elsewhere  
  UprightFont = times.ttf,  
  BoldFont = timesbd.ttf,  
  ItalicFont = timesi.ttf,  
  BoldItalicFont = timesbi.ttf,  
  Extension = .ttf  
]{Times New Roman}  
  
  
\setmathfont{cambria-math.ttf}[  
  Path = /usr/local/share/fonts/,  
  Extension = .ttf,  
  Scale = MatchLowercase  
]  
  
% Russian language support  
\usepackage{mathtext}  
\usepackage[T2A]{fontenc}  
\usepackage[none]{hyphenat}  
\usepackage[english,russian]{babel}  
  
% \usepackage{titlesec}  
% \titleformat*{\section}{\normalfont\fontsize{14}{18}\bfseries}  
  
% Section heading formatting  
\usepackage{titlesec}  
% For numbered sections  
\titleformat{\section}  
  {\normalfont\fontsize{14}{12}\bfseries\centering}  
  {\thesection} % Label  
  %{1em}% Horizontal separation  
  {0pt}% Horizontal separation  
  {}% Before-code  
% For unnumbered sections (like your \section*)  
\titleformat{name=\section,numberless}  
  {\normalfont\fontsize{14}{12}\bfseries\centering}  
  {}% Label (empty for unnumbered)  
  {0pt}% Horizontal separation  
  {}% Before-code  
% Math packages  
% \titlespacing*{\section}{0pt}{3.5ex plus 1ex minus .2ex}{2.3ex plus .2ex}  
% \titlespacing*{name=\section,numberless}{0pt}{3.5ex plus 1ex minus .2ex}{2.3ex plus .2ex}  
  
\titlespacing*{\section}{0pt}{*0}{*0} % {left}{before-sep}{after-sep}    
\titlespacing*{name=\section,numberless}{0pt}{*0}{*0}    
  
\usepackage{amsmath,amsfonts,amsthm}  
\usepackage{icomma}  
  
% Other packages  
\usepackage{accents}  
\usepackage{textcomp}  
\usepackage{pdfpages}  
\usepackage{hyperref}  
\hypersetup{  
    colorlinks,  
    citecolor=black,  
    filecolor=black,  
    linkcolor=black,  
    urlcolor=blue  
}  
  
% Layout  
\usepackage[left=3cm, right=1cm, top=2cm, bottom=2cm]{geometry}  
  
\usepackage{indentfirst}  
\setlength{\parindent}{1.25cm}  
  
\usepackage{setspace} % Для настройки межстрочного интервала  
\setstretch{1.5} % Устанавливает интервал 1.5  
  
% Microtypography  
\usepackage{microtype}  
  
% Fix for itemize bullets  
\usepackage{enumitem}  
\setlist[itemize]{    
  topsep=0pt,      % Space before the list    
  partopsep=0pt,   % Extra space in lists after a paragraph break    
  itemsep=0pt,     % Space between items    
  parsep=0pt,      % Space between paragraphs within an item    
  %leftmargin=*,    % Automatic left margin    
  label={\normalfont\textbullet} % Keep your bullet style    
}    
  
% Page numbering  
\usepackage{fancyhdr}  
\fancyhf{}  
\renewcommand{\headrulewidth}{0pt}  
\rfoot{\small\thepage}  
\pagestyle{fancy}  
  
% Preamble ended, now begins document. Note: compile with xelatex.  
\documentclass[a4paper, 14pt]{extarticle}  
\usepackage{preamble-word}  
  
\begin{document}  
  
\includepdf[pages={1}]{doc/722_Титульный-лист-2025.pdf}  
  
\includepdf[pages={1}]{doc/872_Задание-на-практику-2025.pdf}  
  
\includepdf[pages={1}, pagecommand={\thispagestyle{fancy}}]{doc/Введение-2025.pdf}  
  
\setlength{\emergencystretch}{3em} % Дополнительное растяжение пробелов  
\tolerance=2000 % Большая терпимость к неравномерности  
  
\section*{2. Расчёт некомпланарного перехода второго типа между двумя круговыми орбитами}  
    Начальные данные:  
    \begin{itemize}  
      \setlength\itemsep{0.5pt}  
      \item $R = 6371$ км -- средний радиус Земли;  
      \item $r_0 = 6571$ км -- радиус начальной орбиты;  
      \item $r_f = 8030$ км -- радиус конечной орбиты;  
      \item $\Delta i = \Delta i_1 + \Delta i_2 = 30\textdegree$ -- модуль разницы в наклонениях орбит к экватору;  
      \item $\textmu 398600,44$ км$^3/$c$^2$ -- гравитационный параметр Земли.  
    \end{itemize}  
    Вся скорость измеряется в метрах в секунду. На рис. 1 показан вектор $\Delta V_1 $ при переходе на эллиптическую траекторию, представляющий собой векторную разницу между требуемой скоростью КА в перицентре переходного эллипса и скорость КА на опорной круговой орбите. Модуль этого вектора определяется выражением  
\begin{equation}\label{1}  
  \Delta V_1 = \sqrt{V_0^2 + V_p^2 - 2V_0V_p \cos(\Delta i_1)}, \tag{1}  
\end{equation}  
  где $V_0 = \sqrt{\textmu/r_0} = 7,788$ -- скорость КА на опорной орбите; $V_p = \sqrt{\textmu/p}(1+e) = 8,168$ -- скорость КА в перицентре переходного эллипса, $e$ и $p$ -- эксцентриситет и фокальный параметр переходного эллипса. Величины $e$ и $p$ можно найти по формулам  
\[  
    e = \frac{r_f - r_0}{2a} = 0,1, \quad \quad p = a(1-e^2)=7,228\cdot 10^3  
,\]   
где $a = (r_0 + r_f)/2 = 7,301\cdot 10^3$ -- большая полуось переходного эллипса.  
  % где $\displaystyle V_0 = \sqrt{\frac{\mu}{r_0}} $ --- скорость КА на опорной орбите; \setstretch{2.5} $\displaystyle V_p = \sqrt{\mu/p}(1+e) $ -- скорость КА в перицентре переходного эллипса. \setstretch{1.5}  
  
% \linespread{1.5}\selectfont % для сохранения межстрочного интервала при переходе на новый абзац.  
По аналогии с формулой \eqref{1} модуль вектора $\Delta V_2$ может быть определён как   
\begin{equation}\label{2}  
  \Delta V_2 = \sqrt{V_f^2 + V_a^2 - 2V_f V_a \cos(\Delta i -\Delta i_1)}, \tag{2}  
\end{equation}  
  где $V_f = \sqrt{\textmu/r_f} = 7,045$ -- скорость КА на целевой круговой орбите; $V_a = \sqrt{\textmu/p}(1-e) = 8,168$ -- скорость КА в центре переходного эллипса. Оптимизируемая функция СХС имеет вид   
\begin{equation}\label{3}  
  \Delta V_\Sigma = \sqrt{V_0^2 + V_p^2 - 2V_0V_p \cos(\Delta i_1)} + \sqrt{V_f^2 + V_a^2 - 2V_f V_a \cos(\Delta i -\Delta i_1)}, \tag{3}  
\end{equation}  
\end{document}  
