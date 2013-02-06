Gumstix Repo Manifests for Yocto Build System
=============================================
This repository provides Repo manifests to setup the Yocto build system for
the Gumstix products.

***
**Note**
If you already have a Yocto setup and want only the Gumstix BSP layer, use
the meta-gumstix repository found here:
git://github.com/ashcharles/meta-gumstix.git.
***

Yocto allows the creation of custom Linux distributions for embedded systems
including Gumstix-based systems.  It is a collection of git repositories known
as **layers** each of which provides **recipes** to build software packages as well
as configuration information.

Repo is a tool that enables the management of many git repositories given a
single **manifest** file.  Tell Repo to fetch a manifest from this repository and
it will fetch the git repositories specified in the manifest and, by doing so,
setup a Yocto build environment for you!

Getting Started
---------------
**1.  Install Repo.**

Download the Repo script.

    $ curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > repo

Make it executable.

    $ chmod a+x repo

Move it on to your system path.

    $ sudo mv repo /usr/local/bin/

If it is correctly installed, you should see a Usage message when invoke it
with the help flag.

    $ repo --help

**2.  Initialize a Repo client.**

Create an empty directory to hold your working files.

    $ mkdir yocto
    $ cd yocto

Tell Repo where to find the manifest

    $ repo init -u git://github.com/ashcharles/Gumstix-YoctoProject-Repo.git 

A successful initialization will end with a message stating that Repo is
initialized in your working directory. Your directory should now
contain a .repo directory where repo control files such as the manifest are
stored but you should not need to touch this directory.

***
**Note**
You can use the **-b** switch to specify the branch of the repository
to use.  We develop on the guaranteed-to-break **dev** branch.  The
**master** branch should at least compile.

The **-m** switch selects the manifest file (default is *default.xml*).
Our default.xml on **master** is designed to be stable as it *pins*
particular commits.

To test out the bleeding edge, type:

    $ repo init -u git://github.com/ashcharles/Gumstix-YoctoProject-Repo.git -b dev
    $ repo sync

To get back to the known stable version, type:

    $ repo init -u git://github.com/ashcharles/Gumstix-YoctoProject-Repo.git -b master
    $ repo sync

To learn more about repo, look at http://source.android.com/source/version-control.html 
***

**3.  Fetch all the repositories.**

    $ repo sync

Now go turn on the coffee machine as this may take 20 minutes depending on
your connection.

**4.  Initialize the Yocto Environment.**

    $ TEMPLATECONF=meta-gumstix-extras/conf source ./poky/oe-init-build-env

This copies default configuration information into the **poky/build/conf**
directory and sets up some environment variables for Yocto.  This configuration
directory is not under revision control; you may wish to edit these configuration
files for your specific setup.

**5.  Build an image.**

This process downloads several gigabytes of source code and then proceeds to
do an awful lot of compilation so make sure you have plenty of space (25GB
minimum), and expect a day or so of build time depending on your network
connection.  Don't worry---it is just the first build that takes a while.

    $ bitbake gumstix-console-image

If everything goes well, you should have a compressed root filesystem
tarball as well as kernel and bootloader binaries available in your
**work/deploy** directory.  If you run into problems, the most likely
candidate is missing software packages.  Check out
http://www.yoctoproject.org/docs/current/yocto-project-qs/yocto-project-qs.html#resources
for the list of required packages for operating system. Also, take
a look to be sure your operating system is supported:
https://wiki.yoctoproject.org/wiki/Distribution_Support

Staying Up to Date
------------------
To pick up the latest changes for all source repositories, run:

    $ repo sync

Enter the Yocto environment:

    $ source poky/oe-init-build-env

If you forget to setup these environment variables prior to bitbaking,
your OS will complain that it can't find bitbake on the path.  Don't try
to install bitbake using a package manager, just run the command above.

You can then rebuild as before:

    $ bitbake gumstix-console-image

Starting from Fresh
-------------------
So something broke...

There are several degrees of **starting fresh**; individual packages can be
rebuilt or the whole system can be reconstructed.

 1. clean a package: bitbake <package-name> -c cleansstate
 2. re-download package: bitbake <package-name> -c cleanall
 3. destroy everything but downloads: rm -rf build (or whereever your sstate and work directories are)
 4. destroy it all (not recommended): rm -rf build && rm -rf sources

***
**Note**
If you've made a change to a recipe and want the package to be rebuilt, just
increment the recipe version (the **PR** variable); cleaning is not necessary.

To understand better how bitbake processes recipes, look at the excellent
documentation http://www.yoctoproject.org/docs/current/poky-ref-manual/poky-ref-manual.html.
***

To make sense of the differences between these cleaning methods, it is useful
to understand that Yocto caches both the downloaded source files for all the
packages it tries to build (the **DL_DIR** configuration parameter) and the
packages once built (the **SSTATE_DIR** configuration parameter).  Typically,
deleting the downloaded source is a bad idea---this just means re-fetching
gigabytes of code which wastes network bandwidth.  Cleaning the sstate cache
for a particular package ensures that it actually gets rebuilt from source
rather than simply restored from the cache. 

Customize
---------
Sooner or later, you'll want to customize some aspect of the image either
adding more packages, picking up some upstream patches, or tweaking your kernel.
To this end, you'll want to customize the Repo manifest to point at different
repositories and branches or pull in additional meta-layers.

Clone this repository (or fork it on github):

    $ git clone git://github.com/ashcharles/Gumstix-YoctoProject-Repo.git

Make your changes (and contribute them back if they are generally useful :) ),
and then re-initialize your repo client

    $ repo init -u <file:///path/to/your/git/repository.git>

