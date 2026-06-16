# podman-imagefs-builds

## Quickstart

Install system podman to get all of the extra helper utilites like conmon and crun

```
dnf install -y podman
```

For the imagefs plugin, we can either use squashfs or erofs. There are implementation differences that are captured in the [IMPLEMENTATION.md](IMPLEMENTATION.md)

| Tool | RHEL8 | RHEL9 | RHEL10 |
|--|--|--|--|
| squashfs-tools | system | system | system | 
| squashfuse | EPEL | EPEL | EPEL |
| squashfs-tools-ng | EPEL | EPEL | EPEL | 
| erofs-utils | N/A | EPEL | system |
| erofs-fuse | N/A | EPEL | EPEL | 

### Install EPEL

```
dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-10.noarch.rpm
```

### If using SquashFS

Using either the `squashfs-tools` or `squashfs-tools-ng` packages along with squashfuse

```
dnf install -y squashfs-tools squashfuse
```

### If using EROFS

Install erofs-utils and erofs-fuse
```
dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
dnf install -y erofs-utils erofs-fuse
```

## Testing

Configure `~/.config/containers/storage.conf` (rootless):

```toml
[storage]
driver = "imagefs"

[storage.options.imagefs]
imagefs_format = "squashfs"          # Optional: "erofs" (default) or "squashfs"
imagefs_compression = "zstd"         # Optional: compression algorithm
                                     # EROFS: lz4, lz4hc, deflate, libdeflate, lzma, zstd
                                     # SquashFS: gzip, xz, lz4, zstd, lzo, lzma
```

Pull latest `podman` binary from [releases in this repository](https://github.com/redhat-hpc/podman-imagefs-builds/releases)

### Issues with userspace networking (pasta)

Pasta on my RHEL9 test box is not compatible with latest podman, run with host networking to bypass.

```
[jason@rhel9-local ~]$ ./podman-rhel9-amd64 run -it --rm docker.io/busybox:latest
Trying to pull docker.io/library/busybox:latest...
Getting image source signatures
Copying blob b05093807bb0 done   |
Copying config c6348fa86b done   |
Writing manifest to image destination
Error: pasta failed with exit code 1:
netns dir open: Permission denied, exiting

[jason@rhel9-local ~]$ ./podman-rhel9-amd64 run -it --rm --net=host docker.io/busybox:latest
/ #
```
