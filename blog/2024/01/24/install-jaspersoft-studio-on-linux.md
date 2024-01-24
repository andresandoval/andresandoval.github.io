# Install Jaspersoft® Studio Community Edition on Linux

1. Download Jaspersoft® Studio Community Edition from https://community.jaspersoft.com/files/file/19-jaspersoft%C2%AE-studio-community-edition/

2. Uncompress downloaded file

    ```console
    tar -zvxf js-studiocomm_6.20.6_linux_x86_64.tgz
    ```

3. Move uncompressed to the desired instalation path

    ```console
    sudo mkdir /opt/js-studiocomm
    sudo mv js-studiocomm_6.20.6 /opt/js-studiocomm
    ```

4. Create a .desktop file for integrating with app launcher (GNOME)

    ```console
    sudo touch /opt/js-studiocomm/js-studiocomm.desktop
    ```

    ```ini
    [Desktop Entry]
    Version=1.0
    Name=Jaspersoft® Studio Community Edition
    GenericName=Jaspersoft® Studio Community Edition
    GenericName[de]=Jaspersoft® Studio Community Edition
    GenericName[ru]=Jaspersoft® Studio Community Edition
    Type=Application
    Exec=/opt/js-studiocomm/js-studiocomm_6.20.6/runjss.sh
    Keywords=Reports;Dev;Java;
    Keywords[de]=Reports;Dev;Java;
    Keywords[ru]=Reports;Dev;Java;
    Icon=/opt/js-studiocomm/js-studiocomm_6.20.6/icon.xpm
    Categories=Development;
    ```

5. Create a symlink of the .desktop file to the app launcher path

    ```console
    cd /usr/share/applications
    sudo ln -s /opt/js-studiocomm/js-studiocomm.desktop .
    ```