# kalandra

**Kalandra** is a [Snap](https://snapcraft.io/kalandra) wrapper for [`apt-mirror2`](https://gitlab.com/apt-mirror2/apt-mirror2), designed to support airgapped Debian/Ubuntu mirror deployments. It includes an integrated apache2 setup for easy hosting, and export/import tools for airgapped deployments. The core apache and apt-mirror2 sources are unaltered, so all snapcraft bits act as an overlay to allow for clean rebasing.

## 📦 Features

- Snap-packaged `apt-mirror2` for easy installation
- Persistent data and config in `$SNAP_COMMON`
- Embedded Apache server to serve mirrored packages
- Easy export/import of mirror data for transfer to airgapped systems

---

## 🔧 Commands

### `kalandra`
Runs `apt-mirror2` using a user-provided `mirror.list` configuration file located at:

```
/var/snap/kalandra/common/apt/mirror.list
```

Edit this file to control what repositories and architectures are mirrored. By default, no mirrors are configured, so adjust according to your needs.
One of the key features of apt-mirror2 is explicit includes and excludes. Assuming you wanted to only mirror the wget package from the `noble` release pocket, the following config would work:

```
set base_path         /var/spool/apt-mirror
set mirror_path       $base_path/mirror
set skel_path         $base_path/skel
set var_path          $base_path/var
set nthreads          8
set _tilde            0
deb [ arch=amd64 ] http://archive.ubuntu.com/ubuntu noble main restricted universe multiverse
include_source_name http://archive.ubuntu.com/ubuntu wget
```
This should result in ~200MBs of meta, along with the package itself, significantly more lightweight than the typical full mirror that apt-mirror requires

### `kalandra export`
Creates a compressed tarball of all mirror content and config files from:

- `/var/snap/kalandra/common/var/`
- `/var/snap/kalandra/common/conf/`

The archive is saved in `$SNAP_COMMON` (typically `/var/snap/kalandra/common/`).

Use this to back up or transfer mirror data to another system. Particularly useful in an airgapped deployment for easy repository portability.

### `kalandra import <path-to-archive>`
Restores an exported tarball into the snap’s data directory (`$SNAP_COMMON`). This is used to populate a new system with mirror contents or restore a backup.

```bash
kalandra import /path/to/export.tar.gz
```
Please note that import requires that the import requires that the archive be located in a place accessible to the confined snap. You are best off `mv`ing the file to the /var/snap/kalandra/common/ directory prior to calling import.

---

## 🌐 Apache Integration

Kalandra includes an embedded Apache HTTP server for serving mirrored content.

To start the web server:

```bash
snap start kalandra.apache
```

Apache will serve files from:

```
/var/snap/kalandra/common/var/spool/apt-mirror/
```

Assuming the above example was used for wget, adding the following to the sources.list would allow the client to install the wget package:
```
deb [ arch=amd64 ] http://localhost:8080/ubuntu noble main
```
Keep in mind, clients will need the gpg keys for the appropriate repository as well if they are not present by default. (ex for the Ubuntu Pro repositories).

### Apache Configuration

The default Apache config template is located at:

```
/var/snap/kalandra/common/conf/apache2.conf
```

You may edit this file to adjust document root, logging, or enable directory indexing (to resemble a standard Debian/Ubuntu archive layout). The shipped config already has most of this enabled, hosting on port 8080 by default.

Make sure to restart the service after changes:

```bash
snap restart kalandra.apache
```

You can a persistent daemon with auto-restarts using 
```bash
snap enable kalandra.apache
```

---

## 📁 Data & Configuration Paths

All persistent data lives under the snap's `$SNAP_COMMON` path, typically:

- Mirror data: `/var/snap/kalandra/common/var/spool/apt-mirror/`
- Apache config: `/var/snap/kalandra/common/conf/apache2.conf`
- Mirror config: `/var/snap/kalandra/common/apt/mirror.list`

These locations are safe to mount, inspect, or back up.

---

## 🚀 Installation

This snap is available in the public store via:
```
snap install kalandra
```
or in the [snapstore](https://snapcraft.io/kalandra) directly.

If you would like to build from this source, you can clone the repository and from within the directory call:
```bash
snapcraft
```
and then install the snap locally for testing:

```bash
snap install --dangerous kalandra_*.snap
```

