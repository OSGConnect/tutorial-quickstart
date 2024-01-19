---
ospool:
    path: htc_workloads/submitting_workloads/tutorial-quickstart/README.md
---

# Quickstart - Submit Example HTCondor Jobs

## Login to OSG Access Point

To begin, login to your OSG Access Point. 

### Pretyped Setup
To save some typing, you can download the tutorial materials into your home
directory on the access point. This is highly recommended to
ensure that you don't encounter transcription errors during the
tutorials

<pre class="term"><code>$ git clone https://github.com/OSGConnect/tutorial-quickstart</code></pre>

Now, let's start the quickstart tutorial:

<pre class="term"><code>$ cd tutorial-quickstart </code></pre>

### Manual Setup 

Alternatively, if you want the full manual experience, create a new
directory for the tutorial work: 

<pre class="term"><code>$ mkdir tutorial-quickstart
$ cd tutorial-quickstart
</code></pre>
	 

# Example Jobs 

## Job 1: A single discovery job

Inside the tutorial directory that you created or installed previously,
let's create a test script to execute as your job. For pretyped setup, this is
the `short.sh` file: 

<pre class="file"><code>#!/bin/bash
# short.sh: a short discovery job
printf "Start time: "; /bin/date
printf "Job is running on node: "; /bin/hostname
printf "Job running as user: "; /usr/bin/id
printf "Job is running in directory: "; /bin/pwd
echo
echo "Working hard..."
sleep 20
echo "Science complete!"
</code></pre>

Now, make the script executable.

<pre class="term"><code>$ chmod +x short.sh
</code></pre>

### Run the script locally

When setting up a new job submission, it's important to test your job outside
of HTCondor before submitting into the Open Science Pool. 

<pre class="term"><code>$ ./short.sh
Start time: Wed Aug 08 09:21:35 CDT 2023
Job is running on node: ap50.ux.osg-htc.org
Job running as user: uid=54161(alice), gid=5782(osg) groups=5782(osg),5513(osg.login-nodes),7158(osg.OSG-Staff)
Job is running in directory: /home/alice/tutorial-quickstart
Working hard...
Science complete!
</code></pre>

### Create an HTCondor submit file

So far, so good! Let's create a simple (if verbose) HTCondor submit file. This can be found in `tutorial01.submit`.

<pre class="sub"><code># Our executable is the main program or script that we've created
# to do the 'work' of a single job.
executable = short.sh

# We need to name the files that HTCondor should create to save the
#  terminal output (stdout) and error (stderr) created by our job.
#  Similarly, we need to name the log file where HTCondor will save
#  information about job execution steps.
log = short.log
error = short.error
output = short.output

# This is the default category for jobs
+JobDurationCategory = "Medium"

# The below are good base requirements for first testing jobs on OSG, 
#  if you don't have a good idea of memory and disk usage.
request_cpus = 1
request_memory = 1 GB
request_disk = 1 GB

# The last line of a submit file indicates how many jobs of the above
#  description should be queued. We'll start with one job.
queue 1
</code></pre>

### Submit the job 

Submit the job using `condor_submit`:

<pre class="term"><code>$ condor_submit tutorial01.submit
Submitting job(s). 
1 job(s) submitted to cluster 144121.
</code></pre>

### Check the job status

The `condor_q` command tells the status of currently running jobs.

<pre class="term"><code> $ condor_q
-- Schedd: ap50.ux.osg-htc.org : <192.170.227.22:9618?... @ 08/10/23 14:19:08
OWNER	   BATCH_NAME     SUBMITTED   DONE   RUN    IDLE  TOTAL JOB_IDS
alice	 ID: 1441271  08/10 14:18	 _	1      _      1 1441271.0

Total for query: 1 jobs; 0 completed, 0 removed, 0 idle, 1 running, 0 held, 0 suspended
Total for alice: 1 jobs; 0 completed, 0 removed, 0 idle, 1 running, 0 held, 0 suspended
Total for all users: 3001 jobs; 0 completed, 0 removed, 2189 idle, 754 running, 58 held, 0 suspended
</code></pre>

You can also get the status of a specific job cluster: 

<pre class="term"><code>$ condor_q 1441271
-- Schedd: ap50.ux.osg-htc.org : <192.170.227.22:9618?... @ 08/10/23 14:19:08
OWNER	   BATCH_NAME     SUBMITTED   DONE   RUN    IDLE  TOTAL JOB_IDS
alice	 ID: 1441271  08/10 14:18	 _	1      _      1 1441271.0

Total for query: 1 jobs; 0 completed, 0 removed, 0 idle, 1 running, 0 held, 0 suspended
Total for alice: 1 jobs; 0 completed, 0 removed, 0 idle, 1 running, 0 held, 0 suspended
Total for all users: 3001 jobs; 0 completed, 0 removed, 2189 idle, 754 running, 58 held, 0 suspended
</code></pre>	

