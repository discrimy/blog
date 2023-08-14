---
title: "How to setup secure remote storage from cloud storage (guide)"
date: 2023-08-14T23:23:00+03:00
draft: false
---

# How to setup secure remote storage from cloud storage (guide)
I thought about secure encrypted file storage for a long time. Most cloud providers doesn't encrypt file, so that means your files are not only yours but provider's too. Some providers which states they store files encrypted (like Mega) have bad CLI tools for automation and Android applicatons.

I wrote down criterias which ideal solution must satisfy:
- Client-side encryption of files (server mustn't has access to original files)
- Standart API like WebDAV and SFTP
- Large volume (hunders of GB)
- Reliably, in ideal case provider must manage it by itself
- Fully open source (because I don't believe in tales)

I found great tool named [rclone](https://rclone.org) and there was a sale for Yandex.Disk 2TB disk, so I had everything I needed to make the storage.

## How it works
![](/images/secure-storage-guide_how-it-works.svg)

Yandex.Disk (or any provider you like, rclone supports a lot of providers) stores encrypted files. Personal server hosts rclone server which serves as proxy to Yandex.Disk and handles encryption and descryption on fly. All clients connect to this server via standart protocols like WebDAV.

## How to setup

### Setup encryption on top of cloud
1. Run `rclone config` and create new remote (remote is how rclone names storages) for your cloud provider, let's name it as `external`.
2. Create new remote with type `crypt`, as `Option remote` set `external:some-folder` (directory must be empty), set `password` and `password2`, let's name this remote as `external-encr`. Save config file (you can find it with `rclone config file`) and save it to secure place, because without it you cannot restore files.
3. Check
    1. `rclone copy file.jpg external-encr:file.jpg` - copy file to encrypted remote
    2. `rclone ls external-encr:` - list files inside remote
    3. `rclone copy external-encr:file.jpg file-copy.jpg` - download file from remote

### Setup WebDAV and SFTP servers
To interact with storage via standart protocols we setup rclone server. Choose protocol by their speed, WebDAV is better for me.

```bash
# Run WebDAV server on port 8080, authenticate user `username` with password `password`
rclone serve webdav external-encr: --addr :8080 --user username --pass password
```
Replace `webdav` with `sftp` if SFTP is needed. Also consider to add:
- `--vfs-cache-mode full` - use cache
- `--vfs-cache-max-size 1G` - set cache max size

It's a bad practice to set user password in command because command will be saved to your history. It's better to set them via environment variables `RCLONE_USER` and `RCLONE_PASS`.

## How to use

### CLI
You can use `rclone` itself to interact with your storage using `external-encr` remote for direct access or create new remote with type `webdav` to your proxy server to interact via it.

### Android
There is good file manager [Solid Explorer](https://play.google.com/store/apps/details?id=pl.solidexplorer2&pli=1) (it's partially paid, but it's worth it), and this app can connect to WebDAV or SFTP server.

### Linux (KDE)
KDE Dolphin has good WebDAV support.

### Windows
I don't managed to find good open source file manager with WebDAV support, and builtin File Explorer behaves strangely. But `rclone` can mount remote as a drive! You can write simple script to run command to mount and use it as local storage.

```cmd
# Mount storage as drive N:
# It's recommended to add caching (--vfs-*)
rclone mount external-encr: n:
```

### Web
[Filestash](https://www.filestash.app) is good web application to access WebDAV server. You can self-host it on the same server and connect to storage via Filestash, I use it on desktop.

## Known issues
Listing folder with a lot of files inside (tens for thousands) can take a lot of time, but I think it's provider issue, and this doesn't break anything.
