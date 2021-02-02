---
title: "Scheduling jobs"
teaching: 45
exercises: 30
questions:
- "What is a scheduler and why are they used?"
- "How do I launch a program to run on any one node in the cluster?"
- "How do I capture the output of a program that is run on a node in the cluster?"
objectives:
- "Run a simple Hello World style program on the cluster."
- "Submit a simple Hello World style script to the cluster."
- "Use the batch system command line tools to monitor the execution of your job."
- "Inspect the output and error files of your jobs."
keypoints:
- "The scheduler handles how compute resources are shared between users."
- "Everything you do should be run through the scheduler."
- "A job is just a shell script."
- "If in doubt, request slightly more resources than you will need."
---

## Job scheduler
An HPC system might have thousands of nodes and thousands of users. How do we decide who gets what
and when? How do we ensure that a task is run with the resources it needs? This job is handled by a
special piece of software called the scheduler. On an HPC system, the scheduler manages which jobs
run where and when.

The following illustration compares these tasks of a job scheduler to a waiter in a restaurant.
If you can relate to an instance where you had to wait for a while in a queue to get in to a 
popular restaurant, then you may now understand why sometimes your jobs do not start instantly
as in your laptop.

{% include figure.html max-width="75%" file="/fig/restaurant_queue_manager.svg"
alt="Compare a job scheduler to a waiter in a restaurant" caption="" %}


> ## Job scheduling roleplay (optional)
> 
> Your instructor will divide you into groups taking on different roles in the cluster (users,
> compute nodes and the scheduler). Follow their instructions as they lead you through this
> exercise. You will be emulating how a job scheduling system works on the cluster.
> 
> [*notes for the instructor here*](../guide)
{: .challenge}

The scheduler used in this lesson is {{ site.sched_name }}. Although {{ site.sched_name }} is not used everywhere,
running jobs is quite similar regardless of what software is being used. The exact syntax might change, but the
concepts remain the same.

## Running a batch job

The most basic use of the scheduler is to run a command non-interactively. Any command (or series 
of commands) that you want to run on the cluster is called a *job*, and the process of using a
scheduler to run the job is called *batch job submission*.

In this case, the job we want to run is just a shell script. Let's create a demo shell script to 
run as a test.

> ## Creating our test job
> 
> Using your favorite text editor, create the following script and run it. Does it run on the
> cluster or just our login node?
>
>```
>#!/bin/bash
>
> echo 'This script is running on:'
> hostname
> sleep 60
> ```
> {: .bash}
{: .challenge}

If you completed the previous challenge successfully, you probably realise that there is a
distinction between running the job through the scheduler and just "running it". To submit this job
to the scheduler, we use the `{{ site.sched_submit }}` command.

```
{{ site.host_prompt }} {{ site.sched_submit }} {{ site.sched_submit_options_long }} example-job.sh
```
{: .bash}
```
{% include /snippets/13/submit_output.snip %}
```
{: .output}

And that's all we need to do to submit a job. Our work is done -- now the scheduler takes over and
tries to run the job for us. While the job is waiting to run, it goes into a list of jobs called 
the *queue*. To check on our job's status, we check the queue using the command
`{{ site.sched_status }} {{ site.sched_flag_user }}`.

```
{{ site.host_prompt }} {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}
```
{% include /snippets/13/statu_output.snip %}
```
{: .output}

We can see all the details of our job, most importantly that it is in the "R" or "RUNNING" state.
Sometimes our jobs might need to wait in a queue ("Q" or "QUEUED") or have an error. The best way to check
our job's status is with `{{ site.sched_status }}`. Of course, running `{{ site.sched_status }}` repeatedly to check on things can be
a little tiresome. To see a real-time view of our jobs, we can use the `watch` command. `watch`
reruns a given command at 2-second intervals. This is too frequent, and will likely upset your system
administrator. You can change the interval to a more reasonable value, for example 20 seconds, with the
`-n 20` parameter. Let's try using it to monitor another job.

```
{{ site.host_prompt }} {{ site.sched_submit }} {{ site.sched_submit_options_long }} example-job.sh
{{ site.host_prompt }} watch -n 20 {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}

You should see an auto-updating display of your job's status. When it finishes, it will disappear
from the queue. Press `Ctrl-C` when you want to stop the `watch` command.

## Customising a job

The job we just ran had a lot of command-line options and used all of the scheduler's default options. In a real-world scenario, that's
probably not what we want. The default options represent a reasonable minimum. Chances are, we will
need more cores and more time. To get access to these
resources we must customize our job script.