Note the `DONE`, `RUN`, and `IDLE` columns. Your job will be listed in the `IDLE` column if
it hasn't started yet. If it's currently scheduled and running, it will
appear in the `RUN` column. As it finishes up, it will then show in the `DONE` column.
Once the job completes completely, it will not appear in `condor_q`. 

Let's wait for your job to finish – that is, for `condor_q` not to show
the job in its output. A useful tool for this is `condor_watch_q` – it efficiently
monitors the status of your jobs by monitoring their corresponding log files. 
Let's submit the job again, and use `condor_watch_q` to follow the progress of 
your job (the status will update at two-second intervals): 

<pre class="term"><code>$ condor_submit tutorial01.submit
Submitting job(s). 
1 job(s) submitted to cluster 1441272
$ condor_watch_q 
... 
</code></pre>

*Note*: To close condor_watch_q, hold down `Ctrl` and press C. 

### Check the job output

Once your job has finished, you can look at the files that HTCondor has
returned to the working directory. The names of these files were specified in our
submit file. If everything was successful, it should have returned:

* a log file from HTCondor for the job cluster: `short.log`
	* This file can tell you where a job ran, how long it ran, and what resources it used.
	* If a job shows up as "held" in `condor_q`, this file will have a message 
	that gives a reason why. 
* an output file for each job's output: `short.output`
	* This file can have useful messages that describe how the job progressed. 
* an error file for each job's errors: `short.error`
	* If the job encountered any errors, they will likely be in this file. 

In this case, we will read the output file, which should contain the information printed by our script. It should look something like this: 

<pre class="term"><code>$ cat short.output
Start time: Mon Aug 10 20:18:56 UTC 2023
Job is running on node: osg-84086-0-cmswn2030.fnal.gov
Job running as user: uid=12740(osg) gid=9652(osg) groups=9652(osg)
Job is running in directory: /srv

Working hard...
Science complete!
</code></pre>
	
Job 2: Using Inputs and Arguments in a Job
---------------------------------------

Sometimes it's useful to pass arguments to your executable from your
submit file. For example, you might want to use the same job script
for more than one run, varying only the parameters. You can do that
by adding Arguments to your submission file.

First, let's edit our existing `short.sh` script to accept arguments. To avoid 
losing our original script, we make a copy of the file under the name `short_transfer.sh`
**if you haven't already downloaded this entire tutorial**. 

<pre class="term"><code>$ cp short.sh short_transfer.sh</code></pre>

Now, edit the file to include the added lines below or use `cat` to view the 
existing `short_transfer.sh` file: 

<pre class="file"><code>#!/bin/bash
# short_transfer.sh: a short discovery job
printf "Start time: "; /bin/date
printf "Job is running on node: "; /bin/hostname
printf "Job running as user: "; /usr/bin/id
printf "Job is running in directory: "; /bin/pwd
printf "The command line argument is: "; echo $@
printf "Job number is: "; echo $2
printf "Contents of $1 is "; cat $1
cat $1 > output$2.txt
echo
echo "Working hard..."
ls -l $PWD
sleep 20
echo "Science complete!"
</code></pre>

We need to make our new script executable just as we did before:

<pre class="term"><code>$ chmod +x short_transfer.sh</code></pre>

Notice that with our changes, the new script will now print out the contents of whatever file we specify in our arguments, specified by the `$1`. It will also copy the contents of that file into another file called `output.txt`.

Make a simple text file called `input.txt` that we can pass to our script:

<pre class="file"><code>"Hello World"</code></pre>

Once again, before submitting our job we should test it locally to ensure it runs as we expect:

<pre class="term"><code>$ ./short_transfer.sh input.txt
Start time: Tue Dec 11 10:19:12 CST 2018
Job is running on node: ap50.ux.osg-htc.org
Job running as user: uid=54161(alice), gid=5782(osg) groups=5782(osg),5513(osg.login-nodes),7158(osg.OSG-Staff)
Job is running in directory: /home/alice/tutorial-quickstart
The command line argument is: input.txt
Job number is: 
Contents of input.txt is "Hello World"
Working hard
total 28
drwxrwxr-x 2 alice users   34 Aug 15 09:37 Images
-rw-rw-r-- 1 alice users   13 Aug 15 09:37 input.txt
drwxrwxr-x 2 alice users  114 Aug 11 09:50 log
-rw-r--r-- 1 alice users   13 Aug 11 10:19 output.txt
-rwxrwxr-x 1 alice users  291 Aug 15 09:37 short.sh
-rwxrwxr-x 1 alice users  390 Aug 11 10:18 short_transfer.sh
-rw-rw-r-- 1 alice users  806 Aug 15 09:37 tutorial01.submit
-rw-rw-r-- 1 alice users  547 Aug 11 09:49 tutorial02.submit
-rw-rw-r-- 1 alice users 1321 Aug 15 09:37 tutorial03.submit
Science complete!
</code></pre>

