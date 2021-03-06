netperf-wrapper
===============

Python wrapper to run multiple simultaneous netperf/iperf/ping instances
and aggregate the results. The main documentation is in the man page,
`available as HTML here <https://tohojo.github.io/netperf-wrapper.1.html>`.

Tests are specified as config files (which are really Python), and
various parsers for tool output are supplied. At the moment, parsers for
netperf in -D mode, iperf in csv mode and ping/ping6 in -D mode are
supplied, as well as a generic parser for commands that just outputs a
single number.

Several commands can be run in parallel and, provided they output
timestamped values, (which netperf ping and iperf do, the latter with a
small patch, available in the misc/ directory), the test data points can
be aligned with each other in time, interpolating differences between
the actual measurement points. This makes it possible to graph (e.g.)
ping times before, during and after a link is loaded.

An alternative run mode is running several iterated tests (which each
output one data point, e.g. netperf tests not in -D mode), and
outputting the results of these several runs.

The aggregated data is saved in (gzipped) json format for later
processing and/or import into other tools. The json format is documented
below.

Apart from the json format, the data can be output as csv values, emacs
org mode tables or plots. Each test can specify several different plots,
including time-series plots of the values against each other, as well as
CDF plots of (e.g.) ping times.

Plotting requires a functional matplotlib installation (but everything
else can run without matplotlib), and can be output to the formats
supported by matplotlib by specifying the output filename with -o
output.{png,ps,pdf,svg}. If no output file is specified, the plot is
diplayed using matplotlib's interactive plot browser, which also allows
saving of the output (in .png format).

The basic invocation is ``./netperf-wrapper -H <host> <test_name>``.
Various options to control test parameters are available; try running
``./netperf-wrapper -h``. Tests can be displayed with
``./netperf-wrapper --list-tests`` and the available plots can be
displayed with ``./netperf-wrapper --list-plots <test_name>``.

Running tests and plotting/displaying the output is logically split up
in two separate processes, but can be combined into one. When a test is
run, its data output is always saved in a file called
``<test_name>-<date>.json.gz`` in the same directory as the output file
selected with -o (or the current directory if no output file is
selected). This file can be read back in with the -i switch, in which
case the test will not be run again, but the saved test data will be
used as input for plotting functions etc. If an output format is
selected while a test is run, the test data will be used directly for
this output, but will still be saved in the json file.

Installing
----------

Install the package system-wide by running
``sudo python2 setup.py install`` or
``sudo pip install netperf-wrapper`` for the latest released version.
Packages for Debian/Ubuntu and Arch Linux are available at OBS:
https://build.opensuse.org/project/repositories/home:tohojo:netperf-wrapper.

The json data format
--------------------

The aggregated test data is saved in a file called
``<test_name>-<date>.json.gz``. This file contains the data points
generated during the test, as well as some metadata. The top-level json
object has three keys in it: ``x_values``, ``results`` and ``metadata``.

``x_values`` is an array of the x values for the test data (typically
the time values for timeseries data).

``results`` is a json object containing the result data series. The keys
are the data series names; the value for each key is an array of y
values for that data series. The data array has the same length as the
``x_values`` array, but there may be missing data points (signified by
null values).

``metadata`` is an object containing various data points about the test
run. The metadata values are read in as configuration parameters when
the data set is loaded in for further processing. Not all tests use all
the parameters, but they are saved anyway.

Currently the metadata values are:

-  ``NAME``: The test name.
-  ``TITLE``: Any extra title specified by the -t parameter when the
   test was run.
-  ``HOSTS``: List of the server hostnames connected to during the test.
-  ``LOCAL_HOST``: The hostname of the machine that ran the test.
-  ``LENGTH``: Test length in seconds, as specified by the -l parameter.
-  ``TOTAL_LENGTH``: Actual data series length, after the test has added
   time to the LENGTH.
-  ``STEP_SIZE``: Time step size granularity.
-  ``TIME``: ISO timestamp of the time the test was initiated.
-  ``NOTE``: Arbitrary text as entered with the ``--note`` switch when
   the test was run.
