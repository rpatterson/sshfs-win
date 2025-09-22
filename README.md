<h1 align="center">
    <img src="art/sshfs-glow.png" width="256"/>
    <br/>
    <br/>
    SSHFS-Win &middot; SSHFS for Windows
</h1>

<p align="center">
    <b>Download</b><br>
    <a href="https://github.com/billziss-gh/sshfs-win/releases/latest">
        <img src="https://img.shields.io/github/release/billziss-gh/sshfs-win.svg?label=stable&style=for-the-badge"/>
    </a>
    <a href="https://github.com/billziss-gh/sshfs-win/releases">
        <img src="https://img.shields.io/github/release/billziss-gh/sshfs-win/all.svg?label=latest&colorB=e52e4b&style=for-the-badge"/>
    </a>
    <!-- a href="https://chocolatey.org/packages/sshfs">
        <img src="https://img.shields.io/badge/choco-install%20sshfs-black.svg?style=for-the-badge"/>
    </a -->
</p>

<p align="center">
    <b>GUI Frontends</b><br>
    <a href="https://github.com/mhogomchungu/sirikali/releases/latest">
        <img src="https://img.shields.io/github/release/mhogomchungu/sirikali.svg?label=SiriKali&style=for-the-badge"/>
    </a>
    <a href="https://github.com/evsar3/sshfs-win-manager/releases/latest">
        <img src="https://img.shields.io/github/release/evsar3/sshfs-win-manager.svg?label=SSHFS-Win-Manager&style=for-the-badge"/>
    </a>
</p>

