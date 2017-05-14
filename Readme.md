# Documentation for the Submit Daemon *FireDeamon* for Use in a Runtime-Restricted Envorinment --- Torsten Sachse

**Content Description:** This document serves as documentation of the submit daemon called *FireDeamon* created over the course of my Ph.D.-Thesis. It
elaborates on the daemon's working principle and what functionallity it provides. There is also a section that only explains how to use the daemon.

**Copyright Notice:** Copyright (c) 2017 Torsten Sachse `torsten.sache@uni-jena.de`. This software is licensed under the GNU General Public License
v3. See the file `COPYING` for license details.

## INSTALLATION

You can get FireDeamon easily if you have "git" installed by executing

> git clone git://github.com/razziel89/FireDeamon.git

Now go into the directory and see the file `install.sh`, which is the script that installs the daemon. Be referred to the content of that file to see
what it does, if you are interested. Copy the config file `deamonrc` to `$HOME/.deamonrc` and change what you want to change (Do you want e-mail
notifications? To what address?). Then, copy the file `CONFIG` to `CONFIG.rc` and edit the new file according to the comments within the file.  You
will have to add information about the software running on the cluster and where you want to install *FireDeamon*. After having done so, you can
execute the install script `install.sh`. It will only install such jobtypes that are compatible with the selections in `CONFIG.rc`.

I recommend installing at least **Python**, **OpenBabel**, **NumPy** and  **SciPy** (some of them might even be required for certain functionality). In
addition, I recommend installing **Gnuplot**, **PoVRay** and **VMD** for easy evaluation. You should also consider installing **ManipulateAggregates**
(see below for a link).

## General Remarks

In computer sciences, the term *daemon* usually refers to a programme that runs in the background, does not allow for direct interaction with the user
but provides some automated functionality that is triggered by certain events. This daemon is written mainly in bash and is designed to run on a
system that supports the bash interpreter, e.g., unix clusters. It has only ever been tested on such systems.

This daemon has been developed for the task of computationally screening a wide variety of different molecules for their application in organic
photovoltaics, which is both tedious and demanding at the same time. However, it can also be used for a wide variety of ther tasks. A lot of the time,
the same jobs would be run over and over again with slightly differing input in order to investigate the effect of, e.g., certain chemical changes in
the molecules themselves on the resulting properties (e.g., absorption spectra). This would take up a significant amount of time when managed
manually, which cannot be used for less repetitive tasks. Hence, I created this daemon. Please note that calling it `deamon` instead of `daemon` is a
*deliberate typo*.

### Design Choices

- *Languages:* Most of the daemon is written in bash, although technically every language can be used to extend the daemon's functionality
    as long as the added file can be executed somehow. Scripting languages in general provide the advantage of being more high-level, i.e., complex
    functionality can be implemented in comparatively little time at the expense of processor time required for execution. For a programme that merely
    organises computations, I consider this choice adequate.
- *Stupidity:* The daemon has been designed to be as stupid as possible, meaning that nothing is actually enforced when creating a new module. To this
    end, the daemon's definition of a job is kept as simple as possible, i.e., consisting of preparation, execution with possible restarts and, last
    but not least, postprocessing with none of the aforementioned four steps being mandatory. This also means that the daemon does not hover and wait
    for a certain step of a multi-step computation to finish to start the next step. It expects the previous step to prepare the next step upon
    successful termination and submit the new job.
- *Jobtype Basis:* The daemon *thinks* in jobtypes. A jobtype comprises a (possibly multi-stranded) chain of computations that belong together (for
    instance because all the results of the corresponding computations are needed to draw a comprehensive picture of the problem at hand)
    content-wise. Each single job that belongs to the jobtype is managed by modules and can have a module for each of preparation, restarting and
    postprocessing.
- *Command Line:* The daemon consists of command line utilities because this allows for easy batch processing.

### Current Functionality

This section quickly describes the current functionality of the daemon. It will not go into details about any of the basic ideas of, e.g., quantum
chemistry. Please be referred to an online search for that.
Currently, GPU support will disable multi-node support.
The following jobtypes are currently supported, including their requirements (dependencies
not included in the lists):

