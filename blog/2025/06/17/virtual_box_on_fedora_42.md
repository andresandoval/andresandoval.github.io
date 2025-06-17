# Error running VM on fedora 42

```console
VirtualBox can't operate in VMX root mode. Please disable the KVM kernel extension, recompile your kernel and reboot (VERR_VMX_IN_VMX_ROOT_MODE).
```
means that **VirtualBox is trying to use hardware virtualization (VT-x/VMX), but it's already being used by KVM (Kernel-based Virtual Machine)**, which is typically used by **GNOME Boxes**, **virt-manager**, **QEMU**, etc.

* * * * *

### 🔧 Solutions (choose one):

#### ✅ **Option 1: Run VirtualBox without KVM interfering (temporarily disable KVM)**

You can prevent KVM from loading **temporarily** by blacklisting its modules:

```bash
sudo modprobe -r kvm_intel kvm
```
> For AMD CPUs, use `kvm_amd` instead of `kvm_intel`.

Then try launching VirtualBox again.

> ⚠️ This only works until reboot. To make it permanent, see Option 2.

* * * * *

#### 🛑 **Option 2: Permanently disable KVM (not recommended if you use GNOME Boxes)**

1.  Create a blacklist file:

    ```bash
    echo "blacklist kvm" | sudo tee /etc/modprobe.d/blacklist-kvm.conf
    echo "blacklist kvm_intel" | sudo tee -a /etc/modprobe.d/blacklist-kvm.conf
    ```

2.  Rebuild your initramfs (on Fedora):

    ```bash
    sudo dracut -f
    ```

3Reboot.

* * * * *

#### ⚖️ **Option 3: Use GNOME Boxes (KVM) OR VirtualBox -- not both**

VirtualBox and KVM can't run simultaneously because they both require VT-x in "root mode".

You can:

-   Use GNOME Boxes and uninstall VirtualBox (recommended for Fedora/Ubuntu).

-   Use VirtualBox and disable KVM as above.

* * * * *

#### 🔍 To check if KVM is loaded:

```bash
lsmod | grep kvm
```

If you see `kvm` and `kvm_intel` (or `kvm_amd`), then it's active.