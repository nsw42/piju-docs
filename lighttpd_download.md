# Using lighttpd to download the music files

If you're not running the web ui, it can be useful to have the ability to download your mp3/etc from the server.

```sh
apk add lighttpd
cd /etc/lighttpd/
vi lighttpd.conf
```

Set `var.basedir` to `/media/mmcblk0p2/media`, update `server.document-root` so that it doesn't look for `htdocs`, enable `mod_dirlisting`, and save.

Start the service:

```sh
/etc/init.d/lighttpd start
```
