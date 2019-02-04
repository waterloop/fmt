{fmt}
=====

[![image](https://travis-ci.org/fmtlib/fmt.png?branch=master)](https://travis-ci.org/fmtlib/fmt)

[![image](https://ci.appveyor.com/api/projects/status/ehjkiefde6gucy1v)](https://ci.appveyor.com/project/vitaut/fmt)

[![Join the chat at https://gitter.im/fmtlib/fmt](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/fmtlib/fmt)

**{fmt}** is an open-source formatting library for C++. It can be used
as a safe and fast alternative to (s)printf and IOStreams.

[Documentation](http://fmtlib.net/latest/)

Features
--------

-   Replacement-based [format API](http://fmtlib.net/dev/api.html) with
    positional arguments for localization.
-   [Format string syntax](http://fmtlib.net/dev/syntax.html) similar to
    the one of
    [str.format](https://docs.python.org/2/library/stdtypes.html#str.format)
    in Python.
-   Safe [printf
    implementation](http://fmtlib.net/latest/api.html#printf-formatting)
    including the POSIX extension for positional arguments.
-   Implementation of the ISO C++ standards proposal [P0645 Text
    Formatting](http://fmtlib.net/Text%20Formatting.html).
-   Support for user-defined types.
-   High speed: performance of the format API is close to that of
    glibc\'s [printf](http://en.cppreference.com/w/cpp/io/c/fprintf) and
    better than the performance of IOStreams. See [Speed
    tests](#speed-tests) and [Fast integer to string conversion in
    C++](http://zverovich.net/2013/09/07/integer-to-string-conversion-in-cplusplus.html).
-   Small code size both in terms of source code (the minimum
    configuration consists of just three header files, `core.h`,
    `format.h` and `format-inl.h`) and compiled code. See [Compile time
    and code bloat](#compile-time-and-code-bloat).
-   Reliability: the library has an extensive set of [unit
    tests](https://github.com/fmtlib/fmt/tree/master/test).
-   Safety: the library is fully type safe, errors in format strings can
    be reported at compile time, automatic memory management prevents
    buffer overflow errors.
-   Ease of use: small self-contained code base, no external
    dependencies, permissive BSD
    [license](https://github.com/fmtlib/fmt/blob/master/LICENSE.rst)
-   [Portability](http://fmtlib.net/latest/index.html#portability) with
    consistent output across platforms and support for older compilers.
-   Clean warning-free codebase even on high warning levels
    (`-Wall -Wextra -pedantic`).
-   Support for wide strings.
-   Optional header-only configuration enabled with the
    `FMT_HEADER_ONLY` macro.

See the [documentation](http://fmtlib.net/latest/) for more details.

Examples
--------

This prints `Hello, world!` to stdout:

``` {.sourceCode .c++}
fmt::print("Hello, {}!", "world");  // uses Python-like format string syntax
fmt::printf("Hello, %s!", "world"); // uses printf format string syntax
```

Arguments can be accessed by position and arguments\' indices can be
repeated:

``` {.sourceCode .c++}
std::string s = fmt::format("{0}{1}{0}", "abra", "cad");
// s == "abracadabra"
```

Format strings can be checked at compile time:

``` {.sourceCode .c++}
// test.cc
#define FMT_STRING_ALIAS 1
#include <fmt/format.h>
std::string s = format(fmt("{2}"), 42);
```

``` {.sourceCode .}
$ c++ -Iinclude -std=c++14 test.cc
...
test.cc:4:17: note: in instantiation of function template specialization 'fmt::v5::format<S, int>' requested here
std::string s = format(fmt("{2}"), 42);
                ^
include/fmt/core.h:778:19: note: non-constexpr function 'on_error' cannot be used in a constant expression
    ErrorHandler::on_error(message);
                  ^
include/fmt/format.h:2226:16: note: in call to '&checker.context_->on_error(&"argument index out of range"[0])'
      context_.on_error("argument index out of range");
               ^
```

{fmt} can be used as a safe portable replacement for `itoa`
([godbolt](https://godbolt.org/g/NXmpU4)):

``` {.sourceCode .c++}
fmt::memory_buffer buf;
format_to(buf, "{}", 42);    // replaces itoa(42, buffer, 10)
format_to(buf, "{:x}", 42);  // replaces itoa(42, buffer, 16)
// access the string using to_string(buf) or buf.data()
```

Formatting of user-defined types is supported via a simple [extension
API](http://fmtlib.net/latest/api.html#formatting-user-defined-types):

``` {.sourceCode .c++}
#include "fmt/format.h"

struct date {
  int year, month, day;
};

template <>
struct fmt::formatter<date> {
  template <typename ParseContext>
  constexpr auto parse(ParseContext &ctx) { return ctx.begin(); }

  template <typename FormatContext>
  auto format(const date &d, FormatContext &ctx) {
    return format_to(ctx.out(), "{}-{}-{}", d.year, d.month, d.day);
  }
};

std::string s = fmt::format("The date is {}", date{2012, 12, 9});
// s == "The date is 2012-12-9"
```

You can create your own functions similar to
[format](http://fmtlib.net/latest/api.html#format) and
[print](http://fmtlib.net/latest/api.html#print) which take arbitrary
arguments ([godbolt](https://godbolt.org/g/MHjHVf)):

``` {.sourceCode .c++}
// Prints formatted error message.
void vreport_error(const char *format, fmt::format_args args) {
  fmt::print("Error: ");
  fmt::vprint(format, args);
}
template <typename... Args>
void report_error(const char *format, const Args & ... args) {
  vreport_error(format, fmt::make_format_args(args...));
}

report_error("file not found: {}", path);
```

Note that `vreport_error` is not parameterized on argument types which
can improve compile times and reduce code size compared to fully
parameterized version.

Projects using this library
---------------------------

-   [0 A.D.](http://play0ad.com/): A free, open-source, cross-platform
    real-time strategy game
-   [AMPL/MP](https://github.com/ampl/mp): An open-source library for
    mathematical programming
-   [AvioBook](https://www.aviobook.aero/en): A comprehensive aircraft
    operations suite
-   [Celestia](https://celestia.space/): Real-time 3D visualization of
    space
-   [Ceph](https://ceph.com/): A scalable distributed storage system
-   [CUAUV](http://cuauv.org/): Cornell University\'s autonomous
    underwater vehicle
-   [HarpyWar/pvpgn](https://github.com/pvpgn/pvpgn-server): Player vs
    Player Gaming Network with tweaks
-   [KBEngine](http://kbengine.org/): An open-source MMOG server engine
-   [Keypirinha](http://keypirinha.com/): A semantic launcher for
    Windows
-   [Kodi](https://kodi.tv/) (formerly xbmc): Home theater software
-   [Lifeline](https://github.com/peter-clark/lifeline): A 2D game
-   [Drake](http://drake.mit.edu/): A planning, control, and analysis
    toolbox for nonlinear dynamical systems (MIT)
-   [Envoy](https://lyft.github.io/envoy/): C++ L7 proxy and
    communication bus (Lyft)
-   [FiveM](https://fivem.net/): a modification framework for GTA V
-   [MongoDB Smasher](https://github.com/duckie/mongo_smasher): A small
    tool to generate randomized datasets
-   [OpenSpace](http://openspaceproject.com/): An open-source
    astrovisualization framework
-   [PenUltima Online (POL)](http://www.polserver.com/): An MMO server,
    compatible with most Ultima Online clients
-   [quasardb](https://www.quasardb.net/): A distributed,
    high-performance, associative database
-   [readpe](https://bitbucket.org/sys_dev/readpe): Read Portable
    Executable
-   [redis-cerberus](https://github.com/HunanTV/redis-cerberus): A Redis
    cluster proxy
-   [rpclib](http://rpclib.net/): A modern C++ msgpack-RPC server and
    client library
-   [Saddy](https://github.com/mamontov-cpp/saddy-graphics-engine-2d):
    Small crossplatform 2D graphic engine
-   [Salesforce Analytics
    Cloud](http://www.salesforce.com/analytics-cloud/overview/):
    Business intelligence software
-   [Scylla](http://www.scylladb.com/): A Cassandra-compatible NoSQL
    data store that can handle 1 million transactions per second on a
    single server
-   [Seastar](http://www.seastar-project.org/): An advanced, open-source
    C++ framework for high-performance server applications on modern
    hardware
-   [spdlog](https://github.com/gabime/spdlog): Super fast C++ logging
    library
-   [Stellar](https://www.stellar.org/): Financial platform
-   [Touch Surgery](https://www.touchsurgery.com/): Surgery simulator
-   [TrinityCore](https://github.com/TrinityCore/TrinityCore):
    Open-source MMORPG framework

[More\...](https://github.com/search?q=cppformat&type=Code)

If you are aware of other projects using this library, please let me
know by [email](mailto:victor.zverovich@gmail.com) or by submitting an
[issue](https://github.com/fmtlib/fmt/issues).

Motivation
----------

So why yet another formatting library?

There are plenty of methods for doing this task, from standard ones like
the printf family of function and IOStreams to Boost Format library and
FastFormat. The reason for creating a new library is that every existing
solution that I found either had serious issues or didn\'t provide all
the features I needed.

### Printf

The good thing about printf is that it is pretty fast and readily
available being a part of the C standard library. The main drawback is
that it doesn\'t support user-defined types. Printf also has safety
issues although they are mostly solved with
[\_\_[attribute](http://fmtlib.net/latest/usage.html#building-the-library)
((format (printf,
\...))](http://gcc.gnu.org/onlinedocs/gcc/Function-Attributes.html) in
GCC. There is a POSIX extension that adds positional arguments required
for
[i18n](https://en.wikipedia.org/wiki/Internationalization_and_localization)
to printf but it is not a part of C99 and may not be available on some
platforms.

### IOStreams

The main issue with IOStreams is best illustrated with an example:

``` {.sourceCode .c++}
std::cout << std::setprecision(2) << std::fixed << 1.23456 << "\n";
```

which is a lot of typing compared to printf:

``` {.sourceCode .c++}
printf("%.2f\n", 1.23456);
```

Matthew Wilson, the author of FastFormat, referred to this situation
with IOStreams as \"chevron hell\". IOStreams doesn\'t support
positional arguments by design.

The good part is that IOStreams supports user-defined types and is safe
although error reporting is awkward.

### Boost Format library

This is a very powerful library which supports both printf-like format
strings and positional arguments. Its main drawback is performance.
According to various benchmarks it is much slower than other methods
considered here. Boost Format also has excessive build times and severe
code bloat issues (see [Benchmarks](#benchmarks)).

### FastFormat

This is an interesting library which is fast, safe and has positional
arguments. However it has significant limitations, citing its author:

> Three features that have no hope of being accommodated within the
> current design are:
>
> -   Leading zeros (or any other non-space padding)
> -   Octal/hexadecimal encoding
> -   Runtime width/alignment specification

It is also quite big and has a heavy dependency, STLSoft, which might be
too restrictive for using it in some projects.

### Loki SafeFormat

SafeFormat is a formatting library which uses printf-like format strings
and is type safe. It doesn\'t support user-defined types or positional
arguments. It makes unconventional use of `operator()` for passing
format arguments.

### Tinyformat

This library supports printf-like format strings and is very small and
fast. Unfortunately it doesn\'t support positional arguments and
wrapping it in C++98 is somewhat difficult. Also its performance and
code compactness are limited by IOStreams.

### Boost Spirit.Karma

This is not really a formatting library but I decided to include it here
for completeness. As IOStreams it suffers from the problem of mixing
verbatim text with arguments. The library is pretty fast, but slower on
integer formatting than `fmt::Writer` on Karma\'s own benchmark, see
[Fast integer to string conversion in
C++](http://zverovich.net/2013/09/07/integer-to-string-conversion-in-cplusplus.html).

Benchmarks
----------

### Speed tests

The following speed tests results were generated by building
`tinyformat_test.cpp` on Ubuntu GNU/Linux 14.04.1 with
`g++-4.8.2 -O3 -DSPEED_TEST -DHAVE_FORMAT`, and taking the best of three
runs. In the test, the format string `"%0.10f:%04d:%+g:%s:%p:%c:%%\n"`
or equivalent is filled 2000000 times with output sent to `/dev/null`;
for further details see the
[source](https://github.com/fmtlib/format-benchmark/blob/master/tinyformat_test.cpp).

+-------------------+---------------+-------------+
| Library           | Method        | Run Time, s |
+===================+===============+=============+
| libc              | printf        | > 1.35      |
+-------------------+---------------+-------------+
| libc++            | std::ostream  | > 3.42      |
+-------------------+---------------+-------------+
| fmt 534bff7       | fmt::print    | > 1.56      |
+-------------------+---------------+-------------+
| tinyformat 2.0.1  | tfm::printf   | > 3.73      |
+-------------------+---------------+-------------+
| Boost Format 1.54 | boost::format | > 8.44      |
+-------------------+---------------+-------------+
| Folly Format      | folly::format | > 2.54      |
+-------------------+---------------+-------------+

As you can see `boost::format` is much slower than the alternative
methods; this is confirmed by [other
tests](http://accu.org/index.php/journals/1539). Tinyformat is quite
good coming close to IOStreams. Unfortunately tinyformat cannot be
faster than the IOStreams because it uses them internally. Performance
of fmt is close to that of printf, being [faster than printf on integer
formatting](http://zverovich.net/2013/09/07/integer-to-string-conversion-in-cplusplus.html),
but slower on floating-point formatting which dominates this benchmark.

### Compile time and code bloat

The script
[bloat-test.py](https://github.com/fmtlib/format-benchmark/blob/master/bloat-test.py)
from [format-benchmark](https://github.com/fmtlib/format-benchmark)
tests compile time and code bloat for nontrivial projects. It generates
100 translation units and uses `printf()` or its alternative five times
in each to simulate a medium sized project. The resulting executable
size and compile time (Apple LLVM version 8.1.0 (clang-802.0.42), macOS
Sierra, best of three) is shown in the following tables.

**Optimized build (-O3)**

+---------------+-----------------+----------------------+--------------------+
| Method        | Compile Time, s | Executable size, KiB | Stripped size, KiB |
+===============+=================+======================+====================+
| printf        | > 2.6           | > 29                 | > 26               |
+---------------+-----------------+----------------------+--------------------+
| printf+string | > 16.4          | > 29                 | > 26               |
+---------------+-----------------+----------------------+--------------------+
| IOStreams     | > 31.1          | > 59                 | > 55               |
+---------------+-----------------+----------------------+--------------------+
| fmt           | > 19.0          | > 37                 | > 34               |
+---------------+-----------------+----------------------+--------------------+
| tinyformat    | > 44.0          | > 103                | > 97               |
+---------------+-----------------+----------------------+--------------------+
| Boost Format  | > 91.9          | > 226                | > 203              |
+---------------+-----------------+----------------------+--------------------+
| Folly Format  | > 115.7         | > 101                | > 88               |
+---------------+-----------------+----------------------+--------------------+

As you can see, fmt has 60% less overhead in terms of resulting binary
code size compared to IOStreams and comes pretty close to `printf`.
Boost Format and Folly Format have the largest overheads.

`printf+string` is the same as `printf` but with extra `<string>`
include to measure the overhead of the latter.

**Non-optimized build**

+---------------+-----------------+----------------------+--------------------+
| Method        | Compile Time, s | Executable size, KiB | Stripped size, KiB |
+===============+=================+======================+====================+
| printf        | > 2.2           | > 33                 | > 30               |
+---------------+-----------------+----------------------+--------------------+
| printf+string | > 16.0          | > 33                 | > 30               |
+---------------+-----------------+----------------------+--------------------+
| IOStreams     | > 28.3          | > 56                 | > 52               |
+---------------+-----------------+----------------------+--------------------+
| fmt           | > 18.2          | > 59                 | > 50               |
+---------------+-----------------+----------------------+--------------------+
| tinyformat    | > 32.6          | > 88                 | > 82               |
+---------------+-----------------+----------------------+--------------------+
| Boost Format  | > 54.1          | > 365                | > 303              |
+---------------+-----------------+----------------------+--------------------+
| Folly Format  | > 79.9          | > 445                | > 430              |
+---------------+-----------------+----------------------+--------------------+

`libc`, `lib(std)c++` and `libfmt` are all linked as shared libraries to
compare formatting function overhead only. Boost Format and tinyformat
are header-only libraries so they don\'t provide any linkage options.

### Running the tests

Please refer to Building the library\_\_ for the instructions on how to
build the library and run the unit tests.

Benchmarks reside in a separate repository,
[format-benchmarks](https://github.com/fmtlib/format-benchmark), so to
run the benchmarks you first need to clone this repository and generate
Makefiles with CMake:

    $ git clone --recursive https://github.com/fmtlib/format-benchmark.git
    $ cd format-benchmark
    $ cmake .

Then you can run the speed test:

    $ make speed-test

or the bloat test:

    $ make bloat-test

FAQ
---

Q: how can I capture formatting arguments and format them later?

A: use `std::tuple`:

``` {.sourceCode .c++}
template <typename... Args>
auto capture(const Args&... args) {
  return std::make_tuple(args...);
}

auto print_message = [](const auto&... args) {
  fmt::print(args...);
};

// Capture and store arguments:
auto args = capture("{} {}", 42, "foo");
// Do formatting:
std::apply(print_message, args);
```

License
-------

fmt is distributed under the BSD
[license](https://github.com/fmtlib/fmt/blob/master/LICENSE.rst).

The [Format String Syntax](http://fmtlib.net/latest/syntax.html) section
in the documentation is based on the one from Python [string module
documentation](https://docs.python.org/3/library/string.html#module-string)
adapted for the current library. For this reason the documentation is
distributed under the Python Software Foundation license available in
[doc/python-license.txt](https://raw.github.com/fmtlib/fmt/master/doc/python-license.txt).
It only applies if you distribute the documentation of fmt.

Acknowledgments
---------------

The fmt library is maintained by Victor Zverovich
([vitaut](https://github.com/vitaut)) and Jonathan Müller
([foonathan](https://github.com/foonathan)) with contributions from many
other people. See
[Contributors](https://github.com/fmtlib/fmt/graphs/contributors) and
[Releases](https://github.com/fmtlib/fmt/releases) for some of the
names. Let us know if your contribution is not listed or mentioned
incorrectly and we\'ll make it right.

The benchmark section of this readme file and the performance tests are
taken from the excellent
[tinyformat](https://github.com/c42f/tinyformat) library written by
Chris Foster. Boost Format library is acknowledged transitively since it
had some influence on tinyformat. Some ideas used in the implementation
are borrowed from [Loki](http://loki-lib.sourceforge.net/) SafeFormat
and [Diagnostic
API](http://clang.llvm.org/doxygen/classclang_1_1Diagnostic.html) in
[Clang](http://clang.llvm.org/). Format string syntax and the
documentation are based on Python\'s
[str.format](http://docs.python.org/2/library/stdtypes.html#str.format).
Thanks [Doug Turnbull](https://github.com/softwaredoug) for his valuable
comments and contribution to the design of the type-safe API and
[Gregory Czajkowski](https://github.com/gcflymoto) for implementing
binary formatting. Thanks [Ruslan Baratov](https://github.com/ruslo) for
comprehensive [comparison of integer formatting
algorithms](https://github.com/ruslo/int-dec-format-tests) and useful
comments regarding performance, [Boris
Kaul](https://github.com/localvoid) for [C++ counting digits
benchmark](https://github.com/localvoid/cxx-benchmark-count-digits).
Thanks to [CarterLi](https://github.com/CarterLi) for contributing
various improvements to the code.
