# Your First Slurm Job

Learn how to submit and monitor batch jobs using Slurm for automated, longer-running computations.

## Interactive vs Batch Jobs

**Interactive sessions** (`sinteractive`) are great for:
- Real-time data exploration
- Code development and testing
- Tasks requiring immediate feedback

**Batch jobs** (`sbatch`) are better for:
- Long-running computations (hours to days)
- Automated workflows that don't need interaction
- Production runs with large datasets
- Tasks you want to run while you're away

## Set up your workspace

First, create your personal workspace in the CIL project directory:

```bash
# Navigate to the CIL home directories
cd /project/cil/home_dirs/

# Create your personal directory (replace 'youruser' with your username)
mkdir -p youruser
cd youruser

# Create a subfolder for this onboarding work
mkdir rcc_onboarding
cd rcc_onboarding

# Create a subfolder for Slurm job scripts
mkdir slurm_jobs
cd slurm_jobs
```

Your workspace structure should look like:
```
/project/cil/home_dirs/
├── youruser/              # Your personal directory
│   └── rcc_onboarding/ 
│    └── slurm_jobs/ 
```

## Understanding Slurm job scripts

A Slurm job script contains:
1. **Shebang line** - tells the system how to run the script
2. **SBATCH directives** - resource requests and job parameters
3. **Setup commands** - loading modules, activating environments
4. **Your actual work** - the commands you want to run

## Create your first job script

Let's create a simple job that demonstrates basic functionality:

```bash
# In your slurm_jobs directory
nano first_slurm_job.sh
```

Add this content:

```bash
#!/bin/bash

#SBATCH --account=cil
#SBATCH --partition=cil
#SBATCH --job-name=first_job
#SBATCH --output=first_job_%j.out
#SBATCH --error=first_job_%j.err
#SBATCH --time=00:05:00
#SBATCH --cpus-per-task=1
#SBATCH --mem=2G

# Print job information
echo "Job started at: $(date)"
echo "Running on node: $(hostname)"
echo "Job ID: $SLURM_JOB_ID"
echo "Working directory: $(pwd)"

# Load the Python module
module load python

# Print system information
echo "Python version: $(python --version)"
echo "Available CPUs: $(nproc)"
echo "Available memory: $(free -h)"

# Simple computation to show the job is working
echo "Calculating squares of numbers 1-10:"
python -c "
for i in range(1, 11):
    print(f'{i}^2 = {i**2}')
"

echo "Job completed at: $(date)"
```

## Job script breakdown

**Resource directives:**
- `--account=cil` - Use the CIL computing allocation
- `--partition=cil` - Submit to the CIL partition
- `--job-name=first_job` - Name for easier identification
- `--output=first_job_%j.out` - Standard output file (`%j` = job ID)
- `--error=first_job_%j.err` - Error output file
- `--time=00:05:00` - Maximum runtime (5 minutes)
- `--cpus-per-task=1` - Number of CPU cores
- `--mem=2G` - Memory allocation

**File outputs:**
- `%j` gets replaced with the actual job ID
- Separate files help distinguish normal output from errors

## Submit your job

Check cluster availability first:
```bash
bash /project/cil/home_dirs/rcc_onboard/utils/monitor_partition.sh
```

Submit the job:
```bash
sbatch first_slurm_job.sh
```

You'll see output like:
```
Submitted batch job 12345678
```

The job ID (12345678) is important for monitoring.

## Monitor your job

**Check job status:**
```bash
squeue -u $USER
```

Output shows:
```
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
12345678       cil first_jo youruser  R       0:30      1 midway3-0430
```

**Job states:**
- `PD` (Pending) - Waiting for resources
- `R` (Running) - Currently executing
- `CG` (Completing) - Finishing up
- `CD` (Completed) - Finished successfully

**Check detailed job information:**
```bash
scontrol show job 12345678
```

**View job history:**
```bash
sacct -j 12345678
```

## View job output

Once the job completes, check the output files:

```bash
# Standard output
cat first_job_12345678.out

# Error output (should be empty if successful)
cat first_job_12345678.err
```

Expected output in the `.out` file:
```
Job started at: Mon Jul 21 10:30:00 CDT 2025
Running on node: midway3-0430
Job ID: 12345678
Working directory: /project/cil/home_dirs/youruser/rcc_onboarding/slurm_jobs
Python version: Python 3.9.16
Available CPUs: 20
Available memory:              total        used        free
Mem:           503Gi        35Gi       425Gi
Calculating squares of numbers 1-10:
1^2 = 1
2^2 = 4
3^2 = 9
...
Job completed at: Mon Jul 21 10:30:15 CDT 2025
```

## Job management commands

**Cancel a job:**
```bash
scancel 12345678
```

**Cancel all your jobs:**
```bash
scancel -u $USER
```

**View detailed job accounting:**
```bash
sacct -j 12345678 --format=JobID,JobName,Partition,Account,AllocCPUS,State,ExitCode,Start,End,Elapsed
```

## Best practices for job scripts

**Resource estimation:**
- Start conservative, then increase based on actual usage
- Monitor completed jobs with `sacct` to optimize future requests
- Use `seff JOBID` to see efficiency statistics

**File organization:**
- Keep job scripts in a dedicated folder
- Use descriptive job names and output files
- Clean up old output files periodically

**Error handling:**
- Always check both `.out` and `.err` files
- Add error checking in your scripts
- Test with short time limits first

**Time limits:**
- Set realistic time limits (jobs killed if exceeded)
- Add some buffer time for completion
- Break very long jobs into smaller pieces

## Common issues and solutions

**Job stays pending (PD):**
- Check resource availability with the monitoring script
- Verify you have access to the requested partition
- Try requesting fewer resources

**Job fails immediately:**
- Check the `.err` file for error messages
- Verify file paths and permissions
- Test commands interactively first

**Out of memory errors:**
- Increase `--mem` parameter
- Check actual memory usage with `seff JOBID`
- Optimize your code for memory efficiency

**Time limit exceeded:**
- Increase `--time` parameter
- Profile your code to identify bottlenecks
- Consider breaking work into smaller jobs

## Using job arrays for multiple tasks

For running the same job with different parameters:

```bash
#SBATCH --array=1-10

# Use $SLURM_ARRAY_TASK_ID in your script
echo "Processing task number: $SLURM_ARRAY_TASK_ID"
```

## Next steps

Now that you understand batch jobs, you're ready to:

1. Set up [conda environments](../environment-setup/conda-basics.md) for reproducible workflows
2. Work through the [hands-on tutorial](../hands-on-tutorial/) with real climate datasets
3. Learn advanced Slurm features in the [reference section](../reference/slurm-commands.md)

The combination of interactive sessions for development and batch jobs for production forms the foundation of efficient HPC workflows.