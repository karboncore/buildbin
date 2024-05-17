# buildbin

A utility for building Arch Linux packages within a container

### About:

  Buildbin is a script for building Arch Linux packages in a container without cluttering the base system installation with build dependencies. This script bind mounts the system or specified root directory as read only, upon which an overlay filesystem is created. A chroot container is launched on the overlay filesystem, within which dependencies can be installed and packages can be built without modification to the underlying base system. Upon exiting the container, packages can be automatically moved into the system cache, added to a local repository, or installed.

### Usage:

    buildbin [options]

    Options:
      -h, --help              show this message and exit
      -u, --user  <name>      username to enter container as. defaults to $SUDO_USER
      -d, --build <path>      path to set up the container. defaults to ./
      -s, --source <path(s)>  source directory to enter container and build packages
                              in. if an existing folder is specified, changes within
                              this directory will persist after exiting; otherwise a
                              temporary directory will be created. if --chroot is
                              called, an external source directory may be specified
                              by providing two paths: first the external directory
                              and second the mountpoint within the chroot
      -m, --makedir           make build directory if it does not exist
      -c, --copy              copy packages to the system pacman cache
      -i, --install           install all packages that were built. implies --copy
      -r, --repo <database>   add packages to a local repository. specify by path or
                              by name
      -b, --bind <path(s)>    absolute paths to bind mount in addition to /. may be
                              called multiple times and mounts must be in order. if
                              --chroot is called, an external mount may be specified
                              by providing two paths: first the external directory
                              and second the mountpoint within the chroot
      -t, --chroot <path>     overlay the specified directory as root instead of /
      -l, --launch            launch buildbin with default options. necessary only if
                              no other options are specified
      --no-erase              do not erase temporary container files on exit
      --force-reset           reset the build directory by removing stale mounts and
                              deleting temporary container files
      --about                 show program description
      --                      enter the container, execute a command, and exit if
                              the return status is zero. any arguments after --
                              will be passed directly to bash and executed as --user

    The author's favorite workflow utilizes a tmpfs build folder and a network
    mounted cache and local repository, with the following command:

    > sudo buildbin -d /tmp -c -r /mnt/pacman/cache/home.db.tar -b /mnt/pacman

### Discussion:

  The main use cases for this program are:

  1) To build packages on your computer without leaving behind build dependencies or system modifications. This script creates an overlay chroot on top of your system and works in the overlay rather than modifying the system itself. Upon exiting the script the overlay is torn down and the host system remains as it was before.

  2) If you prefer to build in a clean chroot with perhaps only base and base-devel installed, you can do that too. Create the base installation somewhere and use the --chroot option to overlay that instead of your system root. Afterwards the chroot installation will be untouched.

  When entering the script you will start in the "source" folder. The --source option allows you to define a pre-existing folder where your work and pkgbuilds are kept. This the only folder where modifications are allowed to persist beyond the script (it's mounted read-write so that you can hold on to your work). If you don't use --source a temporary folder is created instead and packages can still be copied out when exiting. Anything you build should stay in the source folder or it will not get picked up when exiting.

  The "build" folder is just a place for the script to set up the overlay filesystem. Subfolders created by the script within this folder are intended to be temporary, but can be left behind with the --no-erase option if desired. If you have sufficient RAM, best performance can be obtained by placing this on a tmpfs drive.

  By default only the root (or chroot) drive is visible in the container. If you have nested drives you want access to, first mount the drives outside of this script and then use the --bind option to make them visible within the chroot.

  Based on the options you choose, upon exiting the program the packages you built can be automatically installed, copied to cache, or added to a local repository.