Comments in UNIX (denoted by `#`) are typically treated as comments and ignored. But there are exceptions. For instance the
special `#!` comment at the beginning of scripts specifies what program should be used to run it
(typically `/bin/bash`). Schedulers like {{ site.sched_name }} also have a special comment
used to denote special scheduler-specific options. Though these comments differ from scheduler to
scheduler, {{ site.sched_name }}'s special comment is `{{ site.sched_comment }}`.
Anything following the `{{ site.sched_comment }}` comment is
interpreted as an instruction to the scheduler. These options can also
be passed as command-line arguments, but it is often more convenient
to have them hard-wired into the job script.

> ## The clever # trick ...
> The use of `{{ site.sched_comment }}` to denote commands to the
> scheduler is a very neat (and commonly used in Unix) trick which has
> a number of useful effects:
>
> - You can specify commands that are interpreted at submission time
>   (e.g. how much wallclock time you require) and commands that are
>   interpreted much later at run time (e.g. the program you want to
>   execute) in *the same script*.
> - If you are using different HPC machines with different schedulers,
>   you can have commands for both schedulers in the same script and
>   the ones that aren't relevant will simply be ignored as commments
> - You can run the script interactively on the login nodes for
>   debugging purposes (e.g. to check that all the input files are
>   copied across properly) and the scheduler commands will be ignored
>   as comment.
{: .callout}

Let's illustrate this by example. By default, a job's name is the name of the script, but the
`{{ site.sched_flag_name }}` option can be used to change the name of a job.

Submit the following job (`{{ site.sched_submit }} {{ site.sched_submit_options }} example-job.sh`):

```
#!/bin/bash
{{ site.sched_comment }} {{ site.sched_flag_name }} new_name
#PBS -A tc011
#PBS -l walltime=00:10:00
#PBS -l select=1

echo 'This script is running on:'
hostname
sleep 60
```

```
{{ site.host_prompt }} {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}
```
{% include /snippets/13/statu_name_output.snip %}
```
{: .output}

Fantastic, we've successfully changed the name of our job!

> ## Setting up email notifications
> 
> Jobs on an HPC system might run for days or even weeks. We probably have better things to do than
> constantly check on the status of our job with `{{ site.sched_status }}`. Looking at the
> man page for `{{ site.sched_submit }}`, can you set up our test job to send you an email
> when it finishes?
{: .challenge}

### Resource requests

But what about more important changes, such as the number of cores and memory for our jobs? One 
thing that is absolutely critical when working on an HPC system is specifying the resources 
required to run a job. This allows the scheduler to find the right time and place to schedule our 
job. If you do not specify requirements (such as the amount of time you need), you will likely be
stuck with your site's default resources, which is probably not what we want.

The following are several key resource requests:

{% include /snippets/13/stat_options.snip %}

Note that just *requesting* these resources does not make your job run faster! We'll talk more 
about how to make sure that you're using resources effectively in a later episode of this lesson.

> ## Submitting resource requests
>
> Submit a job that will use 2 full nodes and 10 minutes of walltime.
{: .challenge}

{% include /snippets/13/env_challenge.snip %}

Resource requests are typically binding. If you exceed them, your job will be killed. Let's use
walltime as an example. We will request 30 seconds of walltime, and attempt to run a job for one minute.

```
{% include /snippets/13/long_job.snip %}
```

Submit the job and wait for it to finish. Once it is has finished, check the error file.

```
{{ site.host_prompt }} {{ site.sched_submit }} {{ site.sched_submit_options }} example-job.sh
{{ site.host_prompt }} watch -n 20 {{ site.sched_status }} {{ site.sched_flag_user }}
{% include /snippets/13/long_job_cat.snip %}
```
{: .bash}
```
{% include /snippets/13/long_job_err.snip %}
```
{: .output}

Our job was killed for exceeding the amount of resources it requested. Although this appears harsh,
this is actually a feature. Strict adherence to resource requests allows the scheduler to find the
best possible place for your jobs. Even more importantly, it ensures that another user cannot use
more resources than they've been given. If another user messes up and accidentally attempts to use
all of the cores or memory on a node, {{ site.sched_name }} will either restrain their job
to the requested resources or kill the job outright. Other jobs on the node will be unaffected.
This means that one user cannot  mess up the experience of others, the only jobs affected by a
mistake in scheduling will be their
own.

> ## But how much does it cost?
> Although your job will be killed if it exceeds the selected runtime,
> a job that completes within the time limit is only charged for the
> time it actually used. However, you should always try and specify a
> wallclock limit that is close to (but greater than!) the expected
> runtime as this will enable your job to be scheduled more
> quickly.
> If you say your job will run for an hour, the scheduler has
> to wait until a full hour becomes free on the machine. If it only ever
> runs for 5 minutes, you could have set a limit of 10 minutes and it
> might have been run earlier in the gaps between other users' jobs.
{: .callout}

## Cancelling a job

Sometimes we'll make a mistake and need to cancel a job. This can be done with the `{{ site.sched_del }}`
command. Let's submit a job and then cancel it using its job number (remember to change the 
walltime so that it runs long enough for you to cancel it before it is killed!).

```
{{ site.host_prompt }} {{ site.sched_submit }} {{ site.sched_submit_options }} example-job.sh
{{ site.host_prompt }} {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}
```
{% include /snippets/13/del_job_output1.snip %}
```
{: .output}

