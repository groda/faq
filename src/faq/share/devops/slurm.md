# slurm

`slurm` is the cluster scheduler. You do not run heavy work on the login node. You submit a job, and `slurmd` on a compute node runs it when there is room. The tools are `sbatch`, `srun`, `squeue`, `scancel`, `sinfo`, and `sacct`. They are not GNU, and they are not on a laptop unless someone installed the clients. A typical Linux cluster with Slurm from SchedMD is assumed here. macOS is not a Slurm node. Exit status of `sbatch` is 0 when the job was accepted, not when it finishes. The job's own status comes later, from `sacct` or from `--wait`. A rejected submission is non-zero and no job id exists.

Basic form: sbatch script

`sbatch` reads a script. Directives are lines starting with `#SBATCH` near the top. The rest is the job. Flags on the command line override directives. `srun` is different. It starts a step, often inside a job, or asks for an interactive allocation. Quote paths. A script that is not executable still runs if `sbatch` can read it. The working directory of the job is the directory you submitted from, unless you set `--chdir`.

## Submit a job
Also asked as: sbatch; submit a job; sbatch script; queue a script; slurm submit
`sbatch script` enqueues the script and prints a job id, `Submitted batch job 12345`. The script runs later, on a compute node, as you. It does not run on the login node. `#SBATCH` lines set time, memory, partition, and output files. A flag such as `--time` on the command line wins over the directive. The submission returns immediately. The work has not finished.

```sh
sbatch -- script
```

People run the script with `bash script` and overload the login node. `sbatch` is the submit. People also expect their shell aliases and an interactive module environment to exist. A batch job is non-interactive. It gets a clean environment plus what the site injects. Load modules inside the script. A missing shebang can make the job fail at start. Put `#!/bin/bash` on line 1. The status of `sbatch` is only "accepted."

## Set time, memory, and CPUs
Also asked as: #SBATCH --time; #SBATCH --mem; cpus-per-task; walltime; request resources; -c
`--time=1:00:00` is a one-hour wall clock limit. `--mem=4G` is memory for the job. `--cpus-per-task=4` is CPUs. `--nodes=1` is the usual single node. The job is killed when time runs out. A job that asks for more than the partition allows is rejected, or it sits pending forever. Directives go before any runnable line.

```sh
sbatch --time=1:00:00 --mem=4G -- script
```

People set `--mem` in megabytes by habit and write `--mem=4`, which is four megabytes. Use `4G`. People also request one CPU and then start eight threads. Slurm will not stop you on every site, and the node will thrash. Ask for what you use. Time is wall time, not CPU time. A job that sleeps still consumes the limit. The format is `hours:minutes:seconds`, or `days-hours`. A bare number is minutes on some versions. Be explicit.

## Choose a partition
Also asked as: sbatch -p; #SBATCH --partition; which queue; slurm partition; sinfo partitions
`-p` or `--partition` picks the queue. `sinfo` lists partitions and their nodes. A job with no partition uses the cluster default. A partition you cannot use rejects the job or leaves it pending with a reason. Partition names are site-specific. `debug`, `batch`, and `gpu` are common and not guaranteed.

```sh
sbatch -p pattern -- script
```

People invent a partition name. `sinfo` is the list that exists here. People also leave the partition off and land in a short debug queue with a one-hour cap. Set it. A GPU job usually needs a GPU partition and a `--gres` request. A CPU partition will not hand you a GPU because you asked in the script's prose. The constraint is the directive.

## Name the output files
Also asked as: #SBATCH -o; #SBATCH -e; slurm out; slurm err; %j in output name; job log
`-o` is standard output. `-e` is standard error. `%j` in the name becomes the job id. `%x` becomes the job name. Default is `slurm-%j.out` in the submit directory, with both streams together. The file appears when the job starts, not when you submit. A path on a filesystem the compute node cannot see stays empty.

```sh
sbatch -o file -e file -- script
```

People `tail -f` the output a second after submit and the file is missing. The job has not started. People also write logs to `$HOME` on a site where home is not mounted on compute nodes. Use the scratch or project path your site documents. `-o` truncates on each run unless the name is unique. `%j` makes it unique. A job that fails at prologue may write the reason only to the account record, not to `-o`.

## See the queue
Also asked as: squeue; list jobs; squeue -u; my jobs; what is pending; slurm queue
`squeue` lists jobs. `-u "$USER"` is yours. `-j id` is one job. The state is `PD` pending, `R` running, `CG` completing, `CD` completed in some views. `NODELIST(REASON)` says why a pending job is waiting, such as `Resources` or `Priority`. `squeue` does not show old jobs. That is `sacct`.

```sh
squeue -u "$USER"
```

