# Run JDownloader2 inside a Docker container

```console
docker run -d \
    --name=jdownloader-2 \
    -p 58000:5800 \
    -p 59000:5900 \
    -v /home/user/JDownloader/config:/config:rw \
    -v /home/user/JDownloader/output:/output:rw \
    jlesage/jdownloader-2


ln -s /home/user/JDownloader/output /home/user/JDownloads
```

with this command you will get:
 
 - A web server running on [http://localhost:58000](http://localhost:58000)
 - A VNC server running on [localhost:59000](localhost:59000)


 ### Source

 - https://github.com/jlesage/docker-jdownloader-2?tab=readme-ov-file#usage
 - (render X gui on container) https://github.com/jlesage/docker-baseimage-gui  