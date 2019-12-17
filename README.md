# Re-enable_rc6

*Confirmed working with kernel __5.4.3__*

**current version does NOT work with kernels <5.4**

Since linux 4.16 the i915.enable_rc6 parameter has been disabled. This patch re-enables it so your computer won't crash. See https://gitlab.freedesktop.org/drm/intel/issues/100 for some background and information (although, if you're here you probably already know all that!). This patch is basically just the output of git revert on [this](https://github.com/torvalds/linux/commit/fb6db0f5bf1d4d3a4af6242e287fa795221ec5b8) commit of the master [linux](https://github.com/torvalds/linux/) branch. Slight modifications were done by hand to try to minimally affect all progress since 4.15.16.

## Getting Started

Just download the patch and use it. See below for two ways you might be compiling your kernel.

### From source

**!NOTE!** Since 5.3 the patch only works on the version tagged branch of the main kernel (currently 5.4).

```
$ git clone https://github.com/torvalds/linux.git
$ cd linux
$ git checkout v5.4
$ patch -p1 < re-enable_rc6.patch
```

And then make the kernel as you normally do.

### Arch Linux

Most of this is ganked from the [Arch Wiki](https://wiki.archlinux.org/index.php/Kernel/Arch_Build_System). You'll need the `asp` and `pacman-contrib` packages and the `base-devel` group.

```
$ mkdir build
$ cd build
$ asp update linux
$ asp checkout linux
$ cd repos/core-[ARCH]
```

Copy the patch to this directory and edit `PKGBUILD` to include the patch at the end of the `source=` statement. At this point also change the `pkgbase` variable to give your custom kernel a different name. Here's the first few lines of my `PKGBUILD`:

```
# Maintainer: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Maintainer: Tobias Powalowski <tpowa@archlinux.org>
# Maintainer: Thomas Baechler <thomas@archlinux.org>

pkgbase=linux-rc6       # Build kernel with a different name
pkgver=5.4.3.arch1
pkgrel=1
arch=(x86_64)
url="https://git.archlinux.org/linux.git/log/?h=v$_srcver"
license=(GPL2)
makedepends=(xmlto kmod inetutils bc libelf git python-sphinx graphviz)
options=('!strip')
_srcname=archlinux-linux
source=(
  "$_srcname::git+https://git.archlinux.org/linux.git?signed#tag=v$_srcver"
   config         # the main kernel config file
   re-enable_rc6.patch # renable i915.enable_rc6 kernel parameter
)
...
```

Now run
```
$ updpkgsums
```
to update the checksums in `PKGBUILD`. You can now compile the kernel with
```
$ makepkg -sr
```
I like `-sr` to install needed build dependencies and then remove them after building is complete.

Once that's done install the package (and probably also the `docs` and `headers` packages) with `pacman -U`.

Finally, update your bootloader so that you can select the new kernel when you restart your computer.

## Versions

If/when the patch is updated to work with newer versions of the linux kernel I will tag those changes with the kernel version I used to test the patch. Thus you can get the version you need with `git checkout`; just find the tag with the closest version that is below your version. For example, the 4.20 update breaks for anything less than 4.20, so if you want to compile 4.19.9 then you'd need to do
```
$ git checkout 4.18.7
```
## Notes

If you have the option of selecting your video drivers then I recommend using the official Intel driver (e.g., xf86-video-intel on Arch). When using other drivers or relying solely on Kernel mode setting I experience non-critical, but annoying bugs with my display. For example, sometimes after waking from sleep the display remains black until I switch to an unused virtual terminal and then back again. I don't know if this behavior is related to this patch or the rc6 parameter, but maybe it is and is easily avoided (at least in my case) by simply using the Intel driver.

## Questions/Contributions

This patch was made to fix my computer with the slim hope that others may benefit from the fix as well. As such I offer no promises that will work for you or that I will keep the patch up-to-date. That said, I like having an up-to-date kernel and any changes to the patch will be pushed here. If you have any contributions or suggestions I would be happy to read/incorporate them!

## Author

Arthur Eigenbrot - aeigenbrot at nso.edu
