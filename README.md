# AUR
Arch User Repository packages maintained by D3vil0p3r.

All users are welcome to contribute on keeping them updated.

## Create SSH configuration file
To authenticate to AUR, you need to create a config file `~/.ssh/config` with the following content:
```
Host aur.archlinux.org
  IdentityFile ~/.ssh/aur
  User D3vil0p3r
```
because we must instruct the system to use our AUR keypair to authenticate.

## Setting SSH keys
Create a new SSH key pair:
```
ssh-keygen -f ~/.ssh/aur
```
Get the content of the public key by:
```
cat ~/.ssh/aur.pub
```
and copy the content of the key (except the `<name>@<hostname>` in your AUR account -> My Account -> SSH Public Key section.

## Test AUR package in clean chroot
Install `devtools` package:
```
sudo pacman -S devtools
```
Then, setting up a chroot:
```
mkdir ~/chroot
CHROOT=$HOME/chroot
mkarchroot $CHROOT/root base-devel
```
Note: On btrfs, the chroot is created as a subvolume, so you have to remove it by removing the subvolume by running `btrfs subvolume delete $CHROOT/root` as root.

Building in chroot. Firstly, make sure the base chroot ($CHROOT/root) is up to date:
```
arch-nspawn $CHROOT/root pacman -Syu
```
Then, build a package by calling makechrootpkg in the directory containing its PKGBUILD:
```
makechrootpkg -c -r $CHROOT
```
To build a package with dependencies unavailable from the repositories enabled in `$CHROOT/root/pacman.conf`, pre-install them to the working chroot with `-I package`:
```
 makechrootpkg -c -r $CHROOT -I build-dependency-1.0-1-x86_64.pkg.tar.xz -I required-package-2.0-2-x86_64.pkg.tar.xz
```
Source: https://wiki.archlinux.org/title/DeveloperWiki:Building_in_a_clean_chroot

## Update AUR package
Clone the package to be updated from AUR:
```
git -c init.defaultbranch=master clone ssh://aur@aur.archlinux.org/pkgbase.git
```
When releasing a new version of the packaged software, update the [pkgver](https://wiki.archlinux.org/title/Pkgver) or [pkgrel](https://wiki.archlinux.org/title/PKGBUILD#pkgrel) variables to notify all users that an upgrade is needed. Do not update those values if only minor changes to the [PKGBUILD](https://wiki.archlinux.org/title/PKGBUILD) such as the correction of a typo are being published.

Do not commit mere `pkgver` bumps for [VCS packages](https://wiki.archlinux.org/title/VCS_package_guidelines). They are not considered out of date when the upstream has new commits. Only do a new commit when other changes are introduced, such as changing the build process.

Be sure to regenerate [.SRCINFO](https://wiki.archlinux.org/title/.SRCINFO) whenever `PKGBUILD` metadata changes, such as `pkgver()` updates; otherwise the AUR will not show updated version numbers.

To upload or update a package, [add](https://wiki.archlinux.org/title/Git#Staging_changes) at least `PKGBUILD` and `.SRCINFO`, then any additional new or modified helper files (such as [.install](https://wiki.archlinux.org/title/PKGBUILD#install) files or [local source files](https://wiki.archlinux.org/title/PKGBUILD#source) such as [patches](https://wiki.archlinux.org/title/Patching_packages)), [commit](https://wiki.archlinux.org/title/Git#Committing_changes) with a meaningful commit message, and finally [push](https://wiki.archlinux.org/title/Git#Push_to_a_repository) the changes to the AUR.

For example:
```
makepkg --printsrcinfo > .SRCINFO
git add PKGBUILD .SRCINFO
git commit -m "useful commit message"
git push
```
Tip: To keep the working directory and commits as clean as possible, create a [gitignore(5)](https://man.archlinux.org/man/gitignore.5) that excludes all files and force-add files as needed.

## Create new AUR package
Just need to clone a non-existing package by:
```
git -c init.defaultbranch=master clone ssh://aur@aur.archlinux.org/non-existing-package.git
```
You should get a message like:
```
warning: You appear to have cloned an empty repository.
```
and follow all the instructions in the previous section.

Source: https://wiki.archlinux.org/title/AUR_submission_guidelines
