---
title: "Using a cluster: Accessing software"
teaching: 30
exercises: 15
questions:
- "How do we load and unload software packages?"
objectives:
- "Understand how to load and use a software package."
keypoints:
- "Load software with `module load softwareName`"
- "Unload software with `module purge`"
- "The module system handles software versioning and package conflicts for you automatically."
- "You can edit your `.bashrc` file to automatically load a software package."
---

On a high-performance computing system, no software is loaded by default.
If we want to use a software package, we will need to "load" it ourselves.

Before we start using individual software packages, however, 
we should understand the reasoning behind this approach.
The two biggest factors are software incompatibilities and versioning.

Software incompatibility is a major headache for programmers.
Sometimes the presence (or absence) 
of a software package will break others that depend on it.
Two of the most famous examples are Python 2 and 3 and C compiler versions.
Python 3 famously provides a `python` command 
that conflicts with that provided by Python 2. 
Software compiled against a newer version of the C libraries 
and then used when they are not present will result in a nasty
`'GLIBCXX_3.4.20' not found` error, for instance. 

Software versioning is another common issue.
A team might depend on a certain package version for their research project - 
if the software version was to change (for instance, if a package was updated),
it might affect their results.
Having access to multiple software versions allow a set of researchers to take 
software version out of the equation if results are weird.

## Environment modules (Lmod)

Environment modules are the solution to these problems.
A module is a self-contained software package - 
it contains all of the files required to run a software packace 
and loads required dependencies.

To see available software modules, use `module avail`

```
module avail
```
{: .bash}
```
------------------------- /sw/Modules/doc/modulefiles --------------------------
HOWTO/IntelMPI                      README/AppSupport
HOWTO/Libraries                     README/CompilerEnvironmentVariables
HOWTO/OpenMPI                       README/Euramoo_Migration
HOWTO/PersonalModules               README/HyperWorks
HOWTO/R-Packages                    README/Modules
HOWTO/X11                           README/Rolls+Modules

--------------------------- /sw/Modules/DEPRECIATED ----------------------------
gromacs/5.1.4-intel

------------------------------ /sw/Modules/INDEX -------------------------------
anaconda/4.2.0                intel_xe/2016.2.181
anaconda/4.3.1                isolve/v3_0
ansys/17.0(default)           Java/1.7.0_79
ansys/17.1                    Java/1.8.0_45
ansys/18.0                    krona/201712
ansys/18.1                    libtool/2.4.2-GCC-4.9.2
ansys_edt/17.1                libtool/2.4.5-GCC-4.9.2
Autoconf/2.69-GCC-4.9.2       macs/2-2.1.1
......which p

```
{: .output}

## Loading and unloading software

To load a software module, use `module load`.
In this example we will use Python 3.

Intially, Python 3 is not loaded. 

```
which python3
```
{: .bash}
```
/usr/bin/which: no python3 in (/opt/software/slurm/16.05.9/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2017/avx2/Compiler/intel2016.4/openmpi/2.1.1/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2017/Core/imkl/11.3.4.258/mkl/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2017/Core/imkl/11.3.4.258/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2017/Core/ifort/2016.4.258/compilers_and_libraries_2016.4.258/linux/bin/intel64:/cvmfs/soft.computecanada.ca/nix/var/nix/profiles/gcc-5.4.0/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2017/Core/icc/2016.4.258/compilers_and_libraries_2016.4.258/linux/bin/intel64:/opt/software/bin:/opt/puppetlabs/puppet/bin:/opt/software/slurm/current/bin:/opt/software/slurm/bin:/cvmfs/soft.computecanada.ca/easybuild/bin:/cvmfs/soft.computecanada.ca/nix/var/nix/profiles/16.09/bin:/cvmfs/soft.computecanada.ca/nix/var/nix/profiles/16.09/sbin:/cvmfs/soft.computecanada.ca/custom/bin:/opt/software/slurm/current/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/yourUsername/.local/bin:/home/yourUsername/bin)
```
{: .output}

We can load the `python3` command with `module load`.

```
module load python
which python3
```
{: .bash}
```
/opt/python/bin/python3
```
{: .output}

So what just happened?

To understand the output, first we need to understand the nature of the 
`$PATH` environment variable.
`$PATH` is a special ennvironment variable that controls where a UNIX system looks for software.
Specifically `$PATH` is a list of directories (separated by `:`)
that the OS searches through for a command before giving up and telling us it can't find it.
As with all environment variables we can print it out using `echo`.

