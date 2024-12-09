# tv-box-pc-demo


## What are deb packages?

This folder have examples for creating deb packages, a deb package is a single compressed file with software artifacts structured to be placed on a target system.
Once you have the name of the package, it can be installed using dpkg by running dpkg -i /path/to/file.deb. dpkg itself only installs the provided deb file. To install the deb and other required dependencies in a easy way, you can use apt. This is a python script that automatically download the required dependencies and calls dpkg to install them, it has other features like upgrading all packages installed in a system.

The structure of a typical repository for packaging into deb are the following:

```
xed_2.6.2+ulyana_amd64
├── DEBIAN
│   ├── control
│   └── md5sums
└── usr
    ├── bin
    │   └── xed
    ├── lib
    │   └── x86_64-linux-gnu
    │       └── xed
    │           ├── girepository-1.0
    │           │   └── Xed-1.0.typelib
    │           ├── libxed.so
    │           └── plugins
    │               ├── docinfo.plugin
    │               ├── filebrowser.plugin
    │               ├── joinlines
    │               │   └── __init__.py
   ...             ...
    │               └── wordcompletion.plugin
    └── share
        ├── applications
        │   └── xed.desktop
        ├── dbus-1
        │   └── services
        │       └── org.x.editor.service
        ├── doc
        │   └── xed
        │       ├── changelog.Debian.gz
        │       ├── changelog.gz
        │       └── copyright
        └── man
            └── man1
                └── xed.1.gz 
```

Observe that the artifacts are disposed in a structured directory tree, and the root location inside the package matches the root of the target filesystem where the package will be installed.
In this package example, the elements will be placed in the following locations:
xed_2.6.2+ulyana_amd64
    /usr/bin/xed                    -> an executable
    /usr/lib/x86_64-linux-gnu/xed/  -> libs, plugins
    /usr/share/applications/        -> .desktop entry file, (desktop or start menu icon)
    /usr/share/dbus-1/services/     -> D-BUS Service
    /usr/share/doc/xed/             -> copyright and changelog files
    /usr/share/man/                 -> manual pages

The DEBIAN folder will not be installed, but are used in the installation. Lets take a look inside it:
It has the file md5sums that have the md5 hash of all files present in the package.
The control file are used in the installation for dependency management, it also contains description of the package, like section on debian apt repository, maintainer contact, etc, you can use apt show <package-name> for print this file.
It can have installation scripts for processing pre or post installation requirements, like restarting services, cheching configuration files, etc.

The installation proccess consist of decompressing the deb package to the root of the target filesystem and merging its contents, directories will be created, if necessary.

### deb source packages:
Source packages are the source with all the elements to construct the package. It are distributed apart of binary packages(need to be specified in sources.list file) and use apt source <package-name> to get it.
The source contains at least two elements:
 - The .dsc file that contains information about the elements inside the package, it can be generated using the code of debian folder inside the compressed code;
 - A .tar compressed file containing all the code of the software including the debian folder with the information to construct the deb package. This code must be constructed in a way that it can generate the artifacts of the software, that will be placed inside the deb package.

To use this source package for building a binary deb, it is needed to be named in accordance with a naming convention, for example <package-name>_<version>.tar.gz are the source and <package-name>-<version>/ are the repository (note the change of "_" for "-"). The binary deb package will be named as follwing <package-name>_<software-version>-<debian-source-version>_<architeture>.deb where <debian-source-version> are the version of the deb souce package itself.

In deb-packaging/examples/example-packages/xed_source we have a example of a source package. The .tar.xz and .dsc files can be obtained by running apt source <package-name>. Inside the repository folder we have structured files including debian folder, inside this folder there are the control file that describes the package itself, it can be printed using apt showsrc <package-name>. 

