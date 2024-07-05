# Install Ubuntu Arm64 by QEMU on Windows

## Install QEMU

- link: [https://www.qemu.org/download/#windows](https://qemu.weilnetz.de/w64/)
- Install file .exe

## Create necessary support files (command on linux)

- Install QEMU on Ubuntu:

```bash
sudo apt install qemu-system-arm
```

- Create a VM-specific flash volume for storing NVRAM variables, which are necessary when booting EFI firmware:

```bash
truncate -s 64m varstore.img
```

- We also need to copy the ARM UEFI firmware into a bigger file:

```bash
truncate -s 64m efi.img
dd if=/usr/share/qemu-efi-aarch64/QEMU_EFI.fd of=efi.img conv=notrunc
```

- Fetch the Ubuntu cloud image: [Ubuntu cloud image](https://cloud-images.ubuntu.com/?_gl=1*1mhwemr*_gcl_au*MTY2NDY0MzY2My4xNzE3MzA0NTc3&_ga=2.84638489.1095170221.1720203776-324385169.1720203776) . I using latest Jammy cloud image : [`ubuntu-22.04-server-cloudimg-arm64.img`.](https://cloud-images.ubuntu.com/jammy/current/)
- Create config file name cloud-ini.iso. This will automatically configure your virtual machine to allow SSH with pre-created users.
    - Step 1: Create a file named user-data with the following content to define the user and enable SSH
    
    ```bash
    #cloud-config
    users:
      - name: m41n
        sudo: ['ALL=(ALL) NOPASSWD:ALL']
        groups: sudo
        shell: /bin/bash
        lock_passwd: false
        passwd: $6$rounds=4096$abcdefghijk$l23Rr98h8wHF9F/ZZ9EmsFIQL4Pll0ad/QlRAI8r2cnxzX.RyQKduD8TvB9dHYkh0ZP8dF5aYhIgDgZC/6XMA/
    
    chpasswd:
      list: |
        user:yourpassword
      expire: False
    
    ssh_pwauth: True
    ```
    
    - Replace `yourpassword` with the password you want to set for user `user`. The password above is encrypted using SHA-512. You can use the mkpasswd command to generate an encrypted password
    
    ```bash
    sudo apt-get install whois
    mkpasswd --method=sha-512
    ```
    
    - Step 2: Create cloud-init disk
    
    ```bash
    sudo apt-get install cloud-image-utils
    cloud-localds cloud-init.iso user-data
    ```
    

### Run

```powershell
C:\'Program Files'\qemu\qemu-system-aarch64 -m 2048 -cpu max -M virt -nographic -drive if=pflash,format=raw,file=efi.img,readonly=on -drive if=pflash,format=raw,file=varstore.img -drive if=none,file=jammy-server-cloudimg-arm64.img,id=hd0 -device virtio-blk-device,drive=hd0 -netdev user,id=net0,hostfwd=tcp::2222-:22 -device virtio-net-device,netdev=net0 -cdrom cloud-init.iso -serial mon:stdio
```

- -serial mon:stdio : Connect to the virtual machine via console to check the status of the SSH server (if you don’t use it. You will connect to the virtual machine via root )

→ You can ssh connect to the virtual machine: `ssh user@127.0.0.1 -p 2222`

## File example (chip Intel i5-11400H)
- varstore.img
- efi.img
- cloud-init.iso (test:123)

# Reference

https://ubuntu.com/server/docs/boot-arm64-virtual-machines-on-qemu

[https://gist.github.com/oznu/ac9efae7c24fd1f37f1d933254587aa4](https://gist.github.com/oznu/ac9efae7c24fd1f37f1d933254587aa4)