```
echo $PATH
```
{: .bash}
```
/opt/intel/composer_xe_2017.4/bin/intel64:/opt/intel/composer_xe_2017.4/mpirt/bin/intel64:/opt/intel/composer_xe_2017.4/debugger/gdb/intel64_mic/bin:/opt/gnu/gcc/bin:/opt/python/bin:/usr/lib64/qt-3.3/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/opt/sdsc/bin:/opt/sdsc/sbin:/opt/bio/ncbi/bin:/opt/bio/mpiblast/bin:/opt/bio/EMBOSS/bin:/opt/bio/clustalw/bin:/opt/bio/tcoffee/bin:/opt/bio/hmmer/bin:/opt/bio/phylip/exe:/opt/bio/mrbayes:/opt/bio/fasta:/opt/bio/glimmer/bin:/opt/bio/glimmer/scripts:/opt/bio/gromacs/bin:/opt/bio/gmap/bin:/opt/bio/tigr/bin:/opt/bio/autodocksuite/bin:/opt/bio/wgs/bin:/opt/ganglia/bin:/opt/ganglia/sbin:/usr/java/latest/bin:/opt/maui/bin:/opt/torque/bin:/opt/torque/sbin:/opt/pbs/bin:/opt/rocks/bin:/opt/rocks/sbin:/home/username/bin
```
{: .output}

You'll notice a similarity to the output of the `which` command. 
In this case, there's only one difference:
the `/opt/python/bin` directory at the beginning.
When we ran `module load python`, 
it added this directory to the beginning of our `$PATH`.
Let's examine what's there:

```
ls /opt/python/bin

```
{: .bash}
```
2to3           easy_install      nosetests      pip3       python2.7-config   python-config
2to3-3.5       easy_install-2.7  nosetests-2.7  pip3.5     python2-config     pyvenv-3.5
cygd-3.5       easy_install-3.5  nosetests-3.5  pydoc      python3            smtpd.py
cygdb          f2py2.7           pbr            pydoc3     python3.5
cython         f2py3.5           pbr-3.5        pydoc3.5   python3.5-config
cython-3.5     idle              pip            python     python3.5m
cythonize      idle3             pip2           python2    python3.5m-config
cythonize-3.5  idle3.5           pip2.7         python2.7  python3-config

```
{: .output}

Taking this to it's conclusion, `module load` will add software to your `$PATH`.
It "loads" software.
A special note on this - 
depending on which version of the `module` program that is installed at your site,
`module load` will also load required software dependencies.
To demonstrate, let's use `module list`.
`module list` shows all loaded software modules.

```
module list
```
{: .bash}
```
Currently Loaded Modules:
  1) nixpkgs/.16.09  (H,S)   3) gcccore/.5.4.0    (H)   5) intel/2016.4  (t)   7) StdEnv/2016.4 (S)
  2) icc/.2016.4.258 (H)     4) ifort/.2016.4.258 (H)   6) openmpi/2.1.1 (m)   8) python/3.5.2  (t)

  Where:
   S:  Module is Sticky, requires --force to unload or purge
   m:  MPI implementations / Implémentations MPI
   t:  Tools for development / Outils de développement
   H:             Hidden Module
```
{: .output}

```
module load beast
module list
```
{: .bash}
```
Currently Loaded Modulefiles:
  1) gnu/7.1.0       3) mkl/2017.3      5) R/3.4.0
  2) intel/2017.4    4) python/2.7.13


```
{: .output}

So in this case, loading the `R` module (a bioinformatics software package),
also loaded `intel/2017.3` as well as others.
Let's try unloading the `beast` package.

```
module unload beast
module list
```
{: .bash}
```
Currently Loaded Modulefiles:
  1) gnu/7.1.0       2) python/2.7.13

```
{: .output}

So using `module unload` "un-loads" a module along with it's dependencies.
If we wanted to unload everything at once, we could run `module purge` (unloads everything).

```
module purge
```
{: .bash}
```
No Modulefiles Currently Loaded.
```
{: .output}

Note that `module purge` is informative. 
It lets us know that all but a default set of packages have been unloaded
(and how to actually unload these if we truly so desired).

## Software versioning

So far, we've learned how to load and unload software packages.
This is very useful.
However, we have not yet addressed the issue of software versioning.
At some point or other, 
you will run into issues where only one particular version of some software will be suitable.
Perhaps a key bugfix only happened in a certain version, 
or version X broke compatibility with a file format you use.
In either of these example cases, 
it helps to be very specific about what software is loaded.

Let's examine the output of `module avail` more closely.

```
module avail
```
{: .bash}
```
----------------------------------------------------------- Core Modules -----------------------------------------------------------
   StdEnv/2016.4 (S,L)     imkl/11.3.4.258 (L,math,D:11)      mcr/R2014b         (t)          python/3.5.2     (t,D:3:3.5)
   bioperl/1.7.1 (bio)     imkl/2017.1.132 (math,2017)        mcr/R2015a         (t)          qt/4.8.7         (t)
   eclipse/4.6.0 (t)       impute2/2.3.2   (bio)              mcr/R2015b         (t)          qt/5.6.1         (t,D)
   eigen/3.3.2   (math)    intel/2016.4    (L,t,D:16:2016)    mcr/R2016a         (t)          signalp/4.1f     (bio)
   fastqc/0.11.5 (bio)     intel/2017.1    (t,17:2017)        mcr/R2016b         (t,D)        spark/2.1.0      (t)
   g2clib/1.6.0            jasper/1.900.1  (vis)              minimac2/2014.9.15 (bio)        spark/2.1.1      (t,D)
   g2lib/1.4.0             java/1.8.0_121  (L,t)              perl/5.22.2        (t)          tbb/2017.2.132   (t)
   gatk/3.7      (bio)     mach/1.0.18     (bio)              pgi/17.3           (t)          tmhmm/2.0c       (bio)
   gcc/4.8.5     (t)       mcr/R2013a      (t)                picard/2.1.1       (bio)        trimmomatic/0.36 (bio)
   gcc/5.4.0     (t,D)     mcr/R2014a      (t)                python/2.7.13      (t,2:2.7)
```
{: .output}

