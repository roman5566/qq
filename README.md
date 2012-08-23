NOTE
====
Currently this is a design document. It amazes me nobody has made such a tool as this yet. Instead we have a million alternatives to autotools (cmake,
scons, Jam, Qmake, BoostJam, etc.) but none of them address the wider issues
that packaging software sucks and consists of vastly duplicated and wasted effort.

So what I am saying is: it's amazing nobody has done this, so maybe it is much
harder than I think it is. And that's why.

qq
==
qq is an opinionated, package-manager-aware build-tool that replaces autotools
and generates Makefiles.

* qq is opinionated:
  Through convention over configuration, you don't have to (though you may)
  write any build-tool configuration files.
* qq is able to install dependencies for you:
  qq tries to find the dependency on your system and is package-manager
  aware. Failing that it will download and install dependencies from source
  into /usr/local, ~/local or ~/.local whichever is available.
* qq is useful to package-managers:
  Package managers can download source packages that use qq and use the
  qq to get a stable JSON response that contains dependency information
  in a form they can use. Packaging maintainers can concentrate on integrating
  packages into their system rather than worrying about duplicating dependency
  determination information and naming-information. qq becomes the
  defacto package-name database.

qq is not a package manager. It will not do updates or upgrades. It is meant
to facilitate installing software from source for end-users and to facilitate
packaging for packagers.

Contributing
============
Please provide ideas via the issue tracker. We'll add them here as the design
proceeds.

Specifying Targets
==================
* Each target is in its own directory.
* Libraries start with lib, binaries do not.
* Binaries must have a version after, eg. foo-1.2. The version must be
  able to be parsed by qq's version-comparator code. We support various
  version strings. Versions can be groups of [0-9a-zA-Z] separated by single
  dots. The comparator is case insensitive. Letters are shunned by allowed.
* Libraries must have a valid soname extension (see example layout).
* Libraries directories are automatically in the includepath for all other
  targets in the project
* Target names can contain almost any characters. We draw the line at forward
  and backslashes.
* The target's source files are in its directory. Anything in here
  (recursively) is compiled individually and then all object files are linked
  to make the target.
* For the *far* more common use case (build-once and install) all source files
  a compiled and linked in one-step. All modern compilers support this and 
  it's way faster, though resource intensive.

Code Generation
===============
* FOO_VERSION defines are automatically created and made available to the
  project, for libfoo, installation adds these defines to any foo.h that is
  installed.

Installing
==========
* foo.h is automatically installed if it exists. Other headers are
  automatically installed too, for example (given libfoo-1.2.3)
  INSTALL_PREFIX/include/foo/1.2.3/. Symlinks for the latest version are
  created in INSTALL_PREFIX/include/foo/. foo.h will be in
  INSTALL_PREFIX/include. qq will warn if no other headers are included
  from foo.h, but which ones get included is up to the library author.
    
Developer Convenience
=====================
* qq provides macros for version visibility etc.

Configure
=========
* configure is a minimal /bin/sh script that works anywhere and does minimal
  steps. We include this because people are used to it
* may not be necessary since convention over configuration can determine most
  stuff
* configure installs qq if necessary
* configure can run pre-build preparations once qq is installed

Dependencies
============
* Dependencies are specified in one sourcefile in the target as comments
* For binary targets this must be main.c
* For libraries this must be main.c or for libfoo, foo.c
* For an example see foo-0.9/main.c
* Dependencies on other projects are installed before hand either /usr/local
  or ~/local or ~/.local (whichever exists, if both exist errors)
* If dependency is qq compatible then great, otherwise we revert to standard
  Homebrew formula. We use the qq naming DB to lookup the formula name
  and then clone the tap for that platform. Supporting all dependencies for
  each platform is doable, most formula we have (in MacHomebrew < 1.0) are not
  dependencies. Long term we hope developers will use qq and thus make
  the formula-taps unecessary. This is the point in qq, if it's not
  clear yet.
* pkg-config is not used, though they are automatically generated. pkg-config
  is made unecessary by qq's convention over configuration approach

Build Process
=============
* Final products go in ROOT, byproducts in ROOT/.o
* configure determines if qq needs to be installed to .local and if a
  mini-ruby for user's platform needs to be installed there too.
* Compile flags are specified in the header for each source file. So you can
  have optimizations specific to each source file if you so choose.
* Compile flags for the target as a whole are specified in main.c.
* Compile flags can be specified on a platform-by-platform basis.
* Generalised flags that are mapped by qq to the slightly different
  flags for different compilers will be added as we determine them.

Caveats
=======
* Only supports modern compilers (at first at least)

qq is Ruby
================
We'll build it in Ruby. When it's so amazing that it works 99% of the time we
can port it to c (if still desired). And then we're portable like make etc.

Brew Naming Database
====================
First come, first served. No duplicates. Obviously you can call your targets
what you want, but if you pick something already picked, you won't get into
the brew database. You can register a name, but we'll only allow you to squat it for a month, you need to show some code in that time or it will be
released.

TOTHINK: Would be nice if this was decentralised (eg. using URLS as keys), but ATEOD we install on unix where names must be unique in each prefix.

Variants
========
* Software can be built differently based on variants. Specified like so:
    ./configure +foo -bar
* Variants are determined from the README in the project. This way they are
  documented just once. Obv. this requires the developer to document variants
  in a standard way. So bonus.
* Some variants may be determined by the configure script.
* Other formula can depend on software built with variants, or conflict with
  it, etc.

Homebrew-style Kegs
===================
* Sure, why not. Will work well on Mac and Linux. Better on mac though as Mac
  LD is more flexible (thanks to install_names).

Developer Responsibility
========================
* Conform to qq conventions in source tree.
* Provide the following items in your homepage's index.html HEAD element:
    <link rel="pkg.latest" href="/downloads/foo-0.1.tgz" />
    <meta name='pkg.description' content='A CLI-tool for fooing the bar.'>

* Private a conforming downloads directory:
  http://homepage.com/downloads/foo-0.1.tgz
* If you source is on Github we can figure out for ourselves if there is
  something new. So host it there :P
  
End-User Usability
==================
Autotools and that are hard to customise well. If you want a base level of
optimisation then setting CFLAGS is a maybe. The generated Makefile may or may
pay attention to their whim. qq pays attention. The fact that variants 
are intrinsically understood at the make-level will help with such usability.


TO SOLVE
========
* When builds break, generate good log and tell user where to submit report
* Allow developer to not support certain compilers, but somehow force them to
  reassess support every now and then. Or the support is never enabled. Force
  dev to say why they don't support it maybe.
* If developer won't fix issues users need a way to provide patches or
  whatever.


Consider
========
pkg-config for everything. pkg-config file in standard URL locations. Frankly maybe a sound idea. It's a standard, despite being a little messy, but properly written pc files are perfect.


Example User Flow
=================
* User types: qq
* qq finds two targets, c2 depends on c1. c1 depends on 'github.com/a/b == 0.9'
* qq fetches github.com/a/b/downloads/tags/0.9
* qq installs 'b-0.9' to ~/local/clr/b/0.9 and symlinks to ~/local
* qq generates makefile
* user runs make
* make builds c1, then c2
* user runs make install
* 'make' installs to ~/local
