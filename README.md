# Langkah-Langkah Membangun Ubuntu Buatan Lokal "BedillOS amd64" 

### Persyaratan


a. Gunakan Linux Debian / Ubuntu Untuk Menjalankan Perintah Ini 

b. Install VirtualBox dan debootstrap > ```apt install schroot debootstrap```

c. Login root dengan perintah > ```sudo su```

d. set HOME ke /root folder

e. Install Dependensi yang diperlukan > ```apt install mkisofs xorriso```

f. Buat folder baru di dalam root folder > ```mkdir /root/live-bedillos```

g. Memakai Xorriso versi 1.56 atau cek dengan perintah > ```xorriso -version ```

### Urutan Prosedur

__1 - Perintah pertama adalah mengunduh dan menyiapkan sistem minimal Ubuntu dengan debootstrap__

- ALERT: Jangan di jalankan jika kamu unduh archive format ```.tar```di Halaman Link Releases

- (nb: check Releases Page ada live-bedillos.tar)
```
sudo debootstrap \
   --arch=amd64 \
   --variant=minbase \
   xenial \
   $HOME/live-bedillos/chroot \
   http://us.archive.ubuntu.com/ubuntu/

```


__2 - Memasang sistem file dev dan run ke dalam lingkungan chroot agar perangkat keras dapat diakses__
```
sudo mount --bind /dev $HOME/live-bedillos/chroot/dev
sudo mount --bind /run $HOME/live-bedillos/chroot/run

```

__3 - Masuk ke lingkungan chroot untuk mulai mengkonfigurasi sistem__

```
sudo chroot $HOME/live-bedillos/chroot

```

__4 - Memasang beberapa sistem file virtual yang diperlukan untuk menjalankan sistem Linux di dalam chroot__
```
mount none -t proc /proc
mount none -t sysfs /sys
mount none -t devpts /dev/pts

```

__5 - Mengatur variabel lingkungan__
```
export HOME=/root
export LC_ALL=C

```


__6 - Mengubah nama host untuk sistem yang sedang dibangun__
```
echo "bedillos" > /etc/hostname

```


__7 - Menambahkan daftar repositori untuk apt__
```
cat <<EOF > /etc/apt/sources.list
deb http://us.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse
deb http://us.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe multiverse
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://us.archive.ubuntu.com/ubuntu/ xenial-updates main restricted universe multiverse
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial-updates main restricted universe multiverse
EOF

apt install software-properti* && add-apt-repository ppa:maarten-baert/simplescreenrecorder

```


__8 - Memperbarui daftar paket dan menginstal beberapa paket dasar__
```
sudo apt update
apt-get install -y libterm-readline-gnu-perl systemd-sysv

```

__9 - Mengatur ID mesin dan mengonfigurasi dbus__
```
dbus-uuidgen > /etc/machine-id
ln -fs /etc/machine-id /var/lib/dbus/machine-id

```

__10 - Mengalihkan initctl untuk memastikan sistem dapat berjalan tanpa masalah__
```
dpkg-divert --local --rename --add /sbin/initctl
ln -s /bin/true /sbin/initctl

```

__11 - Melakukan upgrade sistem dan menginstal beberapa paket tambahan__
```
apt install simplescreenrecorder dolphin
apt-get -y upgrade
apt-get install -y \
   sudo \
   ubuntu-standard \
   casper \
   discover \
   laptop-detect \
   os-prober \
   network-manager \
   net-tools \
   wireless-tools \
   wpagui \
   locales \
   grub-common \
   grub-gfxpayload-lists \
   grub-pc \
   grub-pc-bin \
   grub2-common \
   mtools \
   binutils

```



__12 - Menginstal paket untuk installer Ubuntu__
```
apt-get install -y \
   ubiquity \
   ubiquity-casper \
   ubiquity-frontend-gtk \
   ubiquity-slideshow-ubuntu \
   ubiquity-ubuntu-artwork dolphin

```


