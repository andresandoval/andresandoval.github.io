# Graphic issues after updating mesa drivers on Fedora 39

After updaing Fedora I found some issues related to graphics, specially on Chrome and PWA. some of them;

- Missing emoji icons
- Missing avatar images (PWA Teams)
- Error in Chrome console when loading images


After a quick research I list all the dnf history to check what was updated (since I ran `sudo dnf update -y` I didn't check it before updating):

```console
dnf history
```

```console
[...]
43 | update
[...] 
```

checking in detail this history entry I found:

```console
Transaction ID : 43
Begin time     : Tue 26 Dec 2023 07:58:49 AM -05
Begin rpmdb    : ################
End time       : Tue 26 Dec 2023 08:00:30 AM -05 (101 seconds)
End rpmdb      : ########################
User           : Andres Sandoval <#########>
Return-Code    : Success
Releasever     : 39
Command Line   : update -y
Comment        : 
Packages Altered:
    Install       kernel-6.6.8-200.fc39.x86_64                              @updates
    Install       kernel-core-6.6.8-200.fc39.x86_64                         @updates
    Install       kernel-devel-6.6.8-200.fc39.x86_64                        @updates
    Install       kernel-modules-6.6.8-200.fc39.x86_64                      @updates
    Install       kernel-modules-core-6.6.8-200.fc39.x86_64                 @updates
    Install       kernel-modules-extra-6.6.8-200.fc39.x86_64                @updates
    [...]
    Upgrade       loupe-45.3-1.fc39.x86_64                                  @updates
    Upgraded      loupe-45.2-1.fc39.x86_64                                  @@System
    Upgrade       man-pages-6.05-5.fc39.noarch                              @updates
    Upgraded      man-pages-6.05-3.fc39.noarch                              @@System
    Upgrade       mesa-dri-drivers-23.3.1-4.fc39.x86_64                     @updates
    Upgraded      mesa-dri-drivers-23.3.1-3.fc39.x86_64                     @@System
    Upgrade       mesa-filesystem-23.3.1-4.fc39.x86_64                      @updates
    Upgraded      mesa-filesystem-23.3.1-3.fc39.x86_64                      @@System
    Upgrade       mesa-libEGL-23.3.1-4.fc39.x86_64                          @updates
    Upgraded      mesa-libEGL-23.3.1-3.fc39.x86_64                          @@System
    Upgrade       mesa-libGL-23.3.1-4.fc39.x86_64                           @updates
    Upgraded      mesa-libGL-23.3.1-3.fc39.x86_64                           @@System
    Upgrade       mesa-libgbm-23.3.1-4.fc39.x86_64                          @updates
    Upgraded      mesa-libgbm-23.3.1-3.fc39.x86_64                          @@System
    Upgrade       mesa-libglapi-23.3.1-4.fc39.x86_64                        @updates
    Upgraded      mesa-libglapi-23.3.1-3.fc39.x86_64                        @@System
    Upgrade       mesa-libxatracker-23.3.1-4.fc39.x86_64                    @updates
    Upgraded      mesa-libxatracker-23.3.1-3.fc39.x86_64                    @@System
    Upgrade       mesa-va-drivers-23.3.1-4.fc39.x86_64                      @updates
    Upgraded      mesa-va-drivers-23.3.1-3.fc39.x86_64                      @@System
    Upgrade       mesa-vulkan-drivers-23.3.1-4.fc39.x86_64                  @updates
    Upgraded      mesa-vulkan-drivers-23.3.1-3.fc39.x86_64                  @@System
    Upgrade       nspr-4.35.0-15.fc39.x86_64                                @updates
    [...]
```

Quickly I realized that was something related to `mesa` drivers, since I know that its the responsable of handling Graphic card and video chips. So I decided to revert its update:

```console
sudo dnf downgrade mesa*
```

this was the downgrade result:

```console
Transaction ID : ##
Begin time     : Tue 26 Dec 2023 09:33:47 AM -05
Begin rpmdb    : ###
End time       : Tue 26 Dec 2023 09:33:49 AM -05 (2 seconds)
End rpmdb      : ###
User           : Andres Sandoval <###>
Return-Code    : Success
Releasever     : 39
Command Line   : downgrade mesa*
Comment        : 
Packages Altered:
    Downgrade  mesa-dri-drivers-23.2.1-2.fc39.x86_64    @fedora
    Downgraded mesa-dri-drivers-23.3.1-4.fc39.x86_64    @@System
    Downgrade  mesa-filesystem-23.2.1-2.fc39.x86_64     @fedora
    Downgraded mesa-filesystem-23.3.1-4.fc39.x86_64     @@System
    Downgrade  mesa-libEGL-23.2.1-2.fc39.x86_64         @fedora
    Downgraded mesa-libEGL-23.3.1-4.fc39.x86_64         @@System
    Downgrade  mesa-libGL-23.2.1-2.fc39.x86_64          @fedora
    Downgraded mesa-libGL-23.3.1-4.fc39.x86_64          @@System
    Downgrade  mesa-libgbm-23.2.1-2.fc39.x86_64         @fedora
    Downgraded mesa-libgbm-23.3.1-4.fc39.x86_64         @@System
    Downgrade  mesa-libglapi-23.2.1-2.fc39.x86_64       @fedora
    Downgraded mesa-libglapi-23.3.1-4.fc39.x86_64       @@System
    Downgrade  mesa-libxatracker-23.2.1-2.fc39.x86_64   @fedora
    Downgraded mesa-libxatracker-23.3.1-4.fc39.x86_64   @@System
    Downgrade  mesa-va-drivers-23.2.1-2.fc39.x86_64     @fedora
    Downgraded mesa-va-drivers-23.3.1-4.fc39.x86_64     @@System
    Downgrade  mesa-vulkan-drivers-23.2.1-2.fc39.x86_64 @fedora
    Downgraded mesa-vulkan-drivers-23.3.1-4.fc39.x86_64 @@System
```

After rebooting all worked as before