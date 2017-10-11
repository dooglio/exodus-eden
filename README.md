# exodus-eden-debian-packager
This contains the necessary code to create an Ubuntu Debian package for the Exodus Eden GUI

This archive has a debian folder, and CMake is required to create a working makefile.
The version is parsed from the debian/changelog via CMake, and this helps construct
the appropriate postinst file, which uses exodus-linux-installer to download.

Presently there is no PPA nor debian source servers available, but this may come at a later date.
