# HP WMI Custom Kernel Module Installation Guide

This guide describes how to apply your modified `hp-wmi.c` kernel module that enables 65W GPU Dynamic Boost on the HP Omen Transcend 14 (8C58).

## 1. Manual Installation (Temporary for testing)

If you just want to test the module quickly or are making active changes to `hp-wmi.c`, you can compile and insert the module manually. Note that this change will not persist across reboots or kernel updates.

### Compilation
Inside the directory containing your modified `hp-wmi.c` and `Makefile`:
```bash
make CC=clang # If you don't use clang, just use "make"
```

### Loading the Module
Unload the default module provided by your Linux distribution:
```bash
sudo rmmod hp_wmi
```

Insert your newly compiled module:
```bash
sudo insmod hp-wmi.ko
```
*(Check `dmesg | tail` to verify it loaded successfully without errors).*

---

## 2. Automated DKMS Installation (Persistent across kernel updates)

To ensure your custom module survives system reboots and automatically recompiles whenever you install a new Linux kernel (like `linux-cachyos`), you should use DKMS (Dynamic Kernel Module Support).

### Step 1: Prepare the DKMS Source Directory
Create a directory in `/usr/src` for your custom module. 
```bash
sudo mkdir -p /usr/src/hp-wmi-custom-1.0
```

Copy your modified `hp-wmi.c` and a clean `Makefile` into this new directory:
```bash
sudo cp hp-wmi.c /usr/src/hp-wmi-custom-1.0/
sudo cp Makefile /usr/src/hp-wmi-custom-1.0/
```

### Step 2: Create `dkms.conf`
Inside the `/usr/src/hp-wmi-custom-1.0` directory, create a file named `dkms.conf` with the following contents:

```ini
PACKAGE_NAME="hp-wmi-custom"
PACKAGE_VERSION="1.0"
MAKE[0]="make -C $kernel_source_dir M=$dkms_tree/$PACKAGE_NAME/$PACKAGE_VERSION/build"
CLEAN="make -C $kernel_source_dir M=$dkms_tree/$PACKAGE_NAME/$PACKAGE_VERSION/build clean"
BUILT_MODULE_NAME[0]="hp-wmi"
DEST_MODULE_LOCATION[0]="/kernel/drivers/platform/x86/hp/"
AUTOINSTALL="yes"
```

### Step 3: Register, Build, and Install via DKMS
Add the module to DKMS:
```bash
sudo dkms add -m hp-wmi-custom -v 1.0
```

Instruct DKMS to build the module for your current kernel:
```bash
sudo dkms build -m hp-wmi-custom -v 1.0
```

Finally, install the built module:
```bash
sudo dkms install -m hp-wmi-custom -v 1.0
```

### Verification
DKMS will now automatically override the default `hp-wmi.ko` with your custom version. 
To ensure it loads on boot, simply reboot your computer. You can verify your custom DKMS module is active by running:
```bash
dkms status
```
You should see `hp-wmi-custom, 1.0: installed`.
