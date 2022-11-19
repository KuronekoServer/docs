# ArchLinux iSCSI boot with iBFT 
iBFT による ArchLinux の iSCSI ブート 

```
#
#
#   ISCSI Boot
#
#   [Installer]
#
#
loadkeys jp106
pacman -Sy --noconfirm
pacman -S open-iscsi --noconfirm
systemctl enable --now iscsid
echo "InitiatorName=iqn.1986-03.com.hp:openstack-0" > /etc/iscsi/initiatorname.iscsi
systemctl restart iscsid
iscsiadm -m discovery -t sendtargets -p 192.168.11.141
iscsiadm -m node -T iqn.1991-05.com.microsoft:openstack-0-target -p 192.168.11.141 -l
fdisk /dev/sdb
# -> g -> n -> -> -> +512M (-> Y ) -> n -> -> -> -> (-> Y ) -> w
mkfs.xfs /dev/sdb2
mkfs.fat -F 32 /dev/sdb1
mount /dev/sdb2 /mnt
mkdir /mnt/boot
mount /dev/sdb1 /mnt/boot
pacstrap /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr nano networkmanager open-iscsi openssh
genfstab /mnt
genfstab /mnt > /mnt/etc/fstab
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
nano /etc/locale.gen
    #ja_JP.UTF-8 UTF-8 #<- uncomment
locale-gen
nano /etc/locale.conf
    #LANG=ja_JP.UTF-8
nano /etc/vconsole.conf
    #KEYMAP=jp106
nano /etc/hostname
    #localhost
passwd
useradd -m -G wheel -s /bin/bash archuser
passwd archuser
EDITOR=nano visudo
    #%wheel ALL=(ALL) ALL #<- uncomment
systemctl enable NetworkManager
cat <<EOF > /usr/lib/initcpio/install/iscsi
build ()
{
    local mod
    for mod in iscsi_ibft iscsi_tcp libiscsi libiscsi_tcp scsi_transport_iscsi crc32c; do
        add_module "$mod"
    done

    add_checked_modules "/drivers/net"
    add_binary "/usr/bin/iscsistart"
    add_runscript
}

help ()
{
    cat <<HELPEOF
    This hook allows you to boot from an iSCSI target.
HELPEOF
}
EOF

cat <<EOF > /usr/lib/initcpio/hooks/iscsi
run_hook ()
{
    modprobe iscsi_tcp
    modprobe iscsi_ibft
    echo "Network configuration based on iBFT"
    iscsistart -N || echo "Unable to configure network"
    echo "Waiting 5 seconds..."
    sleep 5
    echo "iSCSI auto connect based on iBFT"
    until iscsistart -b ; do
        sleep 3
    done
}
EOF

nano /etc/mkinitcpio.conf
    #・・・
    #HOOKS=(... iscsi block ...)
    #・・・
pacman-key --init
pacman-key --populate archlinux
pacman -Sy xfsprogs intel-ucode --noconfirm
nano /etc/iscsi/iscsid.conf
    #node.startup = automatic
    #node.session.timeo.replacement_timeout = 86400
    #node.conn[0].timeo.noop_out_interval = 0
    #node.conn[0].timeo.noop_out_timeout = 0

mkinitcpio -p linux
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
exit

umount -a
reboot
```

`/etc/initcpio/install/iscsi`

```
sh
build ()
{
        local mod
        for mod in iscsi_tcp iscsi_ibft libiscsi libiscsi_tcp scsi_transport_iscsi crc32c; do
                add_module "$mod"
        done

        add_checked_modules "/drivers/net"
        add_binary iscsistart
        add_runscript
}

help ()
{
cat <<HELPEOF
        This hook allows you to boot from an iSCSI target.
HELPEOF
}
```

`/etc/initcpio/hooks/iscsi`

```
sh
run_hook ()
{
    modprobe iscsi_tcp
    modprobe iscsi_ibft

    echo "Network configuration based on iBFT"
    iscsistart -N {{|}}{{|}} echo "Unable to configure network"

    echo "Waiting 5 seconds..."
    sleep 5

    echo "iSCSI auto connect based on iBFT"
    until iscsistart -b ; do
        sleep 3
    done
}
```
