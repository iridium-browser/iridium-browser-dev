# HOWTO compile Iridium on Linux

See also https://code.google.com/p/chromium/wiki/LinuxBuildInstructions and
https://code.google.com/p/chromium/wiki/LinuxFasterBuilds


## Clone iridium-browser-dev

We use a clone of the iridium-browser-dev repository as our base
for developing Iridium.

```bash
git clone https://github.com/iridium-browser/iridium-browser-dev.git
```

## Google depot tools

Make sure to have the [depot tools](https://chromium.googlesource.com/chromium/tools/depot_tools.git) from Google in your path.

## Configure Iridium development

You need a base directory for your Iridium development. You can use the
`iridium-browser-dev` directory from above. For all the following commands,
make sure you are in that directory.

```bash
gclient config --name=src "git+https://git.iridiumbrowser.de/git/iridium-browser"
```

This creates the `.gclient` configuration file so that you can use the depot tools
with Iridium.


## Sync Iridium source

The Iridium source and all dependencies are managed by gclient. This will take
a while when you do it for the first time. You should have around 30 GB
disk space available to do Iridium development.

```bash
gclient sync --with_branch_heads --nohooks
```

This creates a new directory called `src`.


## Install build dependencies

Building requires various packages installed on your system.

`bison brlapi-devel curl dejavu-fonts elfutils ffmpeg-devel flex gcc-c++
git-core git-svn gperf krb5-devel lksctp-tools-devel lsb-release
mozilla-nspr-devel mozilla-nss-devel nodejs openbox pam-devel patch perl
perl-libwww-perl pkg-config pkgconfig(alsa) pkgconfig(atk) pkgconfig(bluez)
pkgconfig(gconf-2.0) pkgconfig(gl) pkgconfig(glib-2.0)
pkgconfig(gnome-keyring-1) pkgconfig(gtk+-2.0) pkgconfig(gtk+-3.0)
pkgconfig(libavcodec) pkgconfig(libavdevice) pkgconfig(libavfilter)
pkgconfig(libavformat) pkgconfig(libavresample) pkgconfig(libavutil)
pkgconfig(libdrm) pkgconfig(libelf) pkgconfig(libffi) pkgconfig(libpci)
pkgconfig(libpostproc) pkgconfig(libpulse) pkgconfig(libswresample)
pkgconfig(libswscale) pkgconfig(libxslt) pkgconfig(openssl)
pkgconfig(pulseaudio) pkgconfig(speech-dispatcher) pkgconfig(sqlite3)
pkgconfig(udev) pkgconfig(xkbcommon) pkgconfig(xscrnsaver) pkgconfig(xt)
pkgconfig(xtst) python python-crypto python-devel python-numpy python-opencv
python-openssl python-psutil python-yaml python3-CherryPy rpm ruby subversion
wdiff xcompmgr zip` (+`texlive-ipa-fonts` maybe)

There is an installation helper script (`src/build/install-build-deps.sh`), but
its scope is very limited and only works for x86/x86_64 systems that use both
apt-get and the Debian package names.


## Prepare building

After you have the build dependencies and source, some third party packages need
to be prepared. This happens by running the gclient hooks.

```bash
gclient runhooks
```

(This downloads toolchains and buildroots from google and this will be
deactivated soon enough…)

A symlink may need to be set for nodejs if DEPS did not specify a download for
node.

    src/third_party/node/linux/node-linux-x64/bin/node -> /usr/bin/node


## Build

You are now ready to build. Ninja is managing the build process. It will
only compile what is required and detects changes automatically. This
produces a `chrome` binary at `src/out/Release/chrome`, which is the
Iridium program.

```bash
ninja -C src/out/Release chrome chrome_sandbox
```

## Install sandbox

To run Iridium, the sandbox needs to be set up. On Linux, see [here](https://code.google.com/p/chromium/wiki/LinuxSUIDSandboxDevelopment) for details. Installing the sandbox requires
sudo permission on your machine.

```bash
BUILDTYPE=Release src/build/update-linux-sandbox.sh
```

## Start it

Whenever you start Iridium, it needs to know the location of the sandbox. It
is conveyed by an environment variable. You can add this to your `.bashrc`, too.

```bash
CHROME_DEVEL_SANDBOX=/usr/local/sbin/chrome-devel-sandbox src/out/Release/chrome
```

## Develop

Make your changes and rebuild. To pull in new changes, go to the `src`
directory and run `git pull` (switch to a branch) and after that, run
`gclient sync` in your base directory again, to make sure third party packages
are updated.

In general, development for Iridium is the same as for Chromium. Thus, if you
are familiar with that, you are ready to go.


## Use a temporary profile

Sometimes, it is useful to use a temporary profile.
```bash
CHROME_DEVEL_SANDBOX=/usr/local/sbin/chrome-devel-sandbox src/out/Release/chrome --user-data-dir=/tmp/iridium-temp-profile
```

## Updating

The Iridum source master branch is constantly rebased onto the Chromium upstream.
Therefore, to update, you need to hard-reset the `src` directory to `origin/master` before
running gclient.

```bash
cd src
git fetch
git reset --hard origin/master
cd ..
gclient sync --with_branch_heads --nohooks
```