## dpkg build tools:
FIXME To construct a deb package from a source code, this code repository need to be structured in a way that the build system can use it to build the package.
The tools present by default in deb package managment are listed below:
 - `dpkg`: The Core Package Management Binary, manages installation, removal, and querying of `.deb` packages;
 - `dpkg-deb`: Builds the `.deb` binary package directly from a directory structure;
 - `dpkg-buildpackage`:  is a wrapper written in Perl language that runs the entire packaging process, calling `dpkg-*` tools, and other tools to generate both source and binary packages. It creates both `.deb` and source (`.dsc`, `.orig.tar.gz`, `.diff.gz`) packages. It supports `debuild`, `pbuilder` and other build systems that add other features like isolation, dependency resolution, and signing;
 - Other tolls:
  - `dpkg-source` handles the creation and unpacking of Debian source packages (`.dsc` files and source tarballs).
   - `dpkg-source -x <package.dsc>` extracts source code and applies patches.
   - `dpkg-source -b <source-dir>` creates `.dsc`, `.orig.tar.gz`, and `.diff.gz` files needed for source packages.
  - `dpkg-gencontrol`: It reads metadata `debian/control` in the source package and generate the `DEBIAN/control` for binary package.


### Creating a source package
First step in debian packaging is creating a debian folder with files that describes the information needed to configure the packaging construction and usage.
The debian folder are in root of the source repository, dh-make are a utility to create examples of configuration files. Or debmake.
#TODO: explarin better the creation of these files


### Base images for this project:
 - C/C++: bitnami/minideb:bookworm (libc)
 - Java/Kotlin with maven: https://hub.docker.com/_/maven
 - Golang: golang/golang:alpine3.20
 - Javascript/Node.js #TODO: https://www.npmjs.com/package/node-deb

### Portable environment: use of containers
Containers seems more suitable for building packages.
Building a software form a source in a native way, (i.e. from a OS, getting the source code, installing the build envyronment, calling the build automation) can lead to some problems:
 1. Need to be a privileged user for install build envyronment, sometimes it is not possible;
 2. Dependencies pulled alongside with build envyronments can bloat system software, many of them are not strictly necessary on build proccess;
 3. Dependencies are indisponible or deprecated on default package repository, correcting this by modifying repository sources can lead to breaking instaled software;
 4. Duplicate versions of libraries, compilers, VMs (java);
 5. Need of specific distribution and version;
 6. Need of porting the build envyronment to a unsupported distribution version;
 7. A guest OS can be easily configured on top of a host OS using a VM, it will be like a native way, but it is costly because of the instalation of a complete system and the running performance.
 
Containers address every issue listed above by providing:
 1. Normal users can manage containers;
 2. (Multi-stage)[https://docs.docker.com/build/building/multi-stage/] can be used in a build proccess, it can avoid installing many dependencies and result in smaller container size and faster proccess.
 3. Can be constructed using any version of dependency and are a isolated envyronment from host system;
 4. Can be built and discarted after use;
 5. Base system can be any distribution and version;
 6. Can be used in any container capable system, building it again is not mandatory;
 7. Small and practly (native performance)[https://stackoverflow.com/a/26149994/4086871]

Limitations of containers in building software:
 - One limitation are when building software that requires specific kernel version or configuration (modules), altrough the backyward compatibility of kernel exists, it is not guaranted for all situations. In this case the most convenient way are using a VM.


# Creating packages from examples

## Requirements: 
 - amd64 or arm64 architeture
 - Docker installation

You can use provided Makefile, in this repository to help build each package, `make` will print it.

# Examples description:
In the package-examples folder we have already built packages.
In the examples folder there are Dockerfiles that represent a container creation, more than that, they are intended to be used in the end-to-end building. They are constructed using Docker best practices like Multi-stage builds. The general stages of a build pipeline are:
```
│────────────────────│────────────────────────────────────────│────────────────────────│
│ get the structured │ build artifacts using the project's    │ do the packaging of the│
│ source             │ build system                           │ generated artifacts    │
│────────────────────│────────────────────────────────────────│────────────────────────│
```

The Dockerfile are a text file that contains commands for building the image, it takes a base image and can modify its state with standard tools this base image provides.
Once created, the image can be run or pushed to a repository, to be used later. 
To run the build process with the provided examples, you can use the provided Makefile, or the following commands:

docker build -t <package-name> -f <examples/Dockerfile.<package-name>> .
docker run -v $PWD/examples/output:/root/output <package-name>

The deb package will be stored in the folder examples/output

# References:
- https://wiki.debian.org/Packaging
- https://github.com/streadway/hello-debian/blob/master/README.md
- https://lazyfrosch.github.io/training-debian-packaging/dev/dh_make/
- https://blog.packagecloud.io/how-to-build-debian-packages-for-simple-shell-scripts/

