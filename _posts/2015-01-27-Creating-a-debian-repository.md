---
title: Creating a debian repository
---

So I have a `.deb` file, created with [CPack], that I want to make available on
a public server, so it can be installed on several hosts easily.

It is quite easy to find some documentation on the internet about the various pieces of the
process, but a bit more difficult to find the whole process, including the the
signature and the verification of the package, that avoid the warning:

    WARNING: The following packages cannot be authenticated!
      foo bar
    Install these packages without verification [y/N]?

or the use of the `--allow-unauthenticated` option.

So here is the process:

* First, create a GPG key, if needed

~~~~bash
gpg --gen-key
~~~~

* Prepare an empty directory and put the packages in it:

~~~~bash
mkdir deb-packages
cp *.deb deb-packages
cd deb-packages
~~~~

* Sign the packages

~~~~bash
dpkg-sig --sign builder *.deb
~~~~

* Export the public key in the directory (the key can also be exported to a key
server)

~~~~bash
gpg -a --export > great-developer.asc
~~~~

* Generate the `Packages` and `Packages.gz` files that contains a description of
the packages

~~~~bash
dpkg-scanpackages . /dev/null | tee Packages | gzip > Packages.gz
~~~~

* Generate the `Release` file that contains the md5 sum of the packages, `Packages`
and `Packages.gz` files

~~~~bash
apt-ftparchive release . > Release
~~~~

* Sign the `Release` file, so we can ensure that everything is done by the
expected developer (me)

~~~~bash
gpg --yes --armor --output Release.gpg --detach-sig Release
~~~~

* Copy all the files on a server where it will be publicly available

~~~~bash
cd ..
rsync -av deb-packages/ myserver:/htdocs/deb-packages/
~~~~

Now on the host where the repository should be used, run

~~~~bash
echo "deb http://myserver.com/deb-packages/ ./" >> /etc/apt/sources.list
wget -O - http://myserver.com/deb-packages/great-developer.asc | apt-key add -
apt-get update
~~~~

Done! The packages can be installed, with a simple `apt-get install`.

[CPack]: http://www.cmake.org/Wiki/CMake:Packaging_With_CPack
