.. _configuration:

Configuration Specifications
============================
This page documents the various options and
parameters that can be set in the configuration
file.

MetSim Section
--------------

**Required Variables**

``time_step :: int``: The timestep to disaggregate in minutes.  If given as 1440
(number of minutes in a day), no disaggregation will occur. This value must
divide 1440 evenly.

``start :: str``: The time to start simulation given in the format
``yyyy/mm/dd``

``stop :: str``: The time to end simulation given in the format
``yyyy/mm/dd``.

``forcing :: path``: The path to the input forcing file(s).  See the section
on __forcing_vars__ for more details.

``domain :: path``: The path to the input domain file.  See the section on
__domain_vars__ for more details.

``state :: path``: The path to the input state file.

``out_dir :: path``: The location to write output to.  If this path doesn't
exist, it will be created.

``forcing_fmt :: str``: A string representing the type of input files specified in
the ``forcing`` entry.  Can be one of the following: ``ascii``, ``binary``,
``netcdf``, or ``data``.

**Optional Variables**

``output_prefix :: str``: The output file base name. Defaults to ``forcing``.

``out_precision :: str``: Precision to use when writing output.  Defaults to
``f8``.  Can be either ``f4`` or ``f8``.

``time_grouper :: str``: Whether to chunk up the timeseries into pieces for
processing. This option is useful to set for when you are limited on
memory.  Each chunk of output is written as ``{out_prefix}_{date_range}`` when
active. Any valid ``pandas.TimeGrouper`` string may be used (e.g. use '10AS'
for 10 year chunks).

``verbose :: bool``: Whether to print output to ``stdout``.  Should be set using
the ``-v`` flag for command line usage.  This can be set for scripting purposes,
if desired. Set to ``1`` to print output; defaults to ``0``.

``sw_prec_thresh :: float``: Minimum precipitation threshold to take into
account when simulating incoming shortwave radiation.  Defaults to ``0``.

``rain_scalar :: float``: Scale factor for calculation of cloudy sky
transmittance.  Defaults to ``0.75``, range should be between ``0`` and
``1``.

``utc_offset :: bool``: Whether to use UTC timecode offsets for shifting
timeseries. Without this option all times should be considered local to
the gridcell being processed. Large domain runs probably want to set this
option to ``True``.

``lw_cloud :: str``: Type of cloud correction to longwave radiation to apply.
Can be either ``DEFAULT`` or ``CLOUD_DEARDORFF``.  Defaults to
``CLOUD_DEARDORFF``.  Capitalization does not matter.

``lw_type :: str``: Type of longwave radiation parameterization to apply. Can be
one of the following: ``DEFAULT``, ``TVA``, ``ANDERSON``, ``BRUTSAERT``,
``SATTERLUND``, ``IDSO``, or ``PRATA``.  Defaults to ``PRATA``.  Capitalization
does not matter.

``tdew_tol :: float``: Convergence criteria for the iterative calculation of
dewpoint temperature in MtClim.  Defaults to ``1e-6``.

``tmax_daylength_fraction :: float`` : Weight for calculation of time of maximum
daily temperature.  Must be between ``0`` and ``1``.  Defaults to ``0.67``.

``tday_coef :: float``: Scale factor for calculation of daily mean temperature.
Defaults to ``0.45``, range should be between ``0`` and ``1``.

``lapse_rate :: float``: Used to calculate atmospheric pressure. Defaults to
``0.0065 K/m``.

``out_vars :: list`` : List of variables to write to output.  Should be a list
containing valid variables.  The list of valid variables is dependent on which
simulation method is used, as well as whether disaggregation is used. Defaults
to ``['temp', 'prec', 'shortwave', 'longwave', 'vapor_pressure', 'red_humid']``.

