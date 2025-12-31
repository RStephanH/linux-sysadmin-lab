# QEMU/KVM Setup on a Host

## Goal

Install and verify QEMU/KVM to run Linux virtual machines.

## Environment

- Host OS : Arch based
- User : non-root with sudo

## Recap

QEMU is a generic and open-source machine emulator and virtualizer.

It can operate in several modes:

- **Full system emulation**  
Emulates an entire machine, allowing you to run operating systems for different CPU architectures.  
This mode is flexible but slower because everything is emulated in software.

- **User-mode emulation**  
Allows running individual programs compiled for another architecture on a Linux/BSD system.  
This is mainly used for development and testing, not for system administration.

- **Hardware-assisted virtualization**  
When combined with a hypervisor such as **KVM** or **Xen**, QEMU can run virtual machines with near-native performance by using CPU virtualization extensions (Intel VT-x / AMD-V).

### KVM (Kernel-based Virtual Machine)

KVM is a Linux kernel module that turns the Linux kernel into a type-1 hypervisor.

- Uses hardware virtualization features of the CPU
- Provides near-native performance
- QEMU is commonly used as the userspace component to manage KVM virtual machines

In this lab, **QEMU + KVM** is the main virtualization stack.

### Xen (brief mention)

Xen is another hypervisor that can also be used with QEMU.

- Designed as a standalone hypervisor
- More common in large or legacy enterprise environments

Xen is **not used in this lab**, but it is mentioned for context since QEMU supports it.

## Steps
>
> [!WARNING]
> Please take time to check the hardware and system requirements
>
### Step 1 - Install the required packages
>
> [!note] QEMU variants
> In this lab We will use mostly the "Full system emulation"

```bash
sudo pacman qemu-full libvirt virt-manager
```

### Step 2 - Enable libvirtd

```bash
sudo systemctl enable libvirtd --now 
```

### Step 3 - Add the user to the libvirt group

```bash
sudo usermod -aG libvirt $USER
```
