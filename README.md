# Backend.AI Storage Proxy
Backend.AI Storage Proxy is an RPC daemon to manage vfolders used in Backend.AI agent, with quota and
storage-specific optimization support.


## Package Structure
* `ai.backend.storage`
  - `server`: The agent daemon which communicates between Backend.AI Manager
  - `vfs`
    - The minimal fallback backend which only uses the standard Linux filesystem interfaces
  - `xfs`
    - XFS-optimized backend with a small daemon to manage XFS project IDs for quota limits
    - `agent`: Implementation of `AbstractVolumeAgent` with XFS support
  - `purestorage`
    - PureStorage-optimized backend with RapidFile Toolkit (formerly PureTools)
  - `cephfs`
    - CephFS-optimized backend with quota limit support


## Installation

### Prequisites
* Python 3.8 or higher with [pyenv](https://github.com/pyenv/pyenv)
and [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv) (optional but recommneded)

### Installation Process

First, prepare the source clone of this agent:
```console
# git clone https://github.com/lablup/backend.ai-storage-agent
```

From now on, let's assume all shell commands are executed inside the virtualenv.

Now install dependencies:
```console
# pip install -U -r requirements/dist.txt  # for deployment
# pip install -U -r requirements/dev.txt   # for development
```

Then, copy halfstack.toml to root of the project folder and edit to match your machine:
```console
# cp config/sample.toml storage-proxy.toml
```

When done, start storage server:
```console
# python -m ai.backend.storage.server
```

It will start Storage Agent daemon binded to `127.0.0.1:6020`.

NOTE: Depending on the backend, the server may require to be run as root.


## Filesystem Backends

### VFS

#### Prerequisites

* User account permission to access for the given directory


### XFS

#### Prerequisites

* Local device mounted under `/vfroot`
* Native support for XFS filesystem
* Access to root shell

#### Creating virtual XFS device for testing

Create a virtual block device mounted to `lo` (loopback) if you are the only one
to use the storage for testing:

1. Create file with your desired size
```console
# dd if=/dev/zero of=xfs_test.img bs=1G count=100
```
2. Make file as XFS partition
```console
# mkfs.xfs xfs_test.img
```
3. Mount it to loopback
```console
# export LODEVICE=$(losetup -f)
# losetup $LODEVICE xfs_test.img
```
4. Create mount point and mount loopback device, with pquota option
```console
# mkdir -p /vfroot/xfs
# mount -o loop -o pquota $LODEVICE /vfroot/xfs
```

### PureStorage FlashBlade

#### Prerequisites

* NFSv3 export mounted under `/vfroot`
* Purity API access


### CephFS

#### Prerequisites

* FUSE export mounted unde `/vfroot`
