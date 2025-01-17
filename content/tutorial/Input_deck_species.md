+++
title = ""
draft = false  # Is this a draft? true/false
toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.

# Add menu entry to sidebar.
linktitle = "Input deck species"
[menu.tutorial]
  parent = "Content"
  weight = 410
+++

This block contains information about the species of particles which are
used in the code. Also details of how these are initialised. See [EPOCH
input deck][Input_deck] for more information on the
input deck.

# Basics

The next section of the input deck describes the particle species used
in the code. An example species block for any EPOCH code is given below.

```perl
begin:species
   name = Electron
   charge = -1.0
   mass = 1.0
   frac = 0.5
   # npart = 2000 * 100
   number_density = 1.e4
   temp = 1e6

   temp_x = 0.0
   temp_y = temp_x(Electron)
   number_density_min = 0.1 * den_max
   number_density = if(abs(x) lt thick, den_max, 0.0)
   number_density = if((x gt -thick) and (abs(y) gt 2e-6), \
                        0.0, number_density(Carbon))
end:species

begin:species
   name = Carbon
   charge = 4.0
   mass = 1836.0*12
   frac = 0.5

   number_density = 0.25*number_density(Electron)
   temp_x = temp_x(Electron)
   temp_y = temp_x(Electron)

   dumpmask = full
end:species
```

Each species block accepts the following parameters:\
- `name` - This specifies the name of the particle species
defined in the current block. This name can include any alphanumeric
characters in the basic ASCII set. The name is used to identify the
species in any consequent input block and is also used for labelling
species data in any output dumps. It is a mandatory parameter.\
**`NOTE: IT IS IMPOSSIBLE TO SET TWO SPECIES WITH THE SAME
NAME!`\
**

-   `charge` - This sets the charge of the species in
    multiples of the electron charge. Negative numbers are used for
    negatively charged particles. This is a mandatory parameter.\
-   `mass` - This sets the mass of the species in multiples
    of the electron mass. Cannot be negative. This is a mandatory
    parameter.\
-   `npart` - This specifies the number of pseudoparticles
    which should be loaded into the simulation domain for this species
    block. Using this parameter is the most convenient way of loading
    particles for simulations which contain multiple species with
    different number densities. If *npart* is specified in a species
    block then any value given for *npart* in the
    [*control*][Input_deck_control] block is ignored.
    *npart* should not be specified at the same time as *frac* within a
    *species* block.\
-   `frac` - This specifies what fraction of *npart* (the
    global number of particles specified in the control block) should be
    assigned to the species.\

**`NOTE: frac should not be specified at the same time as npart for
a given species.`\
**

-   `npart_per_cell` - Integer parameter which specifies
    the number of particles per cell to use for the initial particle
    loading. At a later stage this may be extended to allow
    "npart_per_cell" to be a spatially varying function.

If per-species weighting is used then the value of "npart_per_cell"
will be the average number of particles per cell. If "npart" or "frac"
have also been specified for a species, then they will be ignored.

