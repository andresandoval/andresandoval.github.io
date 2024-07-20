# How to check the location of Gnome Destop entries

Run:
```console
echo $XDG_DATA_DIRS
```

this will give you a coma separated list of directories where GDM reads the .desktop files


for updating the mime type list of files supported by each app, you have to run:

```console
update-desktop-database #or
sudo update-desktop-database #or
sudo update-desktop-database /some/dir/with/.desktopfiles

```