---
layout: docs
title: Configuring crosstool-NG
permalink: /docs/configuration/
---

crosstool-NG is configured with a configurator presenting a menu-structured
set of options. These options let you specify the way you want your toolchain
built, where you want it installed, what architecture and specific processor
it will support, the version of the components you want to use, etc.
The value for those options are then stored in a configuration file.

The configurator works the same way you configure your Linux kernel. It is
assumed you now how to handle this.

To enter the menu, type:

    ct-ng menuconfig

Almost every config item has a help entry. Read them carefully.

String and number options can refer to environment variables. In such a case,
you must use the shell syntax: `${VAR}`. You shall neither single- nor double-
quote the string/number options.

There are three environment variables that are computed by crosstool-NG, and
that you can use:

-   `CT_TARGET`: Represents the target tuple you are building for.
    You can use it for example in the installation/prefix directory,
    such as: `/opt/x-tools/${CT_TARGET}`.

-   `CT_TOP_DIR`: The top directory where crosstool-NG is running.
    You shouldn't need it in most cases. There is one case where you
    may need it: If you have local patches and you store them in your
    running directory, you can refer to them by using `CT_TOP_DIR`,
    such as: `${CT_TOP_DIR}/patches.myproject`.

-   `CT_VERSION`: The version of crosstool-NG you are using. Not much
    use for you, but it's there if you need it.


Interesting config options
--------------------------

-   `CT_LOCAL_TARBALLS_DIR`: If you already have some tarballs in a
    directory, enter it here. That will speed up the retrieving phase,
    where crosstool-NG would otherwise download those tarballs.

-   `CT_PREFIX_DIR`: This is where the toolchain will be installed in
    (and for now, where it will run from). Common use is to add the
    target tuple in the directory path, such as (see above):
    `/opt/x-tools/${CT_TARGET}`.

-   `CT_TARGET_VENDOR`: An identifier for your toolchain, will take
    place in the vendor part of the target tuple. It shall *not*
    contain spaces or dashes. Usually, keep it to a one-word string,
    or use underscores to separate words if you need. Avoid dots,
    commas, and special characters.

-   `CT_TARGET_ALIAS`: An alias for the toolchain. It will be used as
    a prefix to the toolchain tools. For example, you will have
    `${CT_TARGET_ALIAS}-gcc`.

Also, if you think you don't see enough versions, you can try to enable one of
those:

-   `CT_OBSOLETE`: Show obsolete versions or tools. Most of the time,
    you don't want to base your toolchain on too old a version (of
    gcc, for example). But at times, it can come handy to use such an
    old version for regression tests. Those old versions are hidden
    behind `CT_OBSOLETE`. Those versions (or features) are so marked
    because maintaining support for those in crosstool-NG would be too
    costly, time-wise, and time is dear.

-  `CT_EXPERIMENTAL`: Show experimental versions or tools. Again, you
    might not want to base your toolchain on too recent tools
    (e.g., gcc) for production. But if you need a feature present only
    in a recent version, or a new tool, you can find them hidden
    behind `CT_EXPERIMENTAL`. Those versions (or features) did not
    (yet) receive thorough testing in crosstool-NG, and/or are not
    mature enough to be blindly trusted.


Re-building an existing toolchain
---------------------------------

If you have an existing toolchain, you can re-use the options used to build it
to create a new toolchain. That needs a very little bit of effort on your side
but is quite easy. The options to build a toolchain are saved with the
toolchain, and you can retrieve this configuration by running:

    ${CT_TARGET}-ct-ng.config

An alternate method is to extract the configuration from a `build.log` file.
This will be necessary if your toolchain was build with crosstool-NG prior to
`1.4.0`, but can be used with build.log files from any version:

    ct-ng extractconfig <build.log >.config

Or, if your `build.log` file is compressed (most probably!):

    bzcat build.log.bz2 | ct-ng extractconfig >.config

The above commands will dump the configuration to `stdout`, so to rebuild a
toolchain with this configuration, just redirect the output to the `.config`
file:

    ${CT_TARGET}-ct-ng.config >.config
    ct-ng oldconfig

Then, you can review and change the configuration by running:

    ct-ng menuconfig


Using as a backend for a build-system
-------------------------------------

Crosstool-NG can be used as a backend for an automated build-system. In this
case, some components that are expected to run on the target (e.g., the native
gdb, ltrace, DUMA …) are not available in the menuconfig, and they are not
build either, as it is considered the responsibility of the build-system to
build its own versions of those tools.

If you want to use crosstool-NG as a backend to generate your toolchains for
your build-system, you have to set and export this environment variable (not
case sensitive, you can say `Y`):

    CT_IS_A_BACKEND=y