To avoid confusion, there is no globally used "npart_per_species". If
you want to have a single value to change in the input deck then this
can be achieved using a
[*constant*][Input_deck_constant] block.\
- `dumpmask` - Determines which output dumps will include
this particle species. The dumpmask has the same semantics as those used
by variables in the [*output*][Input_deck_output_block]
block. The actual dumpmask from the output block is applied first and
then this one is applied afterwards. For example, if the species block
contains "dumpmask = full" and the output block contains "vx = always"
then the particle velocity will be only be dumped at full dumps for this
particle species. The default dumpmask is "always".\
- `dump` - This logical flag is provided for backwards
compatibility. If set to "F" it has the same meaning as "dumpmask =
never". If set to "T" it has the same meaning as "dumpmask = always".\
- `zero_current` - Logical flag switching the particle
species into zero-current particles. Zero-current particles are enabled
if the if the "NO_TRACER_PARTICLES" precompiler option has not been
used and the "zero_current" flag is set to true for a given species.
When set, the species will move correctly for its charge and mass, but
contribute no current. This means that these particles are passive
elements in the simulation. In all other respects they are designed to
behave identically to ordinary particles, so they do take part in
collisions by default. This can be prevented using the [collision
matrices][Input_deck_collisions].
<span style="color: red; font-weight: bold;">WARNING:</span> Since the
particles effectively have zero weight in terms of their numerical
heating properties, they do not always behave in the same way that an
ordinary particle with weight would behave and this can sometimes lead
to unexpected behaviour. If the purpose is merely to track a subset of a
particle species to use as output then a better mechanism to use is
"persistent subsets" (see [here][Input_deck_subset]).
"tracer" is currently accepted as an alias but this will be removed in
version 5.0. "zero_current = F" is the default value.\
- `identify` - Used to identify the type of particle.
Currently this is used primarily by the QED routines. See
[here][Input_deck_qed] for details.\
- `immobile` - Logical flag. If this parameter is set to "T"
then the species will be ignored during the particle push. The default
value is "F".\
The species blocks are also used for specifying initial conditions for
the particle species. The initial conditions in EPOCH can be specified
in various ways, but the easiest way is to specify the initial
conditions in the input deck file. This allows any initial condition
which can be specified everywhere in space by a number density and a
drifting Maxwellian distribution function. These are built up using the
normal maths expressions, by setting the density and temperature for
each species which is then used by the autoloader to actually position
the particles.

The elements of the species block used for setting initial conditions
are:\
- `number_density` - Particle number density in $m^{-3}$. As
soon as a number_density= line has been read, the values are calculated
for the whole domain and are available for reuse on the right hand side
of an expression. This is seen in the above example in the first two
lines for the Electron species, where the number density is first set
and then corrected. If you wish to specify the number density in parts
per cubic metre then you can divide by the "cc" constant (see
[here][maths_parser__constants]). This parameter is
mandatory. "density" is accepted as an alias.\
- `number_density_min` - Minimum particle number density in
$m^{-3}$. When the number density in a cell falls below
number_density_min the autoloader does not load any pseudoparticles
into that cell to minimise the number of low weight, unimportant
particles. If set to 0 then all cells are loaded with particles. This is
the default. "density_min" is accepted as an alias.\
- `number_density_max` - Maximum particle number density in
$m^{-3}$. When the number density in a cell rises above
number_density_max the autoloader clips the number_density to
number_density_max allowing easy implementation of exponential rises
to plateaus. If it is a negative value then no clipping is performed.
This is the default. "density_max" is accepted as an alias.\
- `mass_density` - Particle mass density in $kg\,m^{-3}$.
The same as "number_density" but multiplied by the particle mass. If
you wish to use units of $g\,cm^{-3}$ then append the appropriate
multiplication factor. For example: "`mass_density = 2 * 1e3 / cc`".\
- `temp_{x,y,z}` - The temperature in each direction for a
thermal distribution in Kelvin.\
- `temp` - Sets an isotropic temperature distribution in
Kelvin. If both temp and a specific temp_x, temp_y, temp_z parameter
is specified then the last to appear in the deck has precedence. If
neither are given then the species will have a default temperature of
zero Kelvin.\
- `temp_{x,y,z}_ev, temp_ev` - These are the same as the
temperature parameters described above except the units are given in
electronvolts rather than Kelvin, i.e. using 1ev = 11604.5K .\
- `drift_{x,y,z}` - Specifies a momentum space offset in
$kg\ ms^{-1}$ to the distribution function for this species. By default,
the drift is zero.\
- `offset` - File offset. See below for details.\

# Loading data from a file {#loading_data_from_a_file}

It is also possible to set initial conditions for a particle species
using an external file. Instead of specifying the initial conditions
mathematically in the input deck, you specify in quotation marks the
filename of a simple binary file containing the information required.
For more information on what is meant by a "simple binary file", see
[here][Binary_files].

```perl
begin:species
   name = Electron
   number_density = 'Data/ic.dat'
   offset = 80000
   temp_x = 'Data/ic.dat'
end:species
```

The sizes of the variables to be filled do not need to be provided: the
code will continue reading until the given variable is filled. Note that
ghost or guard cells should not be included in the file as they cannot
be set this way.