__13 - Menginstal tema untuk splash screen__
```
apt-get install -y plymouth-themes

```

__14 - Menginstal beberapa aplikasi dan utilitas__
```
apt-get install -y \
   clamav-daemon \
   terminator \
   apt-transport-https \
   curl \
   vim \
   nano \
   less

```


__15 - Memulai tool "tasksel" untuk memilih dan menginstal lingkungan desktop (Lubuntu)__
```
tasksel

```



__16 - Menginstal paket untuk LXDE dan lingkungan desktop ringan__

```
apt-get install -y lxde-core openbox* lxpanel* pcmanfm lxsession* lxappearance* lxterminal lxrandr lxinput lximage-qt gpicview lightdm* xserver-xorg feh compton gnome-screenshot synaptic tasksel gedit network-manager network-manager-gnome ifupdown nmcli wicd net-tools inetutils-ping curl wget traceroute nmap dnsutils openssh-client openssh-server nano vim gedit pcmanfm thunar vlc mpv audacious ffmpeg firefox chromium-browser midori zip unzip tar gzip bzip2 xz-utils libreoffice evince catfish htop gparted gksu ufw rsync deja-dup

```

__17 - Menginstal aplikasi tambahan seperti kdenlive dan phpmyadmin__
```
apt install kdenlive php php-mbstring libapache2-mod-php apache2 mariadb-server lxde

apt install phpmyadmin firefox snapd snap 

apt install screenfetch blueman bluetooth* pulseaudio* bluez blueman

apt install adb build-essential autoconf libx11-dev libxext-dev libxrender-dev libxrandr-dev libxinerama-dev libxi-dev libxft-dev libgl1-mesa-dev libegl1-mesa-dev


```
__18 - Menghapus sesi GNOME yang tidak diinginkan__
```
apt remove openbox-gnome-session 

```