People run `squeue` with no filter on a busy cluster and miss their line. Filter by user. People also read `PD` as failure. Pending means waiting for resources or for a dependency. The reason column is the explanation. A job you just submitted can take a few seconds to appear. `squeue -j id` on a finished job prints nothing. It has left the queue. Look in `sacct`.

## Why a job is still pending
Also asked as: squeue reason; job pending; Resources; Priority; QOS; why won't my job start
A pending job has a reason. `Resources` means the partition is full. `Priority` means other jobs are ahead. `QOSMax` or a similar reason means your quality-of-service limit is hit. `Dependency` means you asked to wait for another job. `scontrol show job id` prints the full record, including the reason and the requested features.

```sh
scontrol show job -- pattern
```

People cancel and resubmit a `Resources` job. It will wait again. The cluster is busy. People also request a constraint no node has, and the reason stays `QOS` or `Nodes required`. `sinfo -p partition` shows what exists. A typo in `--constraint` or `--gres` looks like a quiet wait. `scontrol show job` is the place to read the request back. Pending is not a stuck scheduler until the reason says the request can never match.

## Cancel a job
Also asked as: scancel; kill a job; cancel job id; scancel -u; stop my jobs
`scancel id` cancels one job. `scancel -u "$USER"` cancels yours. A running job receives a signal, then is killed if it does not exit. A pending job is removed from the queue. `scancel` does not remove the output file. It does not undo files the job already wrote.

```sh
scancel -- pattern
```

People `scancel` and expect the output file to vanish. It remains. People also cancel a job array id and only one task dies, or the whole array dies, depending on whether they passed the array id or a task id. `12345` is the whole array. `12345_3` is one task. A cancel of a finished job is a harmless error. The status of `scancel` is 0 when the request was accepted.

## Wait until a job finishes
Also asked as: sbatch --wait; wait for job; block until job ends; sbatch wait; submit and wait
`sbatch --wait` submits and does not return until the job finishes. The exit status is the job's status, if the site's Slurm is configured to pass it through. This is the simple form in a script that must run the next step after the job. Without `--wait`, `sbatch` returns at submit.

```sh
sbatch --wait -- script
```

People wrap `sbatch` in a loop with `sleep` and never pass `--wait`. `--wait` is the boring flag. It holds the login session. A dropped SSH drops the wait, not the job. For a wait you can leave, submit normally and poll, or use a dependency. `--wait` on a job that stays pending holds the terminal for as long as the queue takes. That is what you asked.

## Poll until a job leaves the queue
Also asked as: wait for squeue; loop until job done; squeue -j until empty; bash wait for slurm; job id variable
When you did not use `--wait`, read the id from `sbatch` and poll `squeue`. An empty `squeue -j id` means the job has left the queue. It may have completed, failed, or been cancelled. `sacct` is how you tell which. The loop is the form people use from a second terminal, or when `--wait` is not available.

```sh
id=$(sbatch --parsable -- script)
while squeue -j "$id" | grep -q "$id"; do
  sleep 30
done
```

`--parsable` prints the bare id, which is safer to capture than the sentence. People grep the whole `squeue` and match another user's similar id. `-j` limits the query. A job that finishes between checks is gone, and the loop ends. Check `sacct -j "$id"` before you treat "left the queue" as success. State `COMPLETED` is success. `FAILED` and `TIMEOUT` and `CANCELLED` are not.

## Chain a job on another job
Also asked as: --dependency; afterok; run after job; sbatch dependency; job chain
`--dependency=afterok:id` starts this job only after that id finished with status 0. `afterany` runs after it ends, whatever the status. `afternotok` runs only if it failed. The dependent job sits pending with reason `Dependency`. You can submit the whole chain at once. You do not have to wait.

```sh
id=$(sbatch --parsable -- script)
sbatch --dependency=afterok:"$id" -- script
```

People write `afterok` and the next job never starts, because the first failed. That is the flag working. Use `afterany` if the second job is a cleanup that must run. People also depend on an array id and the behavior differs between "whole array" and "each task." Pass the id `sbatch` printed. A dependency on a job that was cancelled is a failed dependency. The waiting job is cancelled too on many sites. Read `squeue` for the reason before you assume the scheduler lost it.

## Run one command on a node
Also asked as: srun; srun command; allocate and run; slurm srun; interactive command
`srun command` asks for resources and runs the command on a node. It is the foreground form. It is also how you run a step inside an existing allocation. Flags for resources come before the command. Output comes back to your terminal. A long `srun` holds the session, like `--wait`.

```sh
srun --time=1:00:00 -- command -- file
```

