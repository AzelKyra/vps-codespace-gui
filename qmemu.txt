#!/bin/bash

echo "Đang chuẩn bị môi trường VM cho Windows Server 2012 R2..."
SPICE_PORT=5924
ISO_URL="https://archive.org/download/en_windows_server_2012_r2_x64_dvd_27079461/en_windows_server_2012_r2_x64_dvd_2707946.iso"
ISO_PATH="/mnt/win2012r2.iso"
QCOW2_PATH="/mnt/win2012r2.qcow2"

# Cài các gói cần thiết
sudo apt update
sudo apt install -y qemu-kvm cpulimit wget

# Mount phân vùng nếu cần
if ! mount | grep -q "on /mnt "; then
    echo "Đang mount phân vùng lớn hơn 500GB vào /mnt..."
    partition=$(lsblk -b --output NAME,SIZE,MOUNTPOINT | awk '$2 > 500000000000 && $3 == "" {print $1}' | head -n 1)
    sudo mount "/dev/${partition}1" /mnt
fi

# Tải ISO nếu chưa có
if [ ! -f "$ISO_PATH" ]; then
    echo "Đang tải ISO Win Server 2012 R2..."
    wget -O "$ISO_PATH" "$ISO_URL"
fi

# Tạo ổ cứng qcow2 nếu chưa có
if [ ! -f "$QCOW2_PATH" ]; then
    echo "Tạo ổ đĩa win2012r2.qcow2..."
    qemu-img create -f qcow2 "$QCOW2_PATH" 40G
fi

# Khởi chạy cài đặt máy ảo
echo "Đang khởi chạy VM để cài đặt..."
sudo cpulimit -l 80 -- sudo kvm \
    -m 4096 \
    -smp 2 \
    -cpu host \
    -enable-kvm \
    -drive file="$QCOW2_PATH",format=qcow2 \
    -cdrom "$ISO_PATH" \
    -boot d \
    -net nic -net user,hostfwd=tcp::3389-:3389 \
    -vga virtio \
    -soundhw hda \
    -device usb-tablet \
    -spice port=${SPICE_PORT},disable-ticketing \
    -device virtio-serial-pci \
    -chardev spicevmc,id=vdagent,name=vdagent \
    -device virtserialport,chardev=vdagent,name=com.redhat.spice.0