An additional element is also introduced, the offset element. This is
the offset in bytes from the start of the file to where the data should
be read from. As a given line in the block executes, the file is opened,
the file handle is moved to the point specified by the offset parameter,
the data is read and the file is then closed. Therefore, unless the
offset value is changed between data reading lines the same data will be
read into all the variables. The data is read in as soon as a line is
executed, and so it is perfectly possible to load data from a file and
then modify the data using a mathematical expression.\
The example block above is for 10,000 values at double precision, i.e.
8-bytes each. The density data is the first 80,000 bytes of "ic.dat".
Bytes 80,000 to 160,000 are the temp_x data.

The file should be a simple binary file consisting of floating point
numbers of the same precision as **_num** in the core EPOCH code. For
multidimensional arrays, the data is assumed to be written according to
FORTRAN array ordering rules (i.e. column-major order).\
**`NOTE: The files that are expected by this block are SIMPLE
BINARY files, NOT FORTRAN unformatted files. It is possible to read
FORTRAN unformatted files using the offset element, but care must be
taken!`**

# Delta-f parameters {#delta_f_parameters}

The following entries are used for configuring the [Delta-f
method][Using_delta_f]\
\*number_density_back

-   drift_{x,y,z}_back
-   temp_{x,y,z}_back
-   temp_{x,y,z}_back_ev
-   temp_back
-   temp_back_ev

These all have the same meanings as the parameters listed above that
don't include the "_back" text, except that they specify the values to
use for the background distribution function.\

# Particle migration between species {#particle_migration_between_species}

It is sometimes useful to separate particle species into separate energy
bands and to migrate particles between species when they become more or
less energetic. A method to achieve this functionality has been
implemented. It is specified using two parameters to the "control"
block:\
- `use_migration` - Logical flag which determines whether or
not to use particle migration. The default is "F".\
- `migration_interval` - The number of timesteps between
each migration event. The default is 1 (migrate at every timestep).\
The following parameters are added to the "species" block:\
- `migrate` - Logical flag which determines whether or not to
consider this species for migration. The default is "F".\
- `promote_to` - The name of the species to promote
particles to.\
- `demote_to` - The name of the species to demote particles
to.\
- `promote_multiplier` - The particle is promoted when its
energy is greater than "promote_multiplier" times the local average.
The default value is 1.\
- `demote_multiplier` - The particle is demoted when its
energy is less than "demote_multiplier" times the local average. The
default value is 1.\
- `promote_number_density` - The particle is only
considered for promotion when the local number density is less than
"promote_number_density". The default value is the largest floating
point number.\
- `demote_number_density` - The particle is only considered
for demotion when the local number density is greater than
"demote_number_density". The default value is 0.\

# Ionisation

EPOCH now includes field ionisation which can be activated by defining
ionisation energies and an electron for the ionising species. This is
done via the species block using the following parameters:\
- `ionisation_energies` - This is an array of ionisation
energies (in Joules) starting from the outermost shell. It expects to be
given all energies down to the fully ionised ion; if the user wishes to
exclude some inner shell ionisation for some reason they need to give
this a very large number. Note that the ionisation model assumes that
the outermost electron ionises first always, and that the orbitals are
filled assuming ground state. When this parameter is specified it turns
on ionisation modelling. If you wish to specify the values in
Electron-Volts, add the "ev" [multiplication
factor][maths_parser__constants].\
- `ionisation_electron_species` - Name of the electron
species. This can be specified as an array in the event that the user
wishes some levels to have a different electron species which can be
handy for monitoring ionisation at specific levels. "electron" and
"electron_species" are accepted as synonyms. Either one species for
*all* levels, or one species for *each* species should be specified.\
For example, ionising carbon species might appear in the input deck as:

```perl
begin:species
   charge = 0.0
   mass = 1837.2
   name = carbon
   ionisation_energies = (11.26*ev,24.38*ev,47.89*ev,64.49*ev,392.1*ev,490.0*ev)
   ionisation_electron_species = \
       (electron,electron,electron,fourth,electron,electron)
   number_density= den_gas
end:species

begin:species
   charge = -1.0
   mass = 1.0
   name = electron
   number_density = 0.0
end:species

begin:species
   charge = -1.0
   mass = 1.0
   name = fourth
   number_density = 0.0
end:species
```