Now cancel the job with its job number. Absence of any job info indicates that the job has been
successfully cancelled.

```
{% include /snippets/13/del_job.snip %}
... Note that it might take a minute for the job to disappear from the queue ...
{{ site.host_prompt }} {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}
```
{% include /snippets/13/del_job_output2.snip %}
```
{: .output}

{% include /snippets/13/del_multiple_challenge.snip %}

## Other types of jobs

Up to this point, we've focused on running jobs in batch mode. {{ site.sched_name }}
also provides the ability to start an interactive session.

There are very frequently tasks that need to be done interactively. Creating an entire job
script might be overkill, but the amount of resources required is too much for a login node to
handle. A good example of this might be building a genome index for alignment with a tool like
[HISAT2](https://ccb.jhu.edu/software/hisat2/index.shtml). Fortunately, we can run these types of
tasks as a one-off with `{{ site.sched_interactive }}`.

{% include /snippets/13/interactive_example.snip %}

{% include /snippets/13/filesystem_issues.snip %}

## Running parallel jobs using MPI
As we have already seen, the power of HPC systems comes from *parallelism*, i.e. having lots of
processors/disks etc. connected together rather than having more powerful components than your
laptop or workstation. Often, when running research programs on HPC you will need to run a
program that has been built to use the MPI (Message Passing Interface) parallel library. The MPI
library allows programs to exploit multiple processing cores in parallel to allow researchers
to model or simulate faster on larger problem sizes. The details of how MPI work are not important
for this course or even to use programs that have been built using MPI; however, MPI programs 
typically have to be launched in job submission scripts in a different way to serial programs and
users of parallel programs on HPC systems need to know how to do this. Specifically, launching
parallel MPI programs typically requires four things:

  - A special parallel launch program such as `mpirun`, `mpiexec`, `srun` or `aprun`.
  - A specification of how many processes to use in parallel. For example, our parallel program
    may use 256 processes in parallel.
  - A specification of how many parallel processes to use per compute node. For example, if our
    compute nodes each have 32 cores we often want to specify 32 parallel processes per node.
  - The command and arguments for our parallel program.

To illustrate this process, we will use a simple MPI parallel program that sharpens an image.
(We will meet this example program in more detail in a later episode.) Here is a job submission
script that runs the sharpen program across two compute nodes on the cluster. Create a file
called `run-sharpen.pbs` with the contents of this script in it.

{% include /snippets/13/parallel_script.snip %}

The parallel launch line for the sharpen program can be seen towards the bottom of the script:

{% include /snippets/13/parallel_launch_desc.snip %}

As for our other jobs, we launch using the `{{ site.sched_submit }}` command.

```
{{ site.host_prompt }} {{ site.sched_submit }} {{ site.sched_submit_options }} run-sharpen.pbs
```
{: .bash}
```
{% include /snippets/13/submit_output.snip %}
```
{: .output}

If your job runs correctly, you should see an output file called
`sharpened.pgm`

```
{{ site.host_prompt }} ls -l *.pgm
```
{: .bash}
```
-rw-r--r-- 1 yourUsername tc011 1762743 Jun 26 17:29 fuzzy.pgm
-rw------- 1 yourUsername tc011 1678630 Jun 26 17:33 sharpened.pgm
```
{: .output}

If you only see `fuzzy.pgm` and not `sharpened.pgm` then look at
both the output and error files from {{ site.sched_name }} to work out
what went wrong.

{% include /snippets/13/parallel_challenge.snip %}

{% include /snippets/13/parallel_challenge2.snip %}

{% include links.md %}
