---
title: "Using resources effectively"
teaching: 5
exercises: 20
questions:
- "How do we monitor our jobs?"
- "How can I get my jobs scheduled more easily?" 
objectives:
- "Understand how to look up job statistics and profile code."
- "Understand job size implications."
keypoints:
- "The smaller your job, the faster it will schedule."
---

We now know virtually everything we need to know about getting stuff onto and using a cluster.
We can log on, submit different types of jobs, use preinstalled software, and install and use
software of our own. What we need to do now is use the systems effectively. To do this we need
to understand the basics of *benchmarking*. Benchmarking is essentially performing simple 
experiments to help understand how the performance of our work varies as we change the
properties of the jobs on the cluster - including input parameters, job options and resources used.

> ## Our example
> In the rest of this episode, we will use an example parallel application that sharpens
> an input image. Although this is a toy problem, it exhibits all the properties of a full
> parallel application that we are interested in for this course.
{: .callout}

## Accessing the software and input

The `sharpen` application has been preinstalled on Cirrus, you can access it with the 
command:

```
module load epcc-training/sharpen
```

Once you have loaded the module, you can access the program as `sharpen`. You will also
need to get a copy of the input file for this example. To do this, copy it from the 
central install location to your directory with (note you must have loaded the 
sharpen module as described above for this to work):

```
{{ site.host_prompt }} cp $TRAINING_SHARPEN/fuzzy.pgm .
```
{: .language-bash}

## Baseline: running in serial

Before starting to benchmark an application to understand what resources are best to use to
run it you need a *baseline* performance result. In more formal benchmarking, your baseline
is usually the minimum number of cores or nodes you can run on. However, for understanding
how best to use resources, as we are doing here, your baseline could be the performance on
any number of cores or nodes that you can measure the change in performance from.

Our `sharpen` application is small enough that we can run a serial (i.e. using a single core)
job for our baseline performance so that is where we will start

> ## Run a single core job
> Write a job submission script that runs the `sharpen` application on a single core. You
> will need to take an initial guess as to the walltime to request to give the job time 
> to complete. Submit the job and check the contents of the STDOUT file to see if the 
> application worked or not.
{: .challenge}

Once your job has run, you should look in the output to identify the performance. Most 
HPC programs should print out timing or performance information (usually somewhere near
the bottom of the summary output) and `sharpen` is no exception. You should see two 
lines in the output that look something like:

```
Calculation time was 5.579000 seconds
Overall run time was 5.671895 seconds
```

You can also get an estimate of the overall run time from the final job statistics. If
we look at how long the finished job ran for, this will provide a quick way to see
roughly what the runtime was. This can be useful if you want to know quickly if a 
job was faster or not than a previous job (as oyu do not have to find the output file
to look up the performance) but the number is not as accurate as the performance recorded
by the application itself and also includes static overheads from running the job
(such as loading modules and startup time) that can skew the timings. To do this on
Cirrus use `qstat` with the `-x` option and the job ID:

```
{{ site.host_prompt }} {{ site.sched_hist }} 12345
```
{: .language-bash}
```
{% include /snippets/16/stat_output.snip %}
```
{: .output}

> ## Viewing the sharpened output image
> To see the effect of the sharpening algorithm, you can view the images using the display
> program from the ImageMagick suite.
> ```
> display sharpened.pgm
> ```
> Type `q` in the image window to close the program. To view the image you will need an X
> window client installed and you will have to have logged into Cirrus with the `ssh -Y`
> option to export the display back to your local system. If you are using Windows, the 
> MobaXterm program provides a login shell with X capability. If you are using macOS, then
> you will need to install XQuartz. If you are using Linux then X should just work!
{: .callout}

## Running in parallel and benchmarking performance

We have now managed to run the `sharpen` application using a single core and have a baseline
performance we can use to judge how well we are using resources on the system.

Note that we also now have a good estimate of how long the application takes to run so we can
provide a better setting for the walltime for future jobs we submit. Lets now look at how
the runtime varies with core count.

> ## Benchmarking the parallel performance
> Modify your job script to run on multiple cores and evaluate the performance of `sharpen`
> on a variety of different core counts and use multiple runs to complete the table below.
>
> If you examine the log file you will see that it contains two timings: the total time taken by the
> entire program (including IO) and the time taken solely by the calculation. The image input
> and output is not parallelised so this is a serial overhead, performed by a single processor.
> The calculation part is, in theory, perfectly parallel (each processor operates on different parts
> of the image) so this should get faster on more cores. The IO time in the table below is the
> difference between the calculation time and the overall run time; the total core seconds is the
> *calculation time* multiplied by the number of cores.
> 
> | Cores      | Overall run time (s) | Calculation time (s) | IO time (s) | Total core seconds |
> |------------|----------------------|----------------------|-------------|--------------------|
> | 1 (serial) |                      |                      |             |                    |
> | 2          |                      |                      |             |                    |
> | 4          |                      |                      |             |                    |                    
> | 8          |                      |                      |             |                    | 
> | 18         |                      |                      |             |                    | 
> | 36         |                      |                      |             |                    | 
> | 72         |                      |                      |             |                    |              
> | 144        |                      |                      |             |                    | 
>
> Look at your results – do they make sense? Given the structure of the code, you would
> expect the IO time to be roughly constant, and the performance of the calculation to increase
> linearly with the number of cores: this would give a roughly constant figure for the total CPU
> time. Is this what you observe?
>
> - What do you think a good core count choice would be for this application to use for runs on
>   Cirrus which would use resources efficiently?
> - What is the core count where you get the **most** efficient use of resources, irrespective
>   of run time?
>
{: .challenge}




{% include links.md %}