Ionised states are created automatically and are named according to the
ionising species name with a number appended. For example

```perl
begin:species
   name = Helium
   ionisation_energies = (24.6*ev,54.4*ev)
   dump = F
end:species
```

With this species block, the species named "Helium1" and "Helium2" are
automatically created. These species will also inherit the "dump"
parameter from their parent species, so in this example they will both
have "dump = F" set. This behaviour can be overridden by explicitly
adding a species block of the same name with a differing dumpmask. eg.

```perl
begin:species
   name = Helium1
   dump = T
end:species
```

Field ionisation consists of three distinct regimes; multiphoton in
which ionisation is best described as absorption of multiple photons,
tunnelling in which deformation of the atomic Coulomb potential is the
dominant factor, and barrier suppression ionisation in which the
electric field is strong enough for an electron to escape classically.
It is possible to turn off multiphoton or barrier suppression ionisation
through the input deck using the following control block parameters:\
- `use_multiphoton` - Logical flag which turns on modelling
ionisation by multiple photon absorption. This should be set to "F" if
there is no laser attached to a boundary as it relies on laser
frequency. The default is "T".\
- `use_bsi` - Logical flag which turns on barrier
suppression ionisation correction to the tunnelling ionisation model for
high intensity lasers. The default is "T".\

# Species Boundary Conditions {#species_boundary_conditions}

-   `bc_x_min` - Boundary condition to be applied to this
    species only on the lower x boundary. Can be any normal boundary
    condition apart from periodic. If not specified then the global
    boundary condition is applied.
-   `bc_x_max` - Boundary condition to be applied to this
    species only on the upper x boundary. Can be any normal boundary
    condition apart from periodic. If not specified then the global
    boundary condition is applied.
-   `bc_y_min` - Boundary condition to be applied to this
    species only on the lower y boundary. Can be any normal boundary
    condition apart from periodic. If not specified then the global
    boundary condition is applied.
-   `bc_y_max` - Boundary condition to be applied to this
    species only on the upper y boundary. Can be any normal boundary
    condition apart from periodic. If not specified then the global
    boundary condition is applied.
-   `bc_z_min` - Boundary condition to be applied to this
    species only on the lower z boundary. Can be any normal boundary
    condition apart from periodic. If not specified then the global
    boundary condition is applied.
-   `bc_z_max` - Boundary condition to be applied to this
    species only on the upper z boundary. Can be any normal boundary
    condition apart from periodic. If not specified then the global
    boundary condition is applied.
-   `meet_injectors` - Logical flag determining whether the
    background plasma should be extended to meet the point where
    particle injectors operate from. This means that plasma is loaded
    one particle shape function length outside the boundary. This means
    that it is possible to use an injector to "continue" an existing
    drifting plasma. NOT COMPATIBLE WITH PERIODIC BOUNDARY CONDITIONS!

# Maxwell Juttner distributions {#maxwell_juttner_distributions}

As of version 4.15, EPOCH allows the user to request a Maxwell-Jüttner
distribution rather than a Maxwellian distribution when sampling the
particle momentum for a species.

This feature does not at present work with the delta_f loader and is
not available for particle injectors. It does work correctly with the
moving window.

-   `use_maxwell_juttner` - Logical flag determining
    whether to sample from the Maxwell-Jüttner distribution when loading
    the particle species. If "T" then Maxwell-Jüttner is used and if
    "F" Maxwellian is used. The default value is "F".

<!-- -->

