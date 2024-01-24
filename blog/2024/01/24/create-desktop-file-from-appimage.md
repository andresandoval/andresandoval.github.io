# Create .desktop app entry from AppImage file

1. Download your AppImage file (for this example will be UltiMaker-Cura-5.6.0-linux-X64.AppImage)
2. Make the file executable

    ```console
    chmod +x UltiMaker-Cura-5.6.0-linux-X64.AppImage    
    ```

3. Extract the AppImage file

    ```console
    ./UltiMaker-Cura-5.6.0-linux-X64.AppImage --appimage-extract
    ```

4. Find .desktop file on extracted data

    ```console
    cd squashfs-root/
    find . -name *.desktop
    ```

    the outuput will be something like this:

    ```console
    ./com.ultimaker.cura.desktop
    ```

5. Move the AppImage file to the desired instalation path:

    ```console
    sudo mkdir /opt/ultimaker-cura
    sudo mv ./UltiMaker-Cura-5.6.0-linux-X64.AppImage /opt/ultimaker-cura
    ```

6. Copy the app icon to the desired instalation path:

    the icon path is defined on the .desktop file:

    ```ini
    [Desktop Entry]
    X-AppImage-Arch=x86_64
    X-AppImage-Version=5.6.0
    X-AppImage-Name=UltiMaker Cura
    Name=UltiMaker Cura
    Exec=UltiMaker-Cura
    Icon=cura-icon.png
    ```

    ```console
    sudo mv ./cura-icon.png /opt/ultimaker-cura
    ```

7. Change the Exec and Icon paths on the .desktop file accoording to steps #5 and #6

    file will be like this:

    ```ini
    [Desktop Entry]
    X-AppImage-Arch=x86_64
    X-AppImage-Version=5.6.0
    X-AppImage-Name=UltiMaker Cura
    Name=UltiMaker Cura
    Exec=/opt/ultimaker-cura/UltiMaker-Cura-5.6.0-linux-X64.AppImage
    Icon=/opt/ultimaker-cura/cura-icon.png
    Type=Application
    Terminal=false
    Categories=Utility;
    Comment=
    ```

8. Move the desktop file to the default app launcher icons path (GNOME)
    
    ```console
    sudo mv ./com.ultimaker.cura.desktop /usr/share/applications
    ```

9. Cleanup

    ```console
    cd ..
    rm -rf squashfs-root
    ```