Let's take a closer look at the `gcc` module.
GCC is an extremely widely used C/C++/Fortran compiler.
Tons of software is dependent on the GCC version, 
and might not compile or run if the wrong version is loaded.
In this case, there is GCC/4.9.2
In Awoonga, if it is a default module, it will have (default) marked next to it

if we type `module load GCC/4.9.2`, this is the copy that will be loaded.

```
module load GCC/4.9.2
gcc --version
```
{: .bash}

Lmod is automatically replacing "intel/2016.3" with "GCC/4.9.2".

```

gcc (GCC) 4.9.2
Copyright (C) 2014 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```
{: .output}

Note that three things happened:
the default copy of GCC was loaded (version 4.9.2), 
the Intel compilers (which conflict with GCC) were unloaded,
and software that is dependent on compiler (OpenMPI) was reloaded.
The `module` system turned what might be a super-complex operation into a single command.


> ## Using software modules in scripts
>
> Create a job that is able to run `python3 --version`.
> Remember, no software is loaded by default!
> Running a job is just like logging on to the system 
> (you should not assume a module loaded on the login node is loaded on a worker node).
{: .challenge}

> ## Loading a module by default
> 
> Adding a set of `module load` commands to all of your scripts and
> having to manually load modules every time you log on can be tiresome.
> Fortunately, there is a way of specifying a set of "default modules"
> that always get loaded, regardless of whether or not you're logged on or running a job.
>
> Every user has two hidden files in their home directory: `.bashrc` and `.bash_profile`
> (you can see these files with `ls -la ~`).
> These scripts are run every time you log on or run a job.
> Adding a `module load` command to one of these shell scripts means that
> that module will always be loaded.
> Modify either your `.bashrc` or `.bash_profile` scripts to load a commonly used module like Python.
> Does your `python3 --version` job from before still need `module load` to run?
{: .challenge}

## Installing software of our own

Most HPC clusters have a pretty large set of preinstalled software.
Nonetheless, it's unlikely that all of the software we'll need will be available.
Sooner or later, we'll need to install some software of our own. 

Though software installation differs from package to package,
the general process is the same:
download the software, read the installation instructions (important!),
install dependencies, compile, then start using our software.

As an example we will install the bioinformatics toolkit `seqtk`.
We'll first need to obtain the source code from Github using `git`.

```
git clone https://github.com/lh3/seqtk.git
```
{: .bash}
```
Cloning into 'seqtk'...
remote: Counting objects: 316, done.
remote: Total 316 (delta 0), reused 0 (delta 0), pack-reused 316
Receiving objects: 100% (316/316), 141.52 KiB | 0 bytes/s, done.
Resolving deltas: 100% (181/181), done.
```
{: .output}

Now, using the instructions in the README.md file, 
all we need to do to complete the install is to `cd` 
into the seqtk folder and run the command `make`.

```
cd seqtk
make
```
{: .bash}
```
gcc -g -Wall -O2 -Wno-unused-function seqtk.c -o seqtk -lz -lm
seqtk.c: In function ‘stk_comp’:
seqtk.c:399:16: warning: variable ‘lc’ set but not used [-Wunused-but-set-variable]
    int la, lb, lc, na, nb, nc, cnt[11];
                ^
```
{: .output}

It's done!
Now all we need to do to use the program is invoke it like any other program.

```
./seqtk
```
{: .bash}
```
Usage:   seqtk <command> <arguments>
Version: 1.2-r101-dirty

Command: seq       common transformation of FASTA/Q
         comp      get the nucleotide composition of FASTA/Q
         sample    subsample sequences
         subseq    extract subsequences from FASTA/Q
         fqchk     fastq QC (base/quality summary)
         mergepe   interleave two PE FASTA/Q files
         trimfq    trim FASTQ using the Phred algorithm

         hety      regional heterozygosity
         gc        identify high- or low-GC regions
         mutfa     point mutate FASTA at specified positions
         mergefa   merge two FASTA/Q files
         famask    apply a X-coded FASTA to a source FASTA
         dropse    drop unpaired from interleaved PE FASTA/Q
         rename    rename sequence names
         randbase  choose a random base from hets
         cutN      cut sequence at long N
         listhet   extract the position of each het

```
{: .output}

We've successfully installed our first piece of software!