-   `fractional_tail_cutoff - The sampling is carried out using a
    rejection method with an arbitrary cut-off. This parameter takes a
    floating-point argument which specifies the fraction of maximum
    value at which the sampling should be cut off. Smaller values lead
    to distortion nearer the peak of the distribution but are faster to
    sample. Larger values lead to a better approximation of the
    distribution function but are slower to sample. The default value is
    0.0001.

If drifts are specified with the Maxwell-Jüttner distribution then the
distribution is calculated in the rest frame and then Lorentz
transformed to the specified drifting frame.

# Arbitrary Distribution functions {#arbitrary_distribution_functions}

As of version 4.15, EPOCH also allows the user to request an arbitrary
non-Maxwellian distribution function to use when sampling the particle
momentum for a species. If combined with a specified drift then the
distribution function is calculated first and the drift is applied to
the resulting particles by Lorentz transform.

This feature does not at present work with the delta_f loader and is
not available for particle injectors. It does work correctly with the
moving window.

-   `dist_fn` - Specifies the functional form of the
    distribution function, normalised to have a maximum value of 1. The
    variables "px", "py" and "pz" should be used to parameterise
    the x, y and z components of momentum. This may freely vary in space
    but temporal variation will be ignored since this is only evaluated
    at the start of the simulation.

<!-- -->

-   `dist_fn_p{x,y,z}_range` - Comma separated pair of
    numbers to specify the range of momentum for p_{x,y,z} in SI units.
    Should be of the form "<lower_range>, <upper_range>"

If a range for a momentum direction is not specified then that momentum
is assumed to be zero. It is up to the user to ensure that the range is
large enough to correctly capture their desired distribution function.
Sampling is by a simple rejection sampling and may be much slower than
the existing Maxwellian sampler. EPOCH will print a warning if a large
number of samples are needed to complete the sampling. If this occurs
then you might need to reduce the range of momentum over which sampling
is considered.

If the "dist_fn" key is supplied then any supplied temperature keys
are ignored. An example of setting up a truncated power law distribution
in px would be

```perl
begin:constant
  dens = 10
  v0 = 0.05 * c
  vmax = 0.5 * c
  p0 = v0 * me * (1.0 + 4.0 * x/x_max)
  pmax = vmax * me
  alpha = -2.0
end:constant

begin:species
  name = Electron_pl
  charge = -1
  mass = 1.0
  frac = 0.5
  number_density = dens
  #Truncated power law distribution in px
  dist_fn = exp(-p0/px) * (px/p0)^(alpha)
  dist_fn_px_range = (0, pmax)
