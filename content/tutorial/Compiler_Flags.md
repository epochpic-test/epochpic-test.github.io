+++
title = ""
draft = false  # Is this a draft? true/false
toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.

# Add menu entry to sidebar.
linktitle = "Compiler Flags"
[menu.tutorial]
  parent = "Content"
  weight = 140
+++

As already stated, some features of the code are controlled by compiler
preprocessor directives. The flags for these preprocessor directives are
specified in "Makefile" and are placed on lines which look like the
following:

    DEFINES += $(D)PER_SPECIES_WEIGHT

On most machines `$(D)` just means `-D` but the variable is required to
accommodate more exotic setups.

Most of the flags provided in the "Makefile" are commented out by
prepending them with a "`#`" symbol (the "make" system's comment
character). To turn on the effect controlled by a given preprocessor
directive, just uncomment the appropriate "DEFINES" line by deleting
this "`#`" symbol. The options currently controlled by the preprocessor
are:\

-   PER_SPECIES_WEIGHT - By default, each pseudoparticle in the code
    can represent a different number of real particles. Memory can be
    saved by disabling this feature and have all of the pseudoparticles
    in a species use the same particle weight. Many of the codes more
    advanced features require per-particle weighting so it is enabled by
    default. Use this flag to disable per- particle weighting if you
    need to save on memory, but it this option is recommended only for
    advanced users.
-   NO_TRACER_PARTICLES - This flag will disable the option to specify
    one or more species as zero-current particles. Zero-current
    particles move about as would a normal particle with the same charge
    and mass, but they do not generate any current and are therefore
    passive elements in the simulation. Zero-current particles should be
    included in collisions to ensure they move identically to ordinary
    particles. The implementation of zero-current particles requires an
    additional "IF" clause in the particle push, so it has a slight
    performance impact. If you do not require the feature then setting
    this flag will give a slight performance improvement.
    <span style="color: red; font-weight: bold;">WARNING:</span> Since
    the particles effectively have zero weight in terms of their
    numerical heating properties, they do not always behave in the same
    way that an ordinary particle with weight would behave and this can
    sometimes lead to unexpected behaviour. If the purpose is merely to
    track a subset of a particle species to use as output then a better
    mechanism to use is "persistent subsets" (see
    [here][Input_deck_subset]). In version 5.0, this
    flag will be and replaced with "ZERO_CURRENT_PARTICLES".
-   NO_PARTICLE_PROBES - For laser plasma interaction studies it can
    sometimes be useful to be able to record information about particles
    which cross a plane in the simulation. Since this requires the code
    to check whether each particles has crossed the plane in the
    particles pusher and also to store copies of particles until the
    next output dump, it is a heavyweight diagnostic. If you don't
    require the diagnostic you can set this flag to disable it.
-   PARTICLE_SHAPE_TOPHAT - By default, the code uses a first order
    b-spline (triangle) shape function to represent particles giving
    third order particle weighting. Using this flag changes the particle
    representation to that of a top-hat function (0th order b-spline
    yielding a second order weighting).
-   PARTICLE_SHAPE_BSPLINE3 - This flag changes the particle
    representation to that of a 3rd order b-spline shape function (5th
    order weighting).
-   PARTICLE_ID - When this option is enabled, all particles are
    assigned a unique identification number when writing particle data
    to file. This number can then be used to track the progress of a
    particle during the simulation.
-   PARTICLE_ID4 - This does the same as the previous option except it
    uses a 4-byte integer instead of an 8-byte one. Whilst this saves
    storage space, care must be taken that the number does not overflow.

-   PHOTONS - This enables support for photon particle types in the
    code. These are a pre-requisite for modelling synchrotron emission,
    radiation reaction and pair production (see
    [here][Input_deck_qed]).
-   TRIDENT_PHOTONS - This enables support for virtual photons which
    are used by the Trident process for pair production.