SSHFS-Win is a minimal port of [SSHFS](https://github.com/libfuse/sshfs) to Windows. Under the hood it uses [Cygwin](https://cygwin.com) for the POSIX environment and [WinFsp](https://github.com/billziss-gh/winfsp) for the FUSE functionality.

## Installation

- Install the latest version of [WinFsp](https://github.com/billziss-gh/winfsp/releases/latest).
- Install the latest version of [SSHFS-Win](https://github.com/billziss-gh/sshfs-win/releases). Choose the x64 or x86 installer according to your computer's architecture.

Both can also be easily installed with [WinGet](https://github.com/microsoft/winget-cli):

```console
winget install SSHFS-Win.SSHFS-Win
```

## Basic Usage

Once you have installed WinFsp and SSHFS-Win you can map a network drive to a directory on an SSHFS host using Windows Explorer or the `net use` command.


### Windows Explorer

In Windows Explorer select This PC > Map Network Drive and enter the desired drive letter and SSHFS path using the following UNC syntax:

    \\sshfs\REMUSER@HOST[\PATH]

The first time you map a particular SSHFS path you will be prompted for the SSHFS username and password. You may choose to save these credentials with the Windows Credential Manager in which case you will not be prompted again.

In order to unmap the drive, right-click on the drive icon in Windows Explorer and select Disconnect.

<p align="center">
<img src="cap.gif" height="450"/>
</p>

### Command Line

You can map a network drive from the command line using the `net use` command:

```
> net use X: \\sshfs\billziss@mac2018.local
The password is invalid for \\sshfs\billziss@mac2018.local.

Enter the user name for 'sshfs': billziss
Enter the password for sshfs:
The command completed successfully.
```

You can list your `net use` drives:

```
$ net use
New connections will be remembered.


Status       Local     Remote                    Network

-------------------------------------------------------------------------------
             X:        \\sshfs\billziss@mac2018.local
                                                WinFsp.Np
The command completed successfully.
```

Finally you can unmap the drive as follows:

```
$ net use X: /delete
X: was deleted successfully.
```

## UNC Syntax

The complete UNC syntax is as follows:

    \\sshfs\[LOCUSER=]REMUSER@HOST[!PORT][\PATH]
    \\sshfs.r\[LOCUSER=]REMUSER@HOST[!PORT][\PATH]
    \\sshfs.k\[LOCUSER=]REMUSER@HOST[!PORT][\PATH]
    \\sshfs.kr\[LOCUSER=]REMUSER@HOST[!PORT][\PATH]

- `REMUSER` is the remote user (i.e. the user on the SSHFS host whose credentials are being used for access).
- `HOST` is the SSHFS host.
- `PORT` is the remote port on the SSHFS host (optional; default is 22).
- `PATH` is the remote path. This is interpreted as follows:
    - The `sshfs` prefix maps to `HOST:~REMUSER/PATH` on the SSHFS host (i.e. relative to `REMUSER`'s home directory).
    - The `sshfs.r` prefix maps to `HOST:/PATH` on the SSHFS host (i.e. relative to the `HOST`'s root directory).
    - The `sshfs.k` prefix maps to `HOST:~REMUSER/PATH` and uses the ssh key in
      `%USERPROFILE%/.ssh/id_rsa` (where `%USERPROFILE%` is the home directory of the
      local Windows user). To specify a different specific key, define an alias of the
      HOST with the specific private ssh key you want to use in the [ssh
      config](#ssh-client-configuration). BEWARE: only keys without a pass phrase are
      supported.
    - The `sshfs.kr` prefix maps to `HOST:/PATH` and uses the ssh key in
      `%USERPROFILE%/.ssh/id_rsa`. To specify a different specific key, define an alias
      of the HOST with the specific private ssh key you want to use in the [ssh
      config](#ssh-client-configuration). BEWARE: only keys without a pass phrase are
      supported.
- `LOCUSER` is the local Windows user (optional; `USERNAME` or `DOMAIN+USERNAME` format).
    - Please note that this functionality is rarely necessary with latest versions of WinFsp.

## SSH Client configuration

Beyond the basic/default use cases described above, it's important to understand how
SSHFS-Win interacts with both Windows and SSH. When SSHFS-Win uses parts of the SSH
client configuration, it reproduces parts of the SSH client logic and doesn't use `$ ssh
...` itself, so support for SSH client configuration is incomplete and SSHFS-Win doesn't
behave the same as even the `> C:\Program Files\SSHFS-Win\bin\ssh.exe ...` SSH client
included with the SSHFS-Win installation. For example, SSHFS-Win does *not* use [the
`Port ...` ssh_config directive](https://man.openbsd.org/ssh_config#Port). Thus, even if
you've included `Port ...` in your `**/ssh_config`, you must still specify the `!PORT`
suffix in your [UNC](#unc-syntax).

SSHFS-Win re-uses *only* the following [ssh_config
directives](https://man.openbsd.org/ssh_config#DESCRIPTION):

- `Host bar.localdomain` and `Hostname 192.168.1.1`: Define logical host aliases such
  that `\\sshfs\foo@bar.localdomain` connects to `192.168.1.1` on the LAN.

- `IdentityFile C:/Users/foo/.ssh/id.d/id_ed25519`: [Authenticate to the remote with
  specific private SSH key](https://man.openbsd.org/ssh_config#IdentityFile). Note that
  the path *must* be the absolute Windows path to the key.

Because much of how SSHFS-Win uses the SSH client configuration is specific to
SSHFS-Win, it's best to keep such configuration specific to SSHFS-Win in `C:\Program
Files\SSHFS-Win\etc\ssh_config` instead of the more conventional
`C:\Users\foo\.ssh\config` to avoid clashes with [the Windows OpenSSH
feature](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
and/or Cygwin SSH.

## GUI front ends

There are currently 2 GUI front ends for SSHFS-Win: [SiriKali](https://mhogomchungu.github.io/sirikali/) and [SSHFS-Win-Manager](https://github.com/evsar3/sshfs-win-manager).

### SiriKali

[SiriKali](https://mhogomchungu.github.io/sirikali/) is a GUI front end for SSHFS-Win (and other file systems). Instructions on setting up SiriKali for SSHFS-Win can be found at this [link](https://github.com/mhogomchungu/sirikali/wiki/Frequently-Asked-Questions#90-how-do-i-add-options-to-connect-to-an-ssh-server). Please report problems with SiriKali in its [issues](https://github.com/mhogomchungu/sirikali/issues) page.

SiriKali supports:

- Password authentication.
- Public key authentication.
- Key Agents and KeePass 2.

### SSHFS-Win-Manager

[SSHFS-Win-Manager](https://github.com/evsar3/sshfs-win-manager) is a new GUI front end specifically for SSHFS-Win with a user-friendly and intuitive interface. SSHFS-Win-Manager integrates well with Windows and can be closed to the system tray. Please report problems with SSHFS-Win-Manager in its [issues](https://github.com/evsar3/sshfs-win-manager/issues) page.

SSHFS-Win-Manager supports:

- Password authentication.
- Public key authentication.

## Using Jump Hosts

sshfs-win itself does not currently support ssh tunneling, but something similar can be achieved using the built-in openSSH of windows.

- use openSSH t create a local port forward through the jump host to the target
  ```
  ssh -L <origin port of jump connection>:<target of tunnel>:<port of target to target> <adress of tunnel jump host>
  ```
  All standard settings of the [ssh_config](https://man.openbsd.org/ssh_config] may be
  used in this step.

  Reference example ssh config:
  ```
  create the file C:\Users\<UserName>\.ssh\config and/or add the following lines:

  Host <jump host alias>
    Hostname <adress of jump host>
    User <user name at jump host>
    IdentityFile <path to private key for login to the jump host, may have a pass phrase>
    IdentitesOnly yes
  ```
- connect to the target server using the following
  ```
  \\sshfs\REMUSER@localhost!<origin port of jump connection>
  ```
  or similar.


## Advanced Usage

It is possible to use the `sshfs-win.exe` and `sshfs.exe` programs directly for advanced usage scenarios. Both programs can be found in the `bin` subdirectory of the `SSHFS-Win` installation (usually `\Program Files\SSHFS-Win\bin`).

The `sshfs-win.exe` program is useful to launch `sshfs.exe` from a `cmd.exe` prompt (`sshfs-win cmd`) or to launch `sshfs.exe` under the control of the [WinFsp Launcher](https://github.com/billziss-gh/winfsp/wiki/WinFsp-Service-Architecture) (`sshfs-win svc`). The `sshfs-win.exe` program **SHOULD NOT** be used from Cygwin. The `sshfs-win.exe` program has the following usage:

```
usage: sshfs-win cmd SSHFS_COMMAND_LINE
    SSHFS_COMMAND_LINE  command line to pass to sshfs

usage: sshfs-win svc PREFIX X: [LOCUSER] [SSHFS_OPTIONS]
    PREFIX              Windows UNC prefix (note single backslash)
                        \sshfs[.SUFFIX]\[LOCUSER=]REMUSER@HOST[!PORT][\PATH]
                        sshfs: remote user home dir
                        sshfs.r: remote root dir
                        sshfs.k: remote user home dir with key authentication
                        sshfs.kr: remote root dir with key authentication
    LOCUSER             local user (DOMAIN+USERNAME)
    REMUSER             remote user
    HOST                remote host
    PORT                remote port
    PATH                remote path (relative to remote home or root)
    X:                  mount drive
    SSHFS_OPTIONS       additional options to pass to SSHFS
```

The `sshfs.exe` program can be used with an existing Cygwin installation, but it requires prior installation of FUSE for Cygwin on that Cygwin installation. FUSE for Cygwin is included with WinFsp and can be installed on a Cygwin installation by executing the command:

```
$ sh "$(cat /proc/registry32/HKEY_LOCAL_MACHINE/SOFTWARE/WinFsp/InstallDir | tr -d '\0')"/opt/cygfuse/install.sh
FUSE for Cygwin installed.
```

## Passing options to sshfs for mapped network drives

When using mapped network drives created in Windows Explorer or using "net use", you can't directly pass options to sshfs. You can, however, pass them via a registry patch. When you then map a network drive or use "net use", the options are automatically passed in the background. Registry patches for common issues below are provided and serve as an example.

## Preventing timeouts

A connection will timeout after some minutes when nothing is transferred. To prevent this, pass e.g. "-o ServerAliveInterval=30" as SSHFS_OPTIONS. A keep-alive request is sent every 30 seconds.

Map network drive or "net use": Use the provided "ServerAliveInterval.reg" registry patch.

## Setting looser permissions for new files and directories

On a shared file server, the default permissions for new files created may be too strict and prevent others from reading and writing them. To set looser permissions, pass e.g. "-o create_file_umask=0117,create_dir_umask=0007" as SSHFS_OPTIONS. This will allow owner/group read and write permissions on new files and owner/group read, write and execute permissions on new directories.

Map network drive or "net use": Use the provided "GroupReadWrite.reg" registry patch.

## Project Organization

This is a simple project:

- `sshfs` is a submodule pointing to the original SSHFS project.
- `sshfs-win.c` is a simple wrapper around the sshfs program that is used to implement the "Map Network Drive" functionality.
- `sshfs-win.wxs` is a the Wix file that describes the SSHFS-Win installer.
- `patches` is a directory with a couple of simple patches over SSHFS.
- `Makefile` drives the overall process of building SSHFS-Win and packaging it into an MSI.

## Building

In order to build SSHFS-Win you will need Cygwin and the following Cygwin packages:

- gcc-core
- git
- libglib2.0-devel
- make
- meson
- patch

You will also need:

- FUSE for Cygwin. It is included with WinFsp and can be installed on a Cygwin installation by executing the command:

    ```
    $ sh "$(cat /proc/registry32/HKEY_LOCAL_MACHINE/SOFTWARE/WinFsp/InstallDir | tr -d '\0')"/opt/cygfuse/install.sh
    FUSE for Cygwin installed.
    ```
- [Wix toolset](http://wixtoolset.org). This is a native Windows package that is used to build the SSHFS-Win MSI installer.

To build:

- Open a Cygwin prompt.
- Change directory to the sshfs-win repository.
- Issue `make`.
- The sshfs-win repository includes the upstream SSHFS project as a submodule; if you have not already done so, you must initialize it with `git submodule update --init sshfs`.

## License

SSHFS-Win uses the same license as SSHFS, which is GPLv2+. It interfaces with WinFsp which is GPLv3 with a FLOSS exception.

It also packages the following components:

- Cygwin: LGPLv3
- GLib2: LGPLv2
- SSH: "all components are under a BSD licence, or a licence more free than that"