end:species
```

# Next section {#next_section}

[The laser block][Input_deck_laser]


<!-- ########################  Cross references  ######################## -->


[Acknowledging_EPOCH]: /tutorial/acknowledging_epoch
[Basic_examples]: /tutorial/basic_examples
[Basic_examples__focussing_a_gaussian_beam]: /tutorial/basic_examples/#focussing_a_gaussian_beam
[Binary_files]: /tutorial/binary_files
[Calculable_particle_properties]: /tutorial/calculable_particle_properties
[Compiler_Flags]: /tutorial/compiler_flags
[Compiling]: /tutorial/compiling
[FAQ]: /tutorial/faq
[FAQ__how_do_i_obtain_the_code]: /tutorial/faq/#how_do_i_obtain_the_code
[Input_deck]: /tutorial/input_deck
[Input_deck_adf]: /tutorial/input_deck_adf
[Input_deck_boundaries]: /tutorial/input_deck_boundaries
[Input_deck_boundaries__cpml_boundary_conditions]: /tutorial/input_deck_boundaries/#cpml_boundary_conditions
[Input_deck_boundaries__thermal_boundary_conditions]: /tutorial/input_deck_boundaries/#thermal_boundary_conditions
[Input_deck_collisions]: /tutorial/input_deck_collisions
[Input_deck_constant]: /tutorial/input_deck_constant
[Input_deck_control]: /tutorial/input_deck_control
[Input_deck_control__basics]: /tutorial/input_deck_control/#basics
[Input_deck_control__maxwell_solvers]: /tutorial/input_deck_control/#maxwell_solvers
[Input_deck_control__requesting_output_dumps_at_run_time]: /tutorial/input_deck_control/#requesting_output_dumps_at_run_time
[Input_deck_control__stencil_block]: /tutorial/input_deck_control/#stencil_block
[Input_deck_control__strided_current_filtering]: /tutorial/input_deck_control/#strided_current_filtering
[Input_deck_dist_fn]: /tutorial/input_deck_dist_fn
[Input_deck_fields]: /tutorial/input_deck_fields
[Input_deck_injector]: /tutorial/input_deck_injector
[Input_deck_injector__keys]: /tutorial/input_deck_injector/#keys
[Input_deck_laser]: /tutorial/input_deck_laser
[Input_deck_operator]: /tutorial/input_deck_operator
[Input_deck_output__directives]: /tutorial/input_deck_output/#directives
[Input_deck_output_block]: /tutorial/input_deck_output_block
[Input_deck_output_block__derived_variables]: /tutorial/input_deck_output_block/#derived_variables
[Input_deck_output_block__directives]: /tutorial/input_deck_output_block/#directives
[Input_deck_output_block__dumpmask]: /tutorial/input_deck_output_block/#dumpmask
[Input_deck_output_block__multiple_output_blocks]: /tutorial/input_deck_output_block/#multiple_output_blocks
[Input_deck_output_block__particle_variables]: /tutorial/input_deck_output_block/#particle_variables
[Input_deck_output_block__single-precision_output]: /tutorial/input_deck_output_block/#single-precision_output
[Input_deck_output_global]: /tutorial/input_deck_output_global
[Input_deck_particle_file]: /tutorial/input_deck_particle_file
[Input_deck_probe]: /tutorial/input_deck_probe
[Input_deck_qed]: /tutorial/input_deck_qed
[Input_deck_species]: /tutorial/input_deck_species
[Input_deck_species__arbitrary_distribution_functions]: /tutorial/input_deck_species/#arbitrary_distribution_functions
[Input_deck_species__ionisation]: /tutorial/input_deck_species/#ionisation
[Input_deck_species__maxwell_juttner_distributions]: /tutorial/input_deck_species/#maxwell_juttner_distributions
[Input_deck_species__particle_migration_between_species]: /tutorial/input_deck_species/#particle_migration_between_species
[Input_deck_species__species_boundary_conditions]: /tutorial/input_deck_species/#species_boundary_conditions
[Input_deck_subset]: /tutorial/input_deck_subset
[Input_deck_window]: /tutorial/input_deck_window
[Landing]: /tutorial/landing
[Landing_Page]: /tutorial/landing_page
[Libraries]: /tutorial/libraries
[Links]: /tutorial/links
[Maths_parser__functions]: /tutorial/maths_parser/#functions
[Non-thermal_initial_conditions]: /tutorial/non-thermal_initial_conditions
[Previous_versions]: /tutorial/previous_versions
[Python]: /tutorial/python
[Running]: /tutorial/running
[SDF_Landing_Page]: /tutorial/sdf_landing_page
[Structure]: /tutorial/structure
[Using_EPOCH_in_practice]: /tutorial/using_epoch_in_practice
[Using_EPOCH_in_practice__manually_overriding_particle_parameters_set_by_the_autoloader]: /tutorial/using_epoch_in_practice/#manually_overriding_particle_parameters_set_by_the_autoloader
[Using_EPOCH_in_practice__parameterising_input_decks]: /tutorial/using_epoch_in_practice/#parameterising_input_decks
[Using_delta_f]: /tutorial/using_delta_f
[Visualising_SDF_files_with_IDL_or_GDL]: /tutorial/visualising_sdf_files_with_idl_or_gdl
[Visualising_SDF_files_with_LLNL_VisIt]: /tutorial/visualising_sdf_files_with_llnl_visit
[Workshop_examples]: /tutorial/workshop_examples
[Workshop_examples__a_2d_laser]: /tutorial/workshop_examples/#a_2d_laser
[Workshop_examples__a_basic_em-field_simulation]: /tutorial/workshop_examples/#a_basic_em-field_simulation
[Workshop_examples__getting_the_example_decks_for_this_workshop]: /tutorial/workshop_examples/#getting_the_example_decks_for_this_workshop
[Workshop_examples__specifying_particle_species]: /tutorial/workshop_examples/#specifying_particle_species
[Workshop_examples_continued]: /tutorial/workshop_examples_continued