People `srun` a parallel program without `--ntasks` and get one rank. `srun` defaults to one task. Ask for what you need. People also use `srun` in a script that is already a batch job, and nest allocations by mistake. Inside a job, `srun` starts a step on the nodes you already have. On the login node, `srun` requests a new allocation. Those are different. A rejected `srun` prints the reason and exits non-zero. Nothing ran.

## Get an interactive shell on a node
Also asked as: srun --pty bash; interactive job; salloc; shell on a compute node; debug shell
`srun --pty bash -l` is the usual interactive shell on a compute node. `--pty` gives it a terminal. `salloc` is the other form. It reserves resources and leaves you a shell that can `srun` steps. Either one belongs on a debug partition if you have one. An interactive job still has a time limit. When it hits, the shell dies.

```sh
srun --pty --time=1:00:00 bash -l
```

People `ssh` a compute node and skip the scheduler. Many sites forbid that, and the node may be running someone else's job. Use `srun` or `salloc`. People also forget `--pty` and the shell's prompts and job control are wrong. `--pty` is the piece that makes it a terminal. An interactive job is still a job. `squeue` shows it. `scancel` kills it. Modules you loaded on the login node are not automatically in that shell unless the site's startup loads them.

## Ask for a GPU
Also asked as: --gres=gpu; sbatch gpu; GPU partition; request a gpu; cuda job
`--gres=gpu:1` requests one GPU. Many sites also require a GPU partition, `-p gpu`. The type can be named, `gpu:a100:1`, on clusters that have types. A job that asks for a GPU on a CPU partition stays pending or is rejected. Inside the job, the device is visible only if the request was granted. Check with the tool your stack uses.

```sh
sbatch -p pattern --gres=gpu:1 -- script
```

People request a GPU and then run on the login node to "test." The login node has no allocation, and often no GPU. People also request one GPU and start a program that grabs all of them. Ask for the count you will use. A pending GPU job with reason `Resources` means every GPU in that partition is busy. It is not a broken directive. `sinfo -p` the partition shows idle and allocated GPUs if the site exposes them.

## Use a job array
Also asked as: --array; job array; #SBATCH --array; SLURM_ARRAY_TASK_ID; many similar jobs
`--array=0-9` submits one array of ten tasks. Each task is a job with the same script and a different `SLURM_ARRAY_TASK_ID`. That variable is how the script picks its input, such as a row or a shard. `%4` caps how many run at once: `--array=0-99%4`. One `scancel` of the array id cancels the rest. Output files should include `%a` for the task id.

```sh
sbatch --array=0-9 -- script
```

People write one output file and every task overwrites it. Use `%A` for the array id and `%a` for the task. People also loop inside one job instead of an array, and a single task failure kills the loop. An array isolates tasks. A task that overruns memory is killed alone. The others continue. `squeue` shows the array as one line or as tasks, depending on flags. `-r` expands it.

## Load the environment the job needs
Also asked as: module load in sbatch; batch environment; path in job; conda in slurm; job cannot find command
A batch script does not inherit your interactive shell's aliases, functions, or loaded modules, except what the site pushes. Put `module load` in the script. Activate a virtualenv in the script. Use absolute paths if the node image differs from the login node. `echo "$PATH"` in the script is the check. The login node's `which` is not evidence.

```sh
sbatch -- script
```

People load a module, then `sbatch`, and the job says the command is missing. The load died with the login shell. People also `conda activate` in `.bashrc`, and bash as a non-interactive shell does not read `.bashrc` unless you arrange it. Source the env file from the script. A job that works in `srun --pty bash` and fails under `sbatch` is almost always this difference. Compare `env` in both.

## See a finished job
Also asked as: sacct; job history; finished job status; sacct -j; elapsed time; exit code of a job
`sacct -j id` shows the job after it has left the queue. State is `COMPLETED`, `FAILED`, `TIMEOUT`, `CANCELLED`, or `OUT_OF_MEMORY`. `ExitCode` is the process status. `Elapsed` is wall time. `MaxRSS` is the memory high-water mark if accounting is enabled. `squeue` will not show this. The record can take a minute to appear.

```sh
sacct -j pattern --format=JobID,State,ExitCode,Elapsed,MaxRSS
```

People check `squeue` for a job that ended and conclude it was lost. It completed. `sacct` is the history. People also read `COMPLETED` as "the science was right." It means the process exited 0. A script that ignores errors can complete and still have done nothing. `FAILED` with `ExitCode` `1:0` is your script. `TIMEOUT` means it hit `--time`. `OUT_OF_MEMORY` means it hit `--mem` or the node. Raise the limit that matches the state, not both blindly.

## Show nodes and partitions
Also asked as: sinfo; slurm nodes; partition status; idle nodes; sinfo -p
`sinfo` lists partitions and node states. `idle` can take work. `alloc` is full. `down` and `drain` will not. `-p` limits to one partition. `-N` lists nodes. This is the picture of why a job waits on `Resources`. It is not your job list. That is `squeue`.