__19 - Menghapus paket yang tidak diperlukan lagi__
```
apt autoremove
```
__20 - Menginstal alat pengembangan dan paket lain yang diperlukan__
```
apt install build-essential gdb lcov pkg-config libbz2-dev libffi-dev libgdbm-dev liblzma-dev libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev lzma lzma-dev tk-dev uuid-dev zlib1g-dev

add-apt-repository ppa:alexlarsson/flatpak

add-apt-repository ppa:<ISI DENGAN FIREFOX UBUNTU 16>

apt update

apt install print-manager

apt install flatpak nestopia gnome-chess

snap install ppsspp-emu

```
__21 - Menginstal LibreOffice dan alat Python__
```
apt install libreoffice ant python3-pip python3-venv python3 
```
__22 - Menginstal Java dan Screen Mirroring__
```
flatpak install flathub org.gnome.NetworkDisplays

apt-get install -y openjdk-8-jdk openjdk-8-jre
```
__23 - Mengonfigurasi NetworkManager__
```
dpkg-reconfigure locales

cat <<EOF > /etc/NetworkManager/NetworkManager.conf
[main]
rc-manager=none
plugins=ifupdown,keyfile
dns=systemd-resolved
[ifupdown]
managed=false
EOF

dpkg-reconfigure network-manager
```
__24 - Membuat direktori untuk image ISO__
```
mkdir -p /image/{casper,isolinux,install}
```
__25 - Menyalin kernel dan initrd ke image ISO__
```
cp /boot/vmlinuz-**-**-generic /image/casper/vmlinuz

cp /boot/initrd.img-**-**-generic /image/casper/initrd
```
__26 - Mengunduh dan menyiapkan memtest86+ untuk memeriksa RAM__
```
wget --progress=dot https://memtest.org/download/v7.00/mt86plus_7.00.binaries.zip -O /image/install/memtest86.zip

unzip -p /image/install/memtest86.zip memtest64.bin > /image/install/memtest86+.bin

unzip -p /image/install/memtest86.zip memtest64.efi > /image/install/memtest86+.efi

rm -f /image/install/memtest86.zip
```
__27 - Menambahkan file sistem informasi__
```
touch /image/ubuntu
```
__28 - Mengonfigurasi grub untuk memulai proses boot__
```
cat <<EOF > /image/isolinux/grub.cfg
search --set=root --file /ubuntu
insmod all_video
set default="0"
set timeout=5
menuentry "Try Ubuntu FS without installing" {
   linux /casper/vmlinuz boot=casper nopersistent toram quiet splash ---
   initrd /casper/initrd
}
menuentry "Install Ubuntu FS" {
   linux /casper/vmlinuz boot=casper quiet splash ---
   initrd /casper/initrd
}

menuentry "Check disc for defects" {
   linux /casper/vmlinuz boot=casper integrity-check quiet splash ---
   initrd /casper/initrd
}

grub_platform

if [ "\$grub_platform" = "efi" ]; then
menuentry 'UEFI Firmware Settings' {
   fwsetup
}

menuentry "Test memory Memtest86+ (UEFI)" {
   linux /install/memtest86+.efi
}

else

menuentry "Test memory Memtest86+ (BIOS)" {
   linux16 /install/memtest86+.bin
}

fi
EOF
```
__29 - Membuat manifest untuk paket yang telah diinstal__
```
dpkg-query -W --showformat='${Package} ${Version}\n' | sudo tee /image/casper/filesystem.manifest
```
__30 - Menyalin manifest untuk desktop__
```
cp -v /image/casper/filesystem.manifest image/casper/filesystem.manifest-desktop
```
__31 - Menghapus beberapa entri yang tidak diperlukan dari manifest__
```
sed -i '/ubiquity/d' /image/casper/filesystem.manifest-desktop

sed -i '/casper/d' /image/casper/filesystem.manifest-desktop

sed -i '/discover/d' /image/casper/filesystem.manifest-desktop

sed -i '/laptop-detect/d' /image/casper/filesystem.manifest-desktop

sed -i '/os-prober/d' /image/casper/filesystem.manifest-desktop
```
__32 - Menambahkan file README untuk definisi disk__
```
cat <<EOF > /image/README.diskdefines

#define DISKNAME  Ubuntu from scratch

#define TYPE  binary

#define TYPEbinary  1

#define ARCH  amd64

#define ARCHamd64  1

#define DISKNUM  1

#define DISKNUM1  1

#define TOTALNUM  0

#define TOTALNUM0  1

EOF
```
__33 - Menyalin file bootloader dan konfigurasi untuk ISO__
```
cd /image

cp /usr/lib/shim/shimx64.efi.signed isolinux/bootx64.efi

cp /usr/lib/shim/mmx64.efi isolinux/mmx64.efi

cp /usr/lib/grub/x86_64-efi-signed/grubx64.efi.signed isolinux/grubx64.efi
```

__34 - Pastekan Perintah ini di Terminal__
```
(
   cd isolinux && \  
   dd if=/dev/zero of=efiboot.img bs=1M count=10 && \
   mkfs.vfat -F 16 efiboot.img && \
   LC_CTYPE=C mmd -i efiboot.img efi efi/ubuntu efi/boot && \
   LC_CTYPE=C mcopy -i efiboot.img ./bootx64.efi ::efi/boot/bootx64.efi && \
   LC_CTYPE=C mcopy -i efiboot.img ./mmx64.efi ::efi/boot/mmx64.efi && \
   LC_CTYPE=C mcopy -i efiboot.img ./grubx64.efi ::efi/boot/grubx64.efi && \
   LC_CTYPE=C mcopy -i efiboot.img ./grub.cfg ::efi/ubuntu/grub.cfg
)


grub-mkstandalone \
   --format=i386-pc \
   --output=isolinux/core.img \
   --install-modules="linux16 linux normal iso9660 biosdisk memdisk search tar ls" \
   --modules="linux16 linux normal iso9660 biosdisk search" \
   --locales="" \
   --fonts="" \
   "boot/grub/grub.cfg=isolinux/grub.cfg"

cat /usr/lib/grub/i386-pc/cdboot.img isolinux/core.img > isolinux/bios.img



/bin/bash -c "(find . -type f -print0 | xargs -0 md5sum | grep -v -e 'isolinux' > md5sum.txt)"


truncate -s 0 /etc/machine-id


rm /sbin/initctl

dpkg-divert --rename --remove /sbin/initctl

apt-get clean

rm -rf /tmp/* ~/.bash_history

umount /proc

umount /sys

umount /dev/pts

export HISTSIZE=0

exit

sudo umount $HOME/live-bedillos/chroot/dev

sudo umount $HOME/live-bedillos/chroot/run
```

