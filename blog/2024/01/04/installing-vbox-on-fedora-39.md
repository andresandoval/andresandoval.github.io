# Installing Vbox on Fedora 39

## Installing from repo

Install following VBox steps on official website:

- Download repo file from: https://download.virtualbox.org/virtualbox/rpm/fedora/virtualbox.repo

- Move repo file to `/etc/yum.repos.d/`
- Install VBox using dnf package manager: `sudo dnf install VirtualBox`

This method seems to be not working for Fedora 39 using secure Boot, since it needs to has their modules signed.

I tried to sign the VBox kernel modules without success, anyway, I post this steps for historical tracking effor

### Post install steps

#### Add current user to vbox users group

In order to have access to all features (such as usb drive integration) we need to add current user to the vbox usergroup:

- `sudo usermod -a -G vboxusers $USER`

#### Sign kernel modules manually

== NO SUCCESS TRING TO SIGN MANUALLY KERNEL MODULES ==

1. As root user, create a RSA key pair to sign the modules:

    ```console
    $ sudo -i
    # mkdir /root/module-signing
    # cd /root/module-signing
    # openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=YOUR_NAME/"
    [...]
    # chmod 600 MOK.priv
    ```

    just change `YOUR_NAME` with your real name (just for distinguish the key from others)

    alternatively you can create a bash file with this:

    ```bash
    #!/bin/bash

    name="$(getent passwd $(whoami) | awk -F: '{print $5}')"
    out_dir='/root/module-signing'
    sudo mkdir ${out_dir}
    sudo openssl \
        req \
        -new \
        -x509 \
        -newkey \
        rsa:2048 \
        -keyout ${out_dir}/MOK.priv \
        -outform DER \
        -out ${out_dir}/MOK.der \
        -days 36500 \
        -subj "/CN=${name}/"
    sudo chmod 600 ${out_dir}/MOK*
    ```

    You will be asked to include a password for the key

2. Import the MOK ("Machine Owner Key") so it can be trusted by the system.

    ```console
    sudo mokutil --import /root/module-signing/MOK.der
    ```

    you should provide a password, `123` is ok since its a single use one

3. Reboot the machine. When the bootloader starts, the MOK manager EFI utility should automatically start. It will ask for parts of the password supplied in step 2 (123). Choose to “Enroll MOK”, then you should see the key imported in step 2. Complete the enrollment steps, then continue with the boot

4. We can check if the MOK was enrolled by running the command:
    
    ```console
    mokutil --list-enrolled
    ```

    it should list all the enrolled certificates, including yours:

    ```console
    [...]
    [key 2]
    SHA1 Fingerprint: 
    ##:##:##:##:##:##:##......
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                ##:##:##:##:##:##:##......
            Signature Algorithm: sha256WithRSAEncryption
            Issuer: CN=Andres Sandoval
            Validity
                Not Before: Jan  4 21:14:11 2024 GMT

    [...]
    ```

    as you can see, the `Issuer CN` is the field that identify the key

    some pages says that the command `dmesg | grep '[U]EFI.*cert'` shows the key when enrolled, but it didn't work for me

5. Sign the kernel modules.

    We have to sign all VBox kernel modules under directory `$(dirname $(modinfo -n vboxdrv))/` using this script:

    ```bash
    #!/bin/bash

    for modfile in $(dirname $(modinfo -n vboxdrv))/*.ko; do
    echo "Signing $modfile"
    /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 \
                                    /root/module-signing/MOK.priv \
                                    /root/module-signing/MOK.der "$modfile"
    done
    ```

    you must change mode to `chmod 700` before runnig it

    if this script fails with some errors you must check first if files inside the directory `$(dirname $(modinfo -n vboxdrv))/` are compressed or not (compresed files has .xz extension)

    if kernel modules files are compressed you must uncompress them first using `unxz vbxxxx.ko.xz` command, then you can run the script again, then you have to compress all kernel module files using `xz -f vbxxxx.ko`


6. Load the vboxdrv module.

    ```console
    sudo modprobe vboxdrv
    ```


Step #6 didn't work for me, it shows an error, so no furter speps for this type of instalation.

Reference #1: https://gist.github.com/reillysiemens/ac6bea1e6c7684d62f544bd79b2182a4

Reference #2: https://stegard.net/2016/10/virtualbox-secure-boot-ubuntu-fail/


## Installing from rpm file

I move to this instalation type since I was not able to sign the kernel modules produced by the instalation from repo.

1. Remove previous VBox versions.

    Before installing, you have to uninstall all the VirtualBox related things installed from the repo (if you tried it first with no success) by doing:

    ```console
    sudo dnf remove VirtualBox*
    ```

    you have to include the `*` for remove all related packages including the old kernel module files

    If you want to remove the MOK key previusly enrolled you have to run

    ```console
        sudo mokutil --import /root/module-signing/MOK.der
    ```

    you should provide a password, `123` is ok since its a single use one. Reboot the machine. When the bootloader starts, the MOK manager EFI utility should automatically start. It will ask for parts of the password supplied (123). Choose to “Remove Enroll MOK” or something like that, continue with steps until the manager asks to reboot the machine.

    you can check that the key is no longer enrolled by runnig: 

    ```console
    mokutil --list-enrolled
    ```

2. Download the latest rpm from the VBox Linux downloads page:

    Use download link for your Distro and version here: https://www.virtualbox.org/wiki/Linux_Downloads

3. Install the package

    ```console
    sudo dnf install ./VirtualBox-7.0-7.0.12_159484_fedora36-1.x86_64.rpm
    ```

4. Instalation will fail if you have enable SecureBoot instead of Bios Boot. So you have to sign the kernel modules.

    Usually the same VBox installer give you the steps for this process, those are:

    ```console
    vboxdrv.sh: failed: 

    System is running in Secure Boot mode, however your distribution
    does not provide tools for automatic generation of keys needed for
    modules signing. Please consider to generate and enroll them manually:

        sudo mkdir -p /var/lib/shim-signed/mok
        sudo openssl req -nodes -new -x509 -newkey rsa:2048 -outform DER -addext "extendedKeyUsage=codeSigning" -keyout /var/lib/shim-signed/mok/MOK.priv -out /var/lib/shim-signed/mok/MOK.der
        sudo mokutil --import /var/lib/shim-signed/mok/MOK.der
        sudo reboot

    Restart "rcvboxdrv setup" after system is rebooted

    ```


    All the process is similar to the one described on the [Signing kernel modules](#sign-kernel-modules-manually) section.

    from personal opinion, you can check the command:

    ```console
    sudo openssl req -nodes -new -x509 -newkey rsa:2048 -outform DER -addext "extendedKeyUsage=codeSigning" -keyout /var/lib/shim-signed/mok/MOK.priv -out /var/lib/shim-signed/mok/MOK.der
    ``` 
    adding the parameters:

    - `-subj "/CN=YOUR_NAME/"` for having a more clear identification of the key later

    removing the parameters:

    - `-nodes` for including a password during the key generation


### Post install steps

#### Add current user to vbox users group

In order to have access to all features (such as usb drive integration) we need to add current user to the vbox usergroup:

- `sudo usermod -a -G vboxusers $USER`


## Conclussion

Thats it, it covers all the adventures trying to install VBox on Fedora.

The  kernel modules is the main issue, there is a quick way to handle this by disabling SecureBoot on bios, but is quite insecure, since it disabled all the kernel module validation.

One thing to consider is that with every new kernel version update, you must sign the modules again, so its a good idea to have this guide in hand.