-   PREFETCH - This enables an Intel-specific code optimisation.
-   PARSER_DEBUG - The code outputs more detailed information whilst
    parsing the input deck. This is a debug mode for code development.
-   PARTICLE_DEBUG - Each particle is additionally tagged with
    information about which processor it is currently on, and which
    processor it started on. This is a debug mode for code development.
-   MPI_DEBUG - This option installs an error handler for MPI calls
    which should aid tracking down some MPI related errors.
-   SIMPLIFY_DEBUG - This option enables debugging code related to the
    deck parser simplification routine.
-   NO_IO - This option disables all file I/O which can be useful when
    doing benchmarking.
-   COLLISIONS_TEST - This enables some routines for debugging the
    collision routines. It completely alters the behaviour of the code.
    This flag should never be enabled by the end user.
-   PER_PARTICLE_CHARGE_MASS - By default, the particle charge and
    mass are specified on a per-species basis. With this flag enabled,
    charge and mass become a per-particle property. This is a legacy
    flag which will be removed soon.
-   PARSER_CHECKING - Setting this flag adds code which checks for
    valid values on evaluated deck expressions. This slows down the code
    but may be required if floating point exceptions are enabled.
-   WORK_DONE_INTEGRATED - This enables support for tracking the work
    done on each particle by the electric field. Note that this
    increases the size of each particle by 48 bytes. The information
    gathered can be output to file using the "work_{x,y,z}" and
    "work_{x,y,z}_total" dumpmasks. See
    [here][Input_deck_output_block__particle_variables]
-   DELTAF_METHOD - Compile the code to use the delta-f method to
    represent particles rather than standard PIC. Note that this
    completely changes the code behaviour and should not be enabled for
    normal use. See [here][Using_delta_f].
-   DELTAF_DEBUG - Add debug code for the delta-f method.

-   HC_PUSH - Use the push from [Higuera and
    Cary](https://doi.org/10.1063/1.4979989) rather than the Boris push.
    This is slightly slower than the Boris push but gives the correct
    $\mathbf{E} \times \mathbf{B}$ velocity, improving performance for
    highly relativistic simulations.

-   NO_USE_ISATTY - When printing the initial welcome message, EPOCH
    makes use of the C-library's isatty function. This requires
    Fortran2003 features that might not be available on all platforms.
    The flag allows this functionality to be disabled on platforms that
    don't support it.
-   NO_MPI3 - This compiler flag allows the user to disable MPI-3
    features such as the "MPI_TYPE_SIZE_X" routine. This allows the
    code to be compiled against older versions of the MPI library. The
    flag should only be enabled if the code fails to compile without it.

# Errors for unspecified precompiler directives {#errors_for_unspecified_precompiler_directives}

If a user requests an option which the code has not been compiled to
support then the code will give an error or warning message as follows:

     *** ERROR ***
     Unable to set "use_qed=T" in the "qed" block.
     Please recompile with the -DPHOTONS preprocessor flag.

# Other Makefile flags {#other_makefile_flags}

It is also possible to pass other flags to the compiler. In "Makefile"
there is a line which reads

    FFLAGS = -O3 -fast

The two commands to the right are compiler flags and are passed
unaltered to the FORTRAN compiler. Change this line to add any
additional flags required by your compiler.

By default, EPOCH will write a copy of the source code and input decks
into each restart dump. This can be very useful since a restart dump
contains an exact copy of the code which was used to generate it,
ensuring that you can always regenerate the data or continue running
from a restart. The output can be prevented by using "dump_source_code
= F" and "dump_input_deck = F" in the output block. However, the
functionality is difficult to build on some platforms so the Makefile
contains a line for bypassing this section of the build process. Just
below all the DEFINE flags there is the following line:

    #ENCODED_SOURCE = epoch_source_info_dummy.o

Just uncomment this line and source code in restart dumps will be
permanently disabled.

# Next section {#next_section}

[Running EPOCH][Running]


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