- *CI-Search*: an **ab initio** search for a conical intersection using **TeraChem**
- *ConformerSearch*: search for a molecule's rotamers using **OpenBabel**
- *CTP*: a charge-transport simulation using **Gaussian**, **ManipulateAggregates**, **OpenBabel** and **VOTCA-CTP**
- *energy*: a single-point calculation using **TeraChem**
- *EnergyScan*: predict aggregate geometries using **ManipulateAggregates** and **OpenBabel**
- *frequency*: a frequency calculation using **TeraChem**
- *gradient*: a gradient calculation using **TeraChem**
- *HOMOLUMO*: extract frontier orbital energies using **TeraChem** by first performing a geometry optimization and then a single point calculation
  together with a reference molecule to get energy differences between the frontier orbital energies of both molecules
- *initcond*: create a so-called **initial condition** (similar to **frequency** but results are mass-weighed) using **TeraChem**
- *IonizationPotential*: compute the ionization potential and electron affinity via a delta-SCF method using **TeraChem**
- *mdcrd2xyz*: convert a mdcrd file (**AMBER** trajectory) to **xyz** format using **mdtraj**
- *minimize*: a quantum chemistry geometry optimization using **TeraChem**
- *OmegaTuning*: determine the best range-separation parameter for a molecule using **TeraChem** and **ManipulateAggregates**
- *PotentialPlot*: create files that can be used by **ManipulateAggregates** to create plots of electrostatic potential distributions on different
  molecular surfaces using **ManipulateAggregates** and **OpenBabel**
- *SH*: perform a **surface hopping** calculation using **NewtonX**
- *TDDFT*: a time-dependent DFT computatiion using **TeraChem**
- *TDDFTminimize*: like **minimize** but for an excited state
- *TeraChemMD:* an **ab initio** molecular dynamics run using **TeraChem**

Please note that not all of these jobtypes support all four steps. For instance, the jobtype *SH* does currently not support preprocessing. Currently,
the following queueing systems are supported:

- *SGE* (Sun Grid Engine)
- *TORQUE* (Terascale Open-source Resource and QUEue Manager)
- *SLURM* (SLURM workload manager)

Currently, the following computational software is supported (not all software that can technically perform a task can be used by the daemon for that
task):

