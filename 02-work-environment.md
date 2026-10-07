---
title: "Work Environment for ePIC Reconstruction Software"
teaching: 10
exercises: 10
---

::::::::::::::::::::::::::::::::::::::::::::: questions

- How do I setup a development copy of the EICrecon repository?

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: objectives

- Clone EICrecon repository and build with eic-shell.
- Obtain a simulated data file.

:::::::::::::::::::::::::::::::::::::::::::::

## How do I setup development copy for EICrecon?

A development copy includes a working directory and the EICrecon code. By far, the easiest way to set this
up is using `eic-shell` as outlined in the [Setting up your environment tutorial](https://eic.github.io/tutorial-setting-up-environment/).

Although `eic-shell` comes with a prebuilt copy of EICrecon (`eicrecon`), we will clone the EICrecon repository
so we can modify it and submit changes back to GitHub later:

```bash
git clone https://github.com/eic/EICrecon
```

or, if you have SSH keys set up on github

```bash
git clone git@github.com:eic/EICrecon
```

Check that you can build EICrecon using the packages in `eic-shell` (this may take a while...):

```bash
cd EICrecon
cmake -S . -B build
cmake --build build --target install
```

If you are not familiar with cmake, the first command above (`cmake -S . -B build`) will create a directory `build`
and place files there to drive the build of the project in the source directory `.` (i.e. the current dirctory).
The second cmake command (`cmake --build build --target install`) actually performs the build and installs
the compiled plugins, exectuables, etc.

::::::::::::::::::::::::::::::::::::::::::::: callout

Note: The `-j8` option can be added to tells CMake to use 8 threads to compile. If you have more cores available,
then set this number to an appropriate value.

```bash
cmake --build build --target install -- -j8
```

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise

- Use the commands above to setup your working directory and build *EICrecon*.

::::::::::::::: solution

After `cmake --build build --target install` finishes, you should have an `EICrecon/build`
directory and an installed `eicrecon` executable together with an `EICrecon/bin/eicrecon-this.sh`
environment script. The build completes without errors (it may take several minutes).

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

## How do I run EICrecon?

`eicrecon` is the main reconstruction executable. To run it though, you should add it to your `PATH` and set up
any other environment parameters needed. Do this by sourcing the `eicrecon-this.sh` file that should have been
created and installed into the `bin` directory in the previous step. Once you have done that, run `eicrecon` with
no arguments to see that it is found and prints the usage statement.

```bash
source bin/eicrecon-this.sh
```

::::::::::::::::::::::::::::::::::::::::::::: callout

Note: If you are using the prebuilt `eicrecon` executable in `eic-shell`, the environment will aready be set.

:::::::::::::::::::::::::::::::::::::::::::::

Now you should be able to run the `eicrecon` command, and without options it will give information on how to run it:

```bash
$ eicrecon

Usage:
    eicrecon [options] source1 source2 ...

Description:
    Command-line interface for running JANA plugins. This can be used to
    read in events and process them. Command-line flags control configuration
    while additional arguments denote input files, which are to be loaded and
    processed by the appropriate EventSource plugin.

Options:
   -h   --help                  Display this message
   -v   --version               Display version information
   -j   --janaversion           Display JANA version information
   -c   --configs               Display configuration parameters
   -l   --loadconfigs <file>    Load configuration parameters from file
   -d   --dumpconfigs <file>    Dump configuration parameters to file
   -b   --benchmark             Run in benchmark mode
   -L   --list-factories        List all the factories without running
   -Pkey=value                  Specify a configuration parameter
   -Pplugin:param=value         Specify a parameter value for a plugin

   --list-default-plugins       List all the default plugins
   --list-available-plugins     List plugins at $JANA_PLUGIN_PATH and $EICrecon_MY


Example:
    eicrecon -Pplugins=plugin1,plugin2,plugin3 -Pnthreads=8 infile.root
    eicrecon -Ppodio:print_type_table=1 infile.root
```

The usage statement gives several command line options. Two of the most important ones are the
`-l` and `-Pkey=value` options. Both of these allow you to set *configuration parameters*
in the job. These are how you can modify the behavior of the job. Configuration parameters
will pretty much always have default values set by algorithm authors so it is often not necessary
to set this yourself. If you need to though, these are how you do it.

- Use the `-Pkey=value` form if you want to set the value directly on the command line.
  You may pass mutiple options like this.
- The `-l` option is used to specify a configuration file where you may set a large number
  of values. The file format is one parameter per line with one or more spaces separating the
  configuration parameter name and its value. Empty lines are OK and `#` can be used to specify
  comments.

## Get a simulated data file

The [Simulations with npsim and Geant4 tutorial](https://eic.github.io/tutorial-simulations-using-npsim-and-geant4/)
described how to generate a simulated data file. If you
followed the exercises in that tutorial you can use a file you generated there. If not, then
you can quickly generate a small file with the following command:

```bash
ddsim -N 100 \
  --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml \
  --outputFile pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.edm4hep.root \
  --inputFile root://dtn-eic.jlab.org//volatile/eic/EPIC/EVGEN/DIS/NC/10x100/minQ2=1/pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv_1.hepmc3.tree.root
```

::::::::::::::::::::::::::::::::::::::::::::: callout

Note: The backslash characters, `\`, allow the line to be continued on the next line.

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise

- Run `eicrecon` over your simulated data file by giving it as an argument to the program, e.g.

```bash
eicrecon pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.edm4hep.root
```

::::::::::::::: solution

`eicrecon` processes the events in the file and prints progress to the terminal, finishing without
errors. By default no output file is written; the next section shows how to request output
collections.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

## Generating a podio output file

To write reconstructed values to an output file, you need to tell *eicrecon* what to write.
There are several options available, but the mosrt useful one is *podio:output_collections*.
This is a comma separated list of colelctions to write to the output file. For example:

```bash
eicrecon -Ppodio:output_collections=ReconstructedParticles pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.edm4hep.root
```

To see a list of possible collections, run `eicrecon -L`.

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise

- Use `eicrecon` to generate an output file with both `ReconstructedParticles` and `EcalEndcapNRawHits`.

::::::::::::::: solution

Pass both collection names as a comma-separated list to `podio:output_collections`:

```bash
eicrecon -Ppodio:output_collections=ReconstructedParticles,EcalEndcapNRawHits pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.edm4hep.root
```

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: keypoints

- Use eicrecon executable to run reconstruction on a podio input file and to create podio output file.

:::::::::::::::::::::::::::::::::::::::::::::