```sh
sinfo -p pattern
```

People read `idle` and expect their job to start. Higher priority jobs, a reservation, or a feature request can still be ahead. People also pick a partition with no idle nodes and a long MaxTime, when a smaller partition would have run it. `sinfo` shows the time limit of the partition. A request longer than that is rejected or truncated, depending on site policy. Match the request to a partition that can hold it.

## Hold and release a job
Also asked as: scontrol hold; scontrol release; hold a job; do not start yet; release job
`scontrol hold id` keeps a pending job from starting. `scontrol release id` lets it start. A hold is how you submit a chain and arm it later, or stop a job you are about to edit. You cannot edit a running job's script. The copy that will run is already queued. Hold, cancel, and resubmit if the script was wrong.

```sh
scontrol hold -- pattern
```

People edit the script after `sbatch` and expect the queued job to notice. It will not. The job has a copy, or it will read the file at start, depending on site setup. Assume it will not see your edit. Cancel and resubmit. A hold on a running job does not pause it. Hold is for pending. `scancel` is the stop for a running job. Release of a job that can never be satisfied still leaves it pending.

## Pass arguments to the job script
Also asked as: sbatch script args; arguments after script; $1 in a job; sbatch --export; parameters to sbatch
Arguments after the script name become `$1`, `$2` in the job, as with any script. `sbatch script arg` is the form. `--export=ALL` is the default on many sites and passes your environment. `--export=NONE` passes almost nothing, which is more reproducible. `--export=VAR` passes one variable. Do not put secrets in the argument list if the site logs command lines.

```sh
sbatch -- script pattern
```

People put the arguments before the script and `sbatch` reads them as its own flags. The script name ends `sbatch`'s options. People also rely on an exported variable and the site's `sbatch` was set to `--export=NONE`. Pass what you need, or set it inside the script. A job array should take the task id from `SLURM_ARRAY_TASK_ID`, not from a hand-built argument, so each task can still receive the same extra arguments.

## Watch a job's output as it runs
Also asked as: tail slurm.out; follow job output; sbatch output file; tail -f %j; see job print
Once the job is running, the `-o` file grows as the program prints. `tail -f` follows it. Buffering can hide output for a long time if the program block-buffers stdout when it is not a terminal. A batch job's stdout is a file, so it is fully buffered. `stdbuf -oL` or an explicit flush is the fix in the script. The file is in the submit directory unless `-o` said otherwise.

```sh
tail -f -- file
```

People tail a file named with `%j` using the literal percent. The real name has the job id. `squeue` or the submit message has the id. People also tail before the job starts and the file does not exist yet. Wait for `R` in `squeue`, or use a tool that waits. A job that writes only to a file of its own, not to stdout, will leave the slurm output almost empty. That is the program's choice. Look where it said it would write.

## What a failed submit or a failed job means
Also asked as: sbatch exit code; job failed; slurm error; invalid partition; BATCH host; sacct FAILED
`sbatch` exits non-zero when the submission is rejected. A bad partition, a time over the limit, or a missing script are rejections. No id is printed. A submitted job that later fails still had a successful `sbatch`. The failure is in `sacct`, in the output file, and in the email if you asked for one. `sbatch` will not email you unless `--mail-type` is set.

```sh
sbatch -- script
```

People treat a printed job id as success of the work. It is success of the submit. Check `sacct` for `COMPLETED`. A `TIMEOUT` is your `--time`. An `OUT_OF_MEMORY` is your `--mem`. A `FAILED` with a non-zero exit is your script, and the output file usually has the traceback. A job that stays `PD` is not a failure. Read the reason. Under `set -e`, a rejected `sbatch` aborts a driver script. A `--wait` that returns the job's failure also aborts, which is what you want before the next stage reads the output.


## Reserve nodes and get a shell
Also asked as: salloc; allocate nodes; interactive allocation; salloc bash; hold nodes for several commands
`salloc` reserves resources and starts a shell on the login node with the allocation in the environment. From that shell you `srun` steps onto the reserved nodes. It does not, by itself, put your shell on a compute node. `srun --pty bash` does that. `salloc` is the form for several commands against one reservation. The time limit still applies. When it hits, the allocation is revoked.

```sh
salloc --time=1:00:00 --mem=4G
```

People `salloc` and then run the heavy command in that shell, on the login node. The allocation is held and unused. The work belongs under `srun`. People also expect `salloc` to print a job script. It is interactive. A batch script is `sbatch`. A dropped SSH ends the shell and, on most sites, the allocation. `tmux` on the login node is how you keep it across a disconnect. `squeue` shows the allocation. `scancel` of its job id releases the nodes.