``prec_type :: str``: Type of precipitation disaggregation method to use. Can be
one of the following: ``uniform``, ``triangle``, or ``mix``. Defaults to
``uniform``.  Capitalization does not matter. Under ``uniform`` method,
precipitation is disaggregated by dividing uniformly over all sub-daily
timesteps. Under ``triangle`` the "triangle" method is employed whereby daily
precipitation is distributed assuming an isosceles triangle shape with peak and
width determined from two domain variables, ``t_pk`` and ``dur``.  Under
``mix``, the "uniform" method is used on days when ``t_min`` < 0 C, and
"triangle" is used on all other days; this hybrid method retains the improved
accuracy of "triangle" in terms of warm season runoff but avoids the biases
in snow accumulation that the "triangle" method sometimes yields due to fixed
event timing within the diurnal cycle of temperature. A domain file for the
CONUS+Mexico domain, containing the ``dur`` and ``t_pk`` parameters is
available at: `<https://zenodo.org/record/1402223#.XEI-mM2IZPY>`.  For more
information about the "triangle" method see :doc:`PtriangleMethod.pdf`.

For more information about input and output variables see the :ref:`data` page.

chunks section
--------------
The ``chunks`` section describes how parallel computation should be grouped
in space. For example, to parallelize over 10 by 10 chunks of latitude and
longitude (with netcdf dimensions named ``lat`` and ``lon``, respectively) you would use:

.. code-block:: ini
    [chunks]
    lat = 10
    lon = 10

Alternatively, for an HRU based run chunked into 50 element jobs you would use:

.. code-block:: ini
    [chunks]
    hru = 50

As a general rule of thumb, try to evenly chunk the domain in such a way that
the number of jobs to run is some multiple of the number of processors you wish
to run on.

forcing_vars and state_vars section
---------------
The ``forcing_vars`` and ``state_vars`` sections are where you can specify which variables are in your
input data, and the corresponding symbols which MetSim will recognize. The
format of this section depends on the value given in the ``in_fmt`` entry in
the ``MetSim`` section of the configuration file.  See below for conventions for
each input type.


netcdf and data
```````````````
The ``in_vars`` section for NetCDF and xarray input acts as a mapping between the variable
names in the input dataset to the variable names expected by MetSim.  The format
is given as ``metsim_varname = netcdf_varname``.  The minimum required variables
given have ``metsim_varname``\s corresponding to ``t_min``, ``t_max``, and
``prec``; these variable names correspond to minimum daily temperature (Celcius),
maximum daily temperature (Celcius), and precipitation (mm/day).

ascii
`````
The ``in_vars`` section for ASCII input acts similarly to the NetCDF input
format, except for one key point.  Variables should be given as a tautology: the
format is given as ``metsim_varname = metsim_varname``.  The order that the
variables are given corresponds to the column numbers that they appear in the
input files.  The minimum required variables are ``t_min``, ``t_max``, and
``prec``; these variable names correspond to minimum daily temperature (Celcius),
maximum daily temperature (Celcius), and precipitation (mm/day).

binary
``````
This section has an input style for binary files that mimics the VIC version 4
input style.  Each line is specified as ``varname = scale cdatatype``, where
``varname`` is the name that MetSim should use for the column, ``scale`` is a
floating point scaling factor that should be applied after conversion from
binary to floating point; the conversion applied by the ``scale`` is applied
after the value in the input is converted from binary to the ``cdatatype``
specified for each variable.  Valid ``cdatatype``\s are ``signed`` and
``unsigned``.  ``signed`` values are interpreted as values which can be positive
or negative, whereas ``unsigned`` values are interpreted as values that can only
be greater than or equal to zero.

domain_vars section
-------------------
The ``domain_vars`` section is where information about the domain file is given.
Since the domain file is given as a NetCDF file this section has a similar
format to that of the NetCDF input file format described above.  That is,
entries should be of the form ``metsim_varname = netcdfvarname``. The minimum
required variables have ``metsim_varname``\s corresponding to ``lat``, ``lon``,
``mask``, and ``elev``; these variable names correspond to latitude, longitude,
a mask of valid cells in the domain, and the elevation given in meters. If
``prec_type`` = ``triangle`` or ``mix``, two additonal variables are required
including ``dur`` and ``t_pk`` for disaggregating daily precipitation according
to the "triangle" method.
