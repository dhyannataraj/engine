Building and Installation
=========================

How to Build
------------

To build and install OpenSSL GOST Engine, you will need

* OpenSSL 1.1.*
* an ANSI C compiler
* CMake (3.0 or newer)

Here is a quick build guide:

    $ mkdir build
    $ cd build
    $ cmake -DCMAKE_BUILD_TYPE=Release ..
    $ cmake --build . --config Release

Instead of `Release` you can use `Debug`, `RelWithDebInfo` or `MinSizeRel` configuration.
See [cmake docs](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html) for details.
You will find built binaries in `../bin` directory.

If you want to build against a specific OpenSSL instance (you will need it
if you have more than one OpenSSL instance for example), you can use
the `cmake` variable `OPENSSL_ROOT_DIR` to specify path of the desirable
OpenSSL instance:

    $ cmake -DOPENSSL_ROOT_DIR=/PATH/TO/OPENSSL/ ..

If you use Visual Studio, you can also set `CMAKE_INSTALL_PREFIX` variable
to set install path, like this:

    > cmake -G "Visual Studio 15 Win64" -DCMAKE_PREFIX_PATH=c:\OpenSSL\vc-win64a\ -DCMAKE_INSTALL_PREFIX=c:\OpenSSL\vc-win64a\ ..

Also instead of `cmake --build` tool you can just open `gost-engine.sln`
in Visual Studio, select configuration and call `Build Solution` manually.

Instructions how to build OpenSSL 1.1.0 with Microsoft Visual Studio
you can find [there](https://gist.github.com/terrillmoore/995421ea6171a9aa50552f6aa4be0998).

How to Install
--------------

To install GOST Engine you can call:

    # cmake --build . --target install --config Release

or old plain and Unix only:

    # make install

The engine library `gost.so` should be installed into OpenSSL engine directory.

To ensure that it is installed propery call:

    $ openssl version -e
    ENGINESDIR: "/usr/lib/i386-linux-gnu/engines-1.1"

Then check that `gost.so` there

    # ls /usr/lib/i386-linux-gnu/engines-1.1

Finally, to start using GOST Engine through OpenSSL, you should edit
`openssl.cnf` configuration file as specified below.


How to Configure
----------------

Short configre example with some comments can be found in `example.conf` in this
distribution. More etended howto can be found here.

First, add the following line at the first line of `/etc/ssl/openssl.cnf`:

    openssl_conf = global_defaults

This will define the name of section were global default are defined. This
definition shoud be made in global section (i.e before first section header
(see config(5) ), so first line will do well.

Then add following two section at the end on OpenSSL config file:

    [global_defaults]
    engines = engines_section

    [engines_section]
    engine1 = gost_section

`engines = engine_section` in `global_defaults` section will define the name of
engine section, where you can list all engines that sould be loaded.

`engine1 = gost_section` in `engines_section` will tell OpenSSL to load one engine,
using options from `gost_section` section.


Then you shoud add `gost_section` as follows:

    [gost_section]
    engine_id = gost
    # dynamic_path = /usr/lib/ssl/engines/libgost.so
    default_algorithms = ALL
    CRYPT_PARAMS = id-tc26-gost-28147-param-Z
    # PK_PARAMS = LEGACY_PK_WRAP


Here `engine_id` specifies name of engine (should be `gost` for this engine).

Normally, engine library binary file will be installed into `ENGINESDIR`
(Run `openssl version -e` to find out where it is), and OpenSSL will be
able to find it. But you can also make OpenSSL to load this library from
alternative location, using `dynamic_path` option.

`default_algorithms` parameter allows to specify what classes of
crypthogrphic algorithms provided by this backend, shdould be used. Almost
always you will need all of them, so use `ALL` option here. Please notice,
`ALL` is not default value for this option, you should specify it in explicit
way.

If you are curious about other possible values for `default_algorithms`, you should
look for `ENGINE_METHOD_*` in `include/openssl/engine.h` and for `int_def_cb()` in
`crypto/engine/eng_fat.c` in OpenSSL source code. This part of OpenSSL is not well
documented, and you can get more info mostly from the code.



The `CRYPT_PARAMS` parameter is engine-specific. It allows the user to choose
between different parameter sets of symmetric cipher algorithm. [RFC 4357][1]
specifies several parameters for the GOST 28147-89 algorithm, but OpenSSL
doesn't provide user interface to choose one when encrypting. So use engine
configuration parameter instead.

Value of this parameter can be either short name, defined in OpenSSL
`obj_dat.h` header file or numeric representation of OID, defined in
[RFC 4357][1].


    PK_PARAMS = LEGACY_PK_WRAP

BouncyCastle cryptoprovider has some problems with private key parsing from
PrivateKeyInfo, so if you want to use old private key representation format,
which supported by BC, you must add:



[1]:https://tools.ietf.org/html/rfc4357 "RFC 4357"
