# README - Membangun BedillOS ISO di Fedora (Tanpa Install ARM64, Kompatibel untuk Emulasi Android)

## Persyaratan (Fedora)

### a. Sistem

Gunakan Fedora Linux (minimal Fedora 38 ke atas)

### b. Instalasi alat penting

```bash
sudo dnf install debootstrap qemu-user-static systemd-container xorriso squashfs-tools dosfstools mtools grub2-tools xz syslinux zip unzip wget curl git nano -y
```

### c. Verifikasi emulator ARM64

```bash
cat /proc/sys/fs/binfmt_misc/qemu-aarch64
```

Jika output menunjukkan `enabled`, maka Fedora sudah siap melakukan chroot ARM64.

## Struktur Folder

```bash
export HOME=/root
mkdir -p /root/live-bedillos
cd /root/live-bedillos
```

---

## Tahap 1 - Bootstrap Ubuntu ARM64 (tanpa install arm64 native)

```bash
debootstrap --arch=arm64 --foreign jammy chroot http://ports.ubuntu.com/
cp /usr/bin/qemu-aarch64-static chroot/usr/bin/
chroot chroot /debootstrap/debootstrap --second-stage
```

## Tahap 2 - Persiapan Chroot

```bash
mount --bind /dev chroot/dev
mount --bind /run chroot/run
chroot chroot
```

## Tahap 3 - Setup Sistem Dasar

### Tambahkan Firefox dari PPA jika versi default sudah kadaluarsa

```bash
add-apt-repository ppa:mozillateam/ppa
apt update
apt install -y firefox

# Pastikan prioritas PPA lebih tinggi dari default
cat <<EOF > /etc/apt/preferences.d/mozillateam
Package: *
Pin: release o=LP-PPA-mozillateam
Pin-Priority: 1001
EOF

# Tambahkan GPG key untuk PPA jika diperlukan
apt install -y gnupg
keyserver=hkp://keyserver.ubuntu.com:80
gpg --keyserver "$keyserver" --recv-keys 3B4FE6ACC0B21F32

# Tambahkan ke apt trusted.gpg.d
gpg --export 3B4FE6ACC0B21F32 | tee /etc/apt/trusted.gpg.d/mozillateam.gpg > /dev/null
```

```bash
mount -t proc none /proc
mount -t sysfs none /sys
mount -t devpts none /dev/pts

export HOME=/root
export LC_ALL=C

echo "bedillos" > /etc/hostname

cat <<EOF > /etc/apt/sources.list
deb http://ports.ubuntu.com/ jammy main universe multiverse restricted
deb http://ports.ubuntu.com/ jammy-updates main universe multiverse restricted
deb http://ports.ubuntu.com/ jammy-security main universe multiverse restricted
EOF

apt update && apt install -y sudo nano systemd-sysv dbus htop network-manager net-tools wireless-tools locales openssh-server curl unzip zip xfce4-terminal lightdm firefox

# Install LXDE desktop
apt install -y lxde-core lxappearance pcmanfm openbox lxdm xserver-xorg

# Aktifkan LXDE saat boot
echo "exec startlxde" > /etc/skel/.xsession
chmod +x /etc/skel/.xsession

# Tambahkan user
useradd -m -s /bin/bash bedill
passwd bedill
usermod -aG sudo,adm,audio,video,netdev bedill
```

## Tahap 4 - Bersihkan dan Keluar dari Chroot

```bash
apt clean
rm -rf /tmp/* ~/.bash_history
umount /proc
umount /sys
umount /dev/pts
exit
umount chroot/dev
umount chroot/run
```

## Tahap 5 - Pembuatan ISO

### Struktur Image

```bash
mkdir -p image/{casper,isolinux,install}
cp chroot/boot/vmlinuz-* image/casper/vmlinuz
cp chroot/boot/initrd.img-* image/casper/initrd
```

### Buat file squashfs

```bash
mksquashfs chroot image/casper/filesystem.squashfs -e boot
```

### Buat manifest dan ukuran sistem

```bash
dpkg-query -W --root=chroot --showformat='${Package} ${Version}\n' > image/casper/filesystem.manifest
printf $(du -sx --block-size=1 chroot | cut -f1) > image/casper/filesystem.size
```

### Tambahkan file pendukung GRUB

```bash
touch image/ubuntu
cat <<EOF > image/isolinux/grub.cfg
search --set=root --file /ubuntu
set default=0
timeout=5
menuentry "Try BedillOS LXDE" {
 linux /casper/vmlinuz boot=casper quiet splash ---
 initrd /casper/initrd
}
EOF
```

## Tahap 6 - Buat EFI Boot Image

```bash
cd image/isolinux

dd if=/dev/zero of=efiboot.img bs=1M count=10
mkfs.vfat efiboot.img
mmd -i efiboot.img efi efi/boot
mcopy -i efiboot.img /usr/lib/grub/arm64-efi/bootaa64.efi ::efi/boot/bootaa64.efi
```

## Tahap 7 - Buat ISO (Kompatibel untuk emulasi Android)

```bash
cd ../
xorriso -as mkisofs \
  -iso-level 3 \
  -full-iso9660-filenames \
  -volid "BedillOS ARM64" \
  -output "../bedillos-arm64.iso" \
  -eltorito-boot isolinux/efiboot.img \
  -no-emul-boot \
  -boot-load-size 4 \
  -boot-info-table \
  -eltorito-alt-boot \
  -e isolinux/efiboot.img \
  -no-emul-boot \
  -isohybrid-gpt-basdat \
  -append_partition 2 0xef isolinux/efiboot.img \
  -graft-points \
   "/EFI/boot/bootaa64.efi=isolinux/efiboot.img" \
   "."
```

## Flash ke USB (opsional)

```bash
sudo dd if=../bedillos-arm64.iso of=/dev/sdX status=progress
```

---

## Menjalankan ISO di Android (Linux Machine atau Termux)

1. **Salin ISO ke Android**, ekstrak jika perlu ke IMG.
2. **Gunakan Linux Machine**, Andronix, atau Termux:

   * Jalankan dengan QEMU:

     ```bash
     qemu-system-aarch64 \
       -m 1400 \
       -cpu cortex-a72 \
       -M virt \
       -kernel vmlinuz \
       -initrd initrd.img \
       -append "root=/dev/vda console=ttyAMA0" \
       -drive file=filesystem.img,format=raw,if=virtio \
       -device virtio-keyboard-device \
       -device virtio-mouse-device \
       -netdev user,id=mynet0 \
       -device virtio-net-device,netdev=mynet0 \
       -vnc :1
     ```
   * Gunakan aplikasi **VNC Viewer** untuk akses grafis:

     * Host: `localhost`
     * Port: `5901`
3. Setelah login, buka LXTerminal dan jalankan:

   ```bash
   firefox
   ```

   untuk mengakses WhatsApp Web.

---

## Catatan Penting

* ISO ini **tidak untuk di-boot langsung di Android**, melainkan untuk **dijalankan via emulasi**.
* Untuk jalankan langsung di Termux: bisa dikonversi rootfs ke `.tar.gz` dan jalankan pakai `proot-distro`.

---

**Dikustomisasi khusus untuk Fedora + target HP Android via emulator (Linux Machine / Termux) + kebutuhan Web WhatsApp (Firefox)**