Now, let's edit our submit file to properly handle these new arguments and output files and save this as `tutorial02.submit`

<pre class="sub"><code># We need the job to run our executable script, with the
#  input.txt filename as an argument, and to transfer the
#  relevant input file.
executable = short_transfer.sh
arguments = input.txt

transfer_input_files = input.txt
# output files will be transferred back automatically 

log = job.log
error = job.error
output = job.output

+JobDurationCategory = "Medium"

request_cpus = 1
request_memory = 1 GB
request_disk = 1 GB

# Queue one job with the above specifications.
queue 1
</code></pre>

Notice the added `arguments = input.txt` information. The `arguments` option specifies what arguments should be passed to the executable. 

The `transfer_input_files` options needs to be included as well.  When jobs are executed on the Open Science Pool via HTCondor, they are sent only with files that are specified.
Any new files generated by the job in the working directory will be returned to the 
Access Point. 

Submit the new submit file using `condor_submit`. Be sure to check your output files once the job completes.

<pre class="term"><code>$ condor_submit tutorial02.submit
Submitting job(s).
1 job(s) submitted to cluster 1444781.
</code></pre>

Run the commands from the previous section to check on the job in the queue, and 
view the outputs when the job completes. 

Job 3: Submitting Multiple Jobs at Once
-----------------------------------

What do we need to do to submit several jobs simultaneously? In the
first example, Condor returned three files: out, error, and log. If we
want to submit several jobs, we need to track these three files for each
job. An easy way to do this is to add the `$(Cluster)` and `$(Process)`
macros to the HTCondor submit file. Since this can make our working
directory really messy with a large number of jobs, let's tell HTCondor
to put the files in a directory called `log`. 

We will also include the `$(Process)` value as a second argument to our 
script, which will cause it to give our output files unique names. If you want to 
try it out, you can do so like this: 

<pre class="term"><code>$ ./short_transfer.sh input.txt 12</code></pre>

Incorporating all these ideas, 
here's what the third submit file looks like, called `tutorial03.submit`:

<pre class="sub"><code># For this example, we'll specify unique filenames for each job by using
#  the job's 'Process' value.
executable = short_transfer.sh
arguments = input.txt $(Process)

transfer_input_files = input.txt

log = log/job.$(Cluster).log
error = log/job.$(Cluster).$(Process).error
output = log/job.$(Cluster).$(Process).output

+JobDurationCategory = "Medium"

request_cpus = 1
request_memory = 1 GB
request_disk = 1 GB

# Let's queue ten jobs with the above specifications
queue 10
</code></pre>

Before submitting, we also need to make sure the log directory exists.

<pre class="term"><code>$ mkdir -p log</code></pre>

You'll see something like the following upon submission:

<pre class="term"><code>$ condor_submit tutorial03.submit
Submitting job(s)..........
10 job(s) submitted to cluster 1444786.</code></pre>
	
Look at the output files in the log directory and notice how each job received its own separate output file:

<pre class="term"><code>$ ls log
job.1444786.0.error    job.1444786.3.error    job.1444786.6.error    job.1444786.9.error
job.1444786.0.output  job.1444786.3.output  job.1444786.6.output  job.1444786.9.output
job.1444786.2.error    job.1444786.4.error    job.1444786.7.error    job.1444786.log
job.1444786.1.output  job.1444786.4.output  job.1444786.7.output
job.1444786.2.error    job.1444786.5.error    job.1444786.8.error
job.1444786.2.output  job.1444786.5.output  job.1444786.8.output
</code></pre>


## Removing Jobs

On occasion, jobs will need to be removed for a variety of reasons
(incorrect parameters, errors in submission, etc.). In these instances,
the `condor_rm` command can be used to remove an entire job submission
or just particular jobs in a submission. The `condor_rm` command accepts
a cluster id, a job id, or username and will remove an entire cluster
of jobs, a single job, or all the jobs belonging to a given user
respectively. E.g. if a job submission generates 100 jobs and is
assigned a cluster id of 103, then `condor_rm 103.0` will remove the
first job in the cluster. Likewise, `condor_rm 103` will remove all
the jobs in the job submission and `condor_rm [username]` will remove
all jobs belonging to the user. The `condor_rm` documenation has more
details on using `condor_rm` including ways to remove jobs based on other
constraints.

## Getting Your Work Running

Now that you have some practice with running HTCondor jobs, consider 
reviewing our [Getting Started 
Roadmap](https://portal.osg-htc.org/documentation/htc_workloads/workload_planning/roadmap/)
to see what next steps will get your own computational work running on the OSPool. 
