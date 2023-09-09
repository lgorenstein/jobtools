# jobscript, jobenv, jobcmd

A collection of tools to look up job scripts, environment variables and
submission command lines for active and historic Slurm jobs.

## TL;DR: Requirements
  * Slurm 21.08 or newer;
  * Set `AccountingStoreFlags = job_comment,job_env,job_script` in `slurm.conf`.
  * Watch your database size to avoid blowing it away by an accidental influx
    of very large user scripts. `SchedulerParameters = max_script_size=###`
    is your friend (defaults to 4 megabytes if not set).

## Gory details

Ability to look at scripts, environments and command lines of past and
active jobs for troubleshooting purposes is a very common need and request
in HPC support land.  However, in Slurm pre-21.08 there was no full support
for these features.  Either they were completely unavailable, or available
for only the currently pending or running jobs (no historic look-up), etc.
Sites used all sorts of hooks and filters, but this was a lot of extra
efforts.  See [Bug 7609](https://bugs.schedmd.com/show_bug.cgi?id=7609) for
some history and external solutions.

Starting from verrsion 21.08, Slurm can store this information in the
accounting database, and make it available via `sacct --batch-script`,
`sacct --env-vars` and `sacct -o SubmitLine` calls for any `-j JobID`
stored in the database.  Again, see
[Bug 7609](https://bugs.schedmd.com/show_bug.cgi?id=7609) for discussion.

Note that these features are not on by default:
sites need to enable a combination of
```
AccountingStoreFlags = job_comment,job_env,job_script
```
in `slurm.conf` in order to partake of corresponding features
(`job_comment` is for command line).  Also note that after you enable them,
you may need to watch your database size a little more closely.  There is
a new `slurm.conf` setting `SchedulerParameters = max_script_size=###`
(in bytes) to control allowed script sizes.  Default value is 4 megabytes.

This is an incredible progress and convenience, but there are caveats:
  1. Some extra flags are needed (especially for command line lookup), and
     who wants to remember and type all of them?
  2. Further massaging of the output may be need in some cases.

Hence this collection of wrappers, because `jobscript NNN` is just so much easier.


## Examples

All scripts try to be smart and handle regular jobs, job steps, arrays and
array elements in a way HPC support staff would typically want them to work.
This solves 99% of support cases, but occasionally you may have to whip
something really heavy like a `sacct -P -o JobID,AllocTRES,SubmitLine -j JobID`
call if you ever need to dig deeper into that remaining 1%.

For more details, see each tool's `--help` text, or read the source, Luke!

### Looking up one job
```bash session
$ jobscript 123
#!/bin/bash

hostname
date
sleep 20
```

### Looking up multiple jobs

Default purist mode for all tools is to use no separator between outputs
for multiple requested jobs.  A `--raw` (`-r`) mode may occasionally be handy:
```bash session
$ jobenv -r 123 456
Environment used for 123 (must be batch to display)
--------------------------------------------------------------------------------
SHELL=/bin/bash
[....]

Environment used for 456 (must be batch to display)
--------------------------------------------------------------------------------
NONE

```
(job 456 was interactive).

### Looking up job submission commands

For command lines, lookups are a little special (generally the output is one
line per job ID, but could be longer).  The `--raw` mode may or may not be
as useful as for other tools (even for multiple jobs).
```bash session
$ jobcmd 123 456 789
sbatch myscript.sub
/usr/bin/salloc -J interactive --bell -N 2
sbatch -a 1-3 array.sub
```

## Author:

Lev Gorenstein, Rosen Center for Advanced Computing, Purdue University, 2021.

Contribute: https://github.com/lgorenstein/jobtools

## More to like:

You might find the `jobinfo` tool from https://github.com/birc-aeh/slurm-utils repository handy for a nice summary of job state.