__35 - Pergi ke folder live-bedillos__
```
cd $HOME/live-bedillos
```
__36 - memindahkan folder komfigurasi image dari chroot ke live-bedillos__
```
sudo mv chroot/image .
```
__37 - Membuat Live File System untuk ISO File__
```
sudo mksquashfs chroot image/casper/filesystem.squashfs \
   -noappend -no-duplicates -no-recovery \
   -wildcards \
   -comp xz -b 1M -Xdict-size 100% \
   -e "var/cache/apt/archives/*" \
   -e "root/*" \
   -e "root/.*" \
   -e "tmp/*" \
   -e "tmp/.*" \
   -e "swapfile"
```
__38 - Mencetak dan Menulis Output Chroot ke folder image/casper ke Filesystem.size__
```
printf $(sudo du -sx --block-size=1 chroot | cut -f1) | sudo tee image/casper/filesystem.size
```

__39 - Pergi ke Folder Image__
```
cd $HOME/live-bedillos/image
```

__40 - Menggunakan xorriso 1.56 untuk membuat file ISO yang dapat boot__
```
sudo xorriso \
   -as mkisofs \
   -iso-level 3 \
   -full-iso9660-filenames \
   -J -J -joliet-long \
   -volid "Ubuntu from scratch" \
   -output "../ubuntu-from-scratch.iso" \
   -eltorito-boot isolinux/bios.img \
     -no-emul-boot \
     -boot-load-size 4 \
     -boot-info-table \
     --eltorito-catalog boot.catalog \
     --grub2-boot-info \
     --grub2-mbr ../chroot/usr/lib/grub/i386-pc/boot_hybrid.img \
     -partition_offset 16 \
     --mbr-force-bootable \
   -eltorito-alt-boot \
     -no-emul-boot \
     -e isolinux/efiboot.img \
     -append_partition 2 28732ac11ff8d211ba4b00a0c93ec93b isolinux/efiboot.img \
     -appended_part_as_gpt \
     -iso_mbr_part_type a2a0d0ebe5b9334487c068b6b72699c7 \
     -m "isolinux/efiboot.img" \
     -m "isolinux/bios.img" \
     -e '--interval:appended_partition_2:::' \
   -exclude isolinux \
   -graft-points \
      "/EFI/boot/bootx64.efi=isolinux/bootx64.efi" \
      "/EFI/boot/mmx64.efi=isolinux/mmx64.efi" \
      "/EFI/boot/grubx64.efi=isolinux/grubx64.efi" \
      "/EFI/ubuntu/grub.cfg=isolinux/grub.cfg" \
      "/isolinux/bios.img=isolinux/bios.img" \
      "/isolinux/efiboot.img=isolinux/efiboot.img" \
      "."



```

untuk Ubuntu 16 Xorriso < 1.56

