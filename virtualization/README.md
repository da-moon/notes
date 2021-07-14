# virtualization

## kvm

- validate hypervisor

```bash
virt-host-validate
```

- kernel modules for virtiofs

```bash
# [ NOTE ] => https://gist.github.com/TomFaulkner/389e8e2e9525e11afe2e775355954cdf
kernel_modules=();
kernel_modules+=("virtio-net")
kernel_modules+=("virtio-blk")
kernel_modules+=("virtio-scsi")
kernel_modules+=("virtio-balloon")
kernel_modules+=("vfio-pci")
if [ ${#kernel_modules[@]} -ne 0 ]; then
  for mod in "${kernel_modules[@]}"; do
    echo >&2 "*** removing any exisiting '${mod}' kernel module autoload value.";
    [ -r /etc/modules ] && sudo sed -i "/$mod/d" /etc/modules
    if [ -d /etc/modules-load.d ]; then
      find /etc/modules-load.d -type f -name '*.conf' \
      | xargs -r sudo sed -i "/$mod/d" ;
      sudo find /etc/modules-load.d -type f -empty -delete ;
    fi
  done
  for mod in "${kernel_modules[@]}"; do
    echo >&2 "*** ensuring '${mod}' kernel module is loaded.";
    sudo modprobe "${mod}"
    echo "${mod}" | sudo tee "/etc/modules-load.d/${mod}.conf" > /dev/null ;
  done
fi
# [ NOTE ] => https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Setting_up_IOMMU
export GRUB_CMDLINE_LINUX_DEFAULT="iommu=pt intel_iommu=on"
sudo grub-mkconfig -o /boot/grub/grub.cfg
# sudo dmesg | grep -i -e DMAR -e IOMMU
```

- hugepages setup for virtio-fs

```bash
sudo sed -i -e '/hugetlbfs/d' /etc/fstab ;
echo "hugetlbfs /dev/hugepages hugetlbfs mode=01770,gid=$(getent group kvm | awk -F: '{print $3}') 0 0" \
 | sudo tee -a /etc/fstab > /dev/null
sudo umount /dev/hugepages || true
sudo mount /dev/hugepages
echo 550 | sudo tee /proc/sys/vm/nr_hugepages > /dev/null
echo 'vm.nr_hugepages = nr_hugepages' | sudo tee /etc/sysctl.d/40-hugepage.conf > /dev/null
```

- create vm based on example xml

```bash
# [ NOTE ] =>
# - https://www.reddit.com/r/VFIO/comments/i12uyn/virtiofs_is_amazing_plus_how_i_set_it_up/
sudo sed -i \
  -e '/\/mnt\/user/d' \
  /etc/fstab ;
(
  echo 'user /mnt/user virtiofs rw,noatime,_netdev 0 2';
) | sudo tee -a /etc/fstab > /dev/null
sudo virsh allocpages 2M 1024 ;
rm -rf /tmp/kvm-example ;
mkdir -p /tmp/kvm-example ;
curl -fsSL https://pastebin.com/raw/ijWFDr6K -o /tmp/kvm-example/manjaro.xml ;
[ -r /usr/lib/qemu/virtiofsd ] \
&& sed -i \
  -e 's/\/usr\/bin\/virtiofsd/\/usr\/lib\/qemu\/virtiofsd/g' \
  /tmp/kvm-example/manjaro.xml ;
export LIBVIRT_DEFAULT_URI="qemu:///system"
# https://raw.githubusercontent.com/Cloudxtreme/debian-unattended/master/debian-unattended.sh
# git clone https://github.com/Cloudxtreme/debian-unattended
sudo systemctl enable --now libvirtd > /dev/null 2>&1
sudo systemctl is-active libvirtd > /dev/null 2>&1 && virsh create /tmp/kvm-example/manjaro.xml ;
```
