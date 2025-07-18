# README - Membangun BedillOS ISO untuk amd64 (Kompatibel untuk PC dan Virtual Machine)

## Persyaratan Sistem

### a. Distro Host

* Gunakan **Ubuntu** atau **Debian** (lebih direkomendasikan), atau Fedora jika sudah memahami cara menyesuaikan.

### b. Install Tools

```bash
sudo apt update
sudo apt install -y debootstrap squashfs-tools xorriso grub-pc-bin grub-efi-amd64-bin mtools dosfstools
```

## Tahap 1 - Siapkan Struktur Folder

```bash
export HOME=/root
mkdir -p $HOME/live-bedillos-amd64/chroot
cd $HOME/live-bedillos-amd64
```

## Tahap 2 - Bootstrap Ubuntu Minimal

```bash
sudo debootstrap --arch=amd64 --variant=minbase jammy chroot http://archive.ubuntu.com/ubuntu/
```

## Tahap 3 - Masuk ke Chroot

```bash
sudo mount --bind /dev chroot/dev
sudo mount --bind /run chroot/run
sudo chroot chroot
```

## Tahap 4 - Konfigurasi Dasar Sistem

```bash
mount -t proc proc /proc
mount -t sysfs sys /sys
mount -t devpts devpts /dev/pts

export HOME=/root
export LC_ALL=C

echo "bedillos" > /etc/hostname
```

## Tahap 5 - Tambahkan Repo & Instalasi Paket

```bash
cat <<EOF > /etc/apt/sources.list
deb http://archive.ubuntu.com/ubuntu/ jammy main universe multiverse restricted
deb http://archive.ubuntu.com/ubuntu/ jammy-updates main universe multiverse restricted
deb http://archive.ubuntu.com/ubuntu/ jammy-security main universe multiverse restricted
EOF

apt update
apt install -y sudo lxde-core lightdm firefox network-manager net-tools wireless-tools unzip curl

# Buat user
useradd -m -s /bin/bash bedill
passwd bedill
usermod -aG sudo,adm,audio,video,netdev bedill
```

## Tahap 6 - Bersihkan dan Keluar

```bash
apt clean
umount /proc /sys /dev/pts
exit
sudo umount chroot/dev
sudo umount chroot/run
```

## Tahap 7 - Siapkan Struktur ISO

```bash
mkdir -p image/{casper,isolinux,install}
cp chroot/boot/vmlinuz-* image/casper/vmlinuz
cp chroot/boot/initrd.img-* image/casper/initrd
```

## Tahap 8 - Buat squashfs

```bash
mksquashfs chroot image/casper/filesystem.squashfs -e boot
```

## Tahap 9 - Buat Manifest dan GRUB Config

```bash
dpkg-query -W --root=chroot --showformat='${Package} ${Version}\n' > image/casper/filesystem.manifest
printf $(du -sx --block-size=1 chroot | cut -f1) > image/casper/filesystem.size

cat <<EOF > image/isolinux/grub.cfg
search --set=root --file /ubuntu
set default=0
timeout=5
menuentry "Try BedillOS Live LXDE" {
 linux /casper/vmlinuz boot=casper quiet splash ---
 initrd /casper/initrd
}
EOF

touch image/ubuntu
```

## Tahap 10 - Buat EFI dan BIOS Boot Image

```bash
cd image/isolinux

# EFI
dd if=/dev/zero of=efiboot.img bs=1M count=10
mkfs.vfat efiboot.img
mmd -i efiboot.img efi efi/boot
mcopy -i efiboot.img /usr/lib/grub/x86_64-efi/bootx64.efi ::efi/boot/bootx64.efi

# BIOS GRUB
grub-mkstandalone \
 --format=i386-pc \
 --output=core.img \
 --install-modules="linux16 linux normal iso9660 biosdisk search" \
 "boot/grub/grub.cfg=grub.cfg"

cat /usr/lib/grub/i386-pc/cdboot.img core.img > bios.img
```

## Tahap 11 - Build ISO Final

```bash
cd ../
xorriso -as mkisofs \
 -iso-level 3 \
 -full-iso9660-filenames \
 -volid "BedillOS AMD64" \
 -output "../bedillos-amd64.iso" \
 -eltorito-boot isolinux/bios.img \
   -no-emul-boot \
   -boot-load-size 4 \
   -boot-info-table \
 -eltorito-alt-boot \
   -e isolinux/efiboot.img \
   -no-emul-boot \
   -isohybrid-gpt-basdat \
   -append_partition 2 0xef isolinux/efiboot.img \
   -graft-points \
     "/EFI/boot/bootx64.efi=isolinux/efiboot.img" \
     "."
```

## Estimasi Ukuran ISO

* Ukuran akhir ISO: **700–900 MB**

## Flash ke USB

```bash
sudo dd if=../bedillos-amd64.iso of=/dev/sdX bs=4M status=progress && sync
```

---

**BedillOS amd64 siap digunakan di PC, laptop, atau VM. Berisi LXDE dan Firefox siap pakai untuk WhatsApp Web atau kegiatan ringan lainnya.**