```
sudo xorriso -as mkisofs -iso-level 3 -full-iso9660-filenames -J -J -joliet-long -volid "Bedillos" -output "$HOME/live-bedillos/bedillos.iso" -eltorito-boot isolinux/bios.img -no-emul-boot -boot-load-size 4 -boot-info-table --eltorito-catalog boot.catalog --grub2-boot-info --grub2-mbr /image/usr/lib/grub/i386-pc/boot_hybrid.img -partition_offset 16 -eltorito-alt-boot -no-emul-boot -e isolinux/efiboot.img -append_partition 2 0xEF isolinux/efiboot.img -appended_part_as_gpt -m "isolinux/efiboot.img" -m "isolinux/bios.img" -exclude isolinux -graft-points "/EFI/boot/bootx64.efi=isolinux/bootx64.efi" "/EFI/boot/mmx64.efi=isolinux/mmx64.efi" "/EFI/boot/grubx64.efi=isolinux/grubx64.efi" "/EFI/ubuntu/grub.cfg=isolinux/grub.cfg" .

```
__** 00 - Setelah proses pembuatan ISO selesai, Anda dapat keluar dari chroot__
```
exit
```

__40 - Alternatif proses pembuatan ISO dan file yang dapat dibooting.__
```
cat <<EOF> isolinux/isolinux.cfg
UI vesamenu.c32

MENU TITLE Boot Menu
DEFAULT linux
TIMEOUT 600
MENU RESOLUTION 640 480
MENU COLOR border       30;44   #40ffffff #a0000000 std
MENU COLOR title        1;36;44 #9033ccff #a0000000 std
MENU COLOR sel          7;37;40 #e0ffffff #20ffffff all
MENU COLOR unsel        37;44   #50ffffff #a0000000 std
MENU COLOR help         37;40   #c0ffffff #a0000000 std
MENU COLOR timeout_msg  37;40   #80ffffff #00000000 std
MENU COLOR timeout      1;37;40 #c0ffffff #00000000 std
MENU COLOR msg07        37;40   #90ffffff #a0000000 std
MENU COLOR tabmsg       31;40   #30ffffff #00000000 std

LABEL linux
 MENU LABEL Try Ubuntu FS
 MENU DEFAULT
 KERNEL /casper/vmlinuz
 APPEND initrd=/casper/initrd boot=casper

LABEL linux
 MENU LABEL Try Ubuntu FS (nomodeset)
 MENU DEFAULT
 KERNEL /casper/vmlinuz
 APPEND initrd=/casper/initrd boot=casper nomodeset
EOF


apt install -y syslinux-common && \
cp /usr/lib/ISOLINUX/isolinux.bin image/isolinux/ && \
cp /usr/lib/syslinux/modules/bios/* image/isolinux/


cd $HOME/live-bedillos/image

sudo xorriso \
   -as mkisofs \
   -iso-level 3 \
   -full-iso9660-filenames \
   -J -J -joliet-long \
   -volid "Ubuntu from scratch" \
   -output "../ubuntu-from-scratch.iso" \
 -isohybrid-mbr /usr/lib/ISOLINUX/isohdpfx.bin \
 -eltorito-boot \
     isolinux/isolinux.bin \
     -no-emul-boot \
     -boot-load-size 4 \
     -boot-info-table \
     --eltorito-catalog isolinux/isolinux.cat \
 -eltorito-alt-boot \
     -e /EFI/boot/efiboot.img \
     -no-emul-boot \
     -isohybrid-gpt-basdat \
 -append_partition 2 0xef EFI/boot/efiboot.img \
   "$HOME/live-ubuntu-from-scratch/image"

```


### 45 - For installation

Still in image/ folder. and plug USB DRIVE / FLASHDISK 

Type
``` umount /dev/sdX1 ``` > change X with your Flashdisk in output ```lsblk```
 example ```/dev/sdb1```

Type
``` dd if=../ubuntu-from-scratch.iso of=/dev/sdb status=progress```

```reboot```

after reboot power on, press F9 for HP1000 Computer or F12 or search your Boot option Computer Key in Google Search.

Choose your Flashdisk with ```arrow up``` or ```arrow down``` key and ```ENTER```

AFTER LOGIN SCREEN PASSWROD ```ubuntu```

open Teminal 

Type ```ubiquity``` in terminal


after run make sure you not checklist this option and not connect any network

![IMG-20250414-WA0014](https://github.com/user-attachments/assets/81816515-23e1-4d25-aa8d-0798587e5bd2)
