# sudo-1.8.3p1-patched

This is a custom version of sudo, based on the sudo 1.8.3p1 package as provided by Canonical for Ubuntu 12.04 using the URLs below, with the [CVE-2021-3156](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-3156) patches applied.

- http://us.archive.ubuntu.com/ubuntu/pool/main/s/sudo/sudo_1.8.3p1-1ubuntu3.7.dsc
- http://us.archive.ubuntu.com/ubuntu/pool/main/s/sudo/sudo_1.8.3p1.orig.tar.gz
- http://us.archive.ubuntu.com/ubuntu/pool/main/s/sudo/sudo_1.8.3p1-1ubuntu3.7.debian.tar.gz

(The easiest way to download and unpack these files is by using `apt-get source sudo` on a machine running Ubuntu 12.04. If you do not wish to mess with a production machine, this [Vagrant](https://www.vagrantup.com/) image was helpful to me in making this: https://app.vagrantup.com/hashicorp/boxes/precise64)

## Symptoms of the issue

If you have the unpatched version installed (`1.8.3p1-1ubuntu3.7`), it typically behaves like this, indicating the vulnerability which could potentially be exploited:

```
vagrant@precise64:~/src$ sudoedit -s '\' `perl -e 'print "A" x 65536'`
Segmentation fault
```

The patched version will give you something like this instead:

```
vagrant@precise64:~/src$ sudoedit -s '\' `perl -e 'print "A" x 65536'`
usage: sudoedit [-AknS] [-C fd] [-D level] [-g groupname|#gid] [-p prompt] [-u user name|#uid] file ...
```

## Step-by-step description on how this patched version was made

- `apt-get source sudo`
- `cd sudo-1.8.3p1`, apply all the CVE-2021-3156 patches from the Xenial package. The easiest way to get these packages is by running `apt-get source sudo` on a Xenial box, or by adding the Xenial `apt-get` package source to your Precise box temporarily: `deb-src http://security.ubuntu.com/ubuntu xenial-security main`.
  - `patch -p1 < debian/patches/CVE-2021-3156-1.patch` (3 out of 9 hunks failed)
  - `patch -p1 < debian/patches/CVE-2021-3156-2.patch` (refers to nonexistent file)
  - `patch -p1 < debian/patches/CVE-2021-3156-3.patch` (1 out of 4 hunks failed)
  - `patch -p1 < debian/patches/CVE-2021-3156-4.patch` (refers to nonexistent file)
  - `patch -p1 < debian/patches/CVE-2021-3156-5.patch` (1 out of 1 hunk failed)
- CVE-2021-3156-2 and CVE-2021-3156-4 refers to fixes in plugins (files in `plugins/sudoers`) not present in the 1.8.3p1 version. It is my sincere belief that since this code does not exist in that version, these patches are not applicable in our case.
- For the other failing hunks, look at the `.rej` files and apply the changes manually. This is the hard part, but it's sometimes not so hard. The reason for the patches not applying cleanly is typically because the line numbers differ too much (the patched line(s) are present, but in a different section of the patched file), or that the code looks differently in the 1.8.3p1 code base. Errors like this are also possible to resolve; they just require a bit more thinking, analyzing the code and figuring out how to apply the changes.

## Building the package from this repository

```shell
$ sudo apt-get build-dep sudo
$ dpkg-buildpackage -rsudo -b
```

If all goes well, you should now get a `sudo_1.8.3p1-1ubuntu3.8_amd64.deb` and `sudo-ldap_1.8.3p1-1ubuntu3.8_amd64.deb` package in the parent directory.

## VERY IMPORTANT NOTE

**Note that ONLY the CVE-2021-3156 patches were applied to this tree.** It's quite possible that one or more of the following Xenial patches would also be applicable to this tree; I did an initial attempt to apply them but they didn't apply cleanly, and thus, they are not included in this repository. There is ABSOLUTELY NO WARRANTY that the `sudo` package provided by this repository does not have critical security issues.

Pull requests that applies any of these patches are _highly welcome_! For your convenience, the full set of Xenial patches (as of sudo `1.8.16-0ubuntu1.10`) is available in [debian/patches](debian/patches) in this tree. Do note that the Precise (12.04) package sources (which this repository is based on) has already applied everything up to `CVE-2014-9680.patch`. The full set of patches already applied are listed in [.pc/applied-patches](.pc/applied-patches).

## Disclaimer

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