- *TERACHEM* (the TeraChem GPU-accelerated quantum chemistry programme, highly recommended)
- *MANIPAGG* (*ManipulateAggregates* - a programme to estimate aggregate geometries and evaluate quantum chemistry data and much more. See
  [https://github.com/razziel89/ManipulateAggregates](https://github.com/razziel89/ManipulateAggregates) and also consider it's dependencies
- *OBABEL* (*OpenBabel* - generate conformers and translate between chemical file formats)
- *NEWTONX* (surface hopping ab initio dyamics, among other things)
- *GAUSSIAN* (a CPU-based quantum chemistry software package, support limited to the *CTP* jobtype)
- *MDTRAJ* (Convert mdcrd-files to xyz-files)
- *VOTCACTP* (A software suite that can perform charge transfer simulations)

### Order of Script Execution

Before a job can be preprocessed, saved, restarted or postprocessed, the daemon checks whether the corresponding module is present. This can either be
a directory called `$ACTIONdir/$jobtype.$ACTION.d` or a single file called `$ACTIONdir/$jobtype.$ACTION`. Here, `$ACTIONdir` is the directory
containing all scripts for the corresponding action, `$ACTION` is the corresponding action (e.g. `preprocess`) and `$jobtype` is the type of the job.
If none of these are found, the daemon considers the jobtype to not be implemented on the current host (it might be implemented in the daemon but the
content of `CONFIG.rc` might not say that required software is installed). 

## How to Use
 
This section describes how to use the daemon once it has been installed.  Before reading this, it is recommended (but by no means required) to
familiarize yourself with the folder structure of the daemon directory and the daemon `deamon.sh` itself to get to know the basic workflow. This
section will describe the most important files and directories that comprise the daemon and the naming convention deemed appropriate for everything. 

### Naming Conventions

So far, the naming convention in the repository is quite simple. If you see a file or directory ending in something like
`QM=GAUSSIAN.OP=MANIPAGG.OP=OBABEL.OP=VOTCACTP`, it means that whatever functionality is contained within requires `MANIPAGG`, `OBABEL` and `VOTCACTP`
to be set to supported in `CONFIG.rc`, the first one in the section `QM` and the last two in the section `OP`. If a file ends in something like
`QS=SGE,TORQUE`, it means that either `SGE` or `TORQUE` are required. If a file ends in something like `QS:SGE,TORQUE`, it means that the
functionality contained within does work with neither `SGE` nor `TORQUE`. Files whose names indicate requirements that are not met by `CONFIG.rc` will
not be copied over during installation. Furthermore, the two letters `.d` at the end of a name or directly before the requirements definition means
that you are looking at a directory. In the directories `preprocess`, `restart` and `postprocess` this convention is required to differentiate between
executing something directly or executing every executable in the directory in succession.

For the files created during the preprocess step, I follow the following naming conventions:

   Name of File or Part of Filename         |   Description
--------------------------------------------|--------------------------------------------------------------------------------------------
   `NAME.JOBTYPE.submit.sh`                 | - script to be submitted to the cluster's queue, i.e., the one organizing the computation
   `NAME`                                   | - an arbitrary string, has to be ASCII without spaces or newlines, used by the user to identify the computation
   `JOBTYPE`                                | - the jobtype assigned to the job organized by this script, used by the daemon to select appropriate preprocessing, restarting and postprocessing procedures
   `submit.sh`                              | - the mandatory suffix determining the type of file (here: a submit script)
   `NAME.JOBTYPE.tc_input`                  | - TeraChem input deck
   `NAME.EXT`                               | - a geometry file that contains the molecule / aggregate / arrangement of atoms to be used in the computation
   `JOBTYPE.preprocess`                     | - script that prepares all the necessary files for a computation and submits a script of the type `submit.sh` (all located in the `preprocess` subdirectory)
   `JOBTYPE.preprocess.d`                   | - a directory containing files to be executed in order that do the same as an equivalent `JOBTYPE.preprocess`
   `JOBTYPE.restart`                        | - similar to `JOBTYPE.preprocess` but performs a restart of a calculation that was killed because it ran out of time (all located in the `restart` subdirectory)
   `JOBTYPE.postprocess`                    | - similar to `JOBTYPE.preprocess` but postprocesses a calculation (all located in the `postprocess` subdirectory)
   `JOBTYPE.master`                         | - similar to `JOBTYPE.postprocess` but postprocesses only a master once every slave has finished (all located in the `postprocess` subdirectory)

### Important Scripts, Files and Directories
 
This section describes some of the most useful scripts that comprise the daemon. Scripts that are useful for manual postprocessing of computations or
other data will also be listed. The following table details scripts that allow for user interaction. All of them will print a useful help message when
given the command line option `--help`. Be referred to that help message for more information than given here.  The name in brackets, if given, is the
actual file and the name without brackets is only a sybmolic link whose name is shorter making it easier to use.

   Name of File (Long Name)  |   Description
-------------------------------|-----------------------------------------------------------------------------------------------------------
   `change_jobtype`            | - allows for changing the jobtype of a single job that is identified by its job-id
   `demail` (`send_email_via_deamon`)  |  - allows sending emails to the e-mail address that is used by the daemon for notification
   `dkill` (`stop_deamon`)     | - terminate the daemon if possible
   `dqdel` (`kill_job_via_deamon`)     |   - given a set of space-separated job-ids, the daemon will terminate those jobs
   `dqsub` (`submit_job_to_deamon`)    |   - submit a job that is fully configured using a `deamon_config` and a geometry file
   `dscriptqsub` (`submit_script_to_deamon`)  |  - submit a job to the daemon that is not configured by a `deamon_config`-file but its own script instead
   `dstart` (`start_deamon`)   | - if you ever have to start the daemon manually, this script will do the trick
   `resource_deamonrc`         | - will cause the daemon to read in its config files again, should be used if changes to the config files were made but the daemon shall not be restarted
   `softmutation` (`softmutation.sh`) | - will displace/distort a geometry (xyz-file) along one of its normal modes

The following table details scripts that do not allow for user interaction as well as other important files, such as several config files. Please see
comments in the files themselves for further information.

   Name of Script               |   Description
--------------------------------|-----------------------------------------------------------------------------------------------------------
   `deamon.sh`                  | - the main daemon programme itself that organizes all the jobs
   `deamonrc`                   | - the default config file that determines things like notifications and email-addresses, you can have a customized copy in your home directory which overrides the default one
   `install.sh`                 | - install and reinstall the daemon
   `sh.vim`                     | - a config file for my most favourite text editor *vim* that allows for proper syntax highlighting of the `deamon_config` files
   `sge_wrapper`                | - a wrapper for all the functionality associated with the different queueing systems to be able to treat every system the same (there is one file for every supported queueing system)


The following table details all the directories that comprise the daemon. Some files are required to reside in the corresponding directory, e.g. all
files related to postprocessing need to be in the folder `postprocess`.

   Name of Directory            |   Description
--------------------------------|-----------------------------------------------------------------------------------------------------------
   `deamon_config.d`            | - the directory containing all the parts that make up the main/default config file for a job that is to be run by the daemon
   `deamon_functions.d`         | - the directory containing all the parts that make up the file that defines many functions that are used excessively during preprocessing, postprocessing and restarting jobs
   `executables.default.d`      | - the directory containing all the parts that make up the file that defines how executables should be accessed for the individual jobtypes
   `postprocess`                | - contains module files and directories to postprocess a job
   `preprocess`                 | - contains module files and directories to preprocess a job
   `restart`                    | - contains module files and directories to restart a job
   `savedata`                   | - contains module files and directories to save a job's data (GENERIC used if none present for module)
   `scripts`                    | - some important scripts as described in the above table are contained within
   `submit_scripts`             | - contains most of the scripts allowing for user interaction as described in the above table
   `exe`                        | - contains all the scripts (or symlinks) that are useful to have directly available (this directory, once installed, should be in your `$PATH`)
   `relations`                  | - when executing jobtypes one after the other using `dqsub`'s `--multi-conf`, `--multi-geom`, `--reap` and `--sow` switches, the files and directories in this directory describe how the output of one is used as the input for the next one
   `fast_deamon`                | - this directory contains a `C++` programme that is used to determine the state a job is currently in depending on its state reported by the queueing system and the time (will be compiled upon installation, uses much `CPU` when done in *Bash*)
   `FIRE_DEAMON_DIRECTORY`      | - the directory wherein the daemon is installed (not where the repository is located), stored in an environmental variable and exported in your `$HOME/.bashrc`
   `DEAMON_PIPE`                | - the location of the file used to communicate with the daemon, stored in an environmental variable and exported in your `$HOME/.bashrc`

### Multi Executable Functionality

Imagine a case where you run a computation using a programme called `prog` in version `1.0` to which I will refer as `p1`. New functionality has been
added to `p2` which is the same programme in version `2.0`. You often perform two different tasks `T1` and `T2`. `T2` strongly benefits from the new
functionality in `p2` but some of the functionality you need for `T1` is unfortunately broken in `p2` so you want to use `p2` for `T2` and `p1` for
`T1`. However, `T1` and `T2` are quite similar and have so far been prepared by the same preprocess script. I don't think it's desirable to make an
altered copy of the preprocess script and hard-code the executable. Instead, it would be better to be able to use different executables as you see fit
without having to alter any preprocess script.

I realized this scenario in the following way. Whenever a job is preprocessed, restarted or postprocessed, the file `deamon_functions` in the main
daemon's installation directory is sourced. This file was created from a lot of snippets in `deamon_functions.d` during installation.  Depending on
the environmental variable `executables` (which can be defined in a `deamon_config` file), the following is being done:

- source default executable definitions in `$FIRE_DEAMON_DIRECTORY/executables.default`
- look for the directory with the user's definitions of executables `$HOME/.deamonrc.d`
- if found, see whether the file `$HOME/.deamonrc.d/$executables` exists
- if so, source it
- source the function definitions in `$FIRE_DEAMON_DIRECTORY/deamon_functions` where the stuff that has been sourced from the above files is used

### Combining Multiple Jobs Into One Result

Sometimes it is required to combine the results of multiple computations into one bigger picture. Take the jobtype `OmegaTuning` for example: several
single point calculations for several values of the range-separation parameter are performed to determine the optimum value of that parameter for the
system at hand. This is realized in two ways:

- Among a set of computations that are submitted, one is called a `master` and all the others are `slaves` (see the `--help` message of the script
   `command_start` for some more details). For each job in this set, the ordinary postprocessing is performed. Once that has been done for all jobs
   in the set, the master's `.master` postprocess script is also being executed in the mater's directory. This `.master` postprocess script has to
   fulfill the same naming convention as the ordinary postprocess script but with the word `postprocess` replaced by the word `master`.
- A computation can prepare another one (or several) during its postprocessing step and can also submit it. This way, for instance, a geometry
    optimization can be performed that automatically starts a frequency calculation or the computation of absorption spectra afterwards for the
    optimized geometry.
