# Interactive Sessions

Learn how to start interactive work sessions on RCC using `sinteractive` for real-time computing tasks.

## What are interactive sessions?

Interactive sessions allow you to work directly on compute nodes with allocated resources (CPU, memory, time). Unlike login nodes, which are shared and have limited resources, interactive sessions give you dedicated computational power for tasks like:

- Data exploration and analysis
- Code development and testing
- Running Jupyter notebooks
- Interactive debugging
- Small-scale computations

## Understanding partitions

RCC organizes compute nodes into **partitions** - groups of nodes with similar hardware and access policies. For CIL work, you'll primarily use the `cil` partition, which provides:

- Dedicated nodes for CIL research
- Higher memory availability (up to 512GB per node)
- Priority access for CIL account holders
- Typically less congested than general partitions

## Check partition status

Before requesting resources, check what's available using our cluster monitoring script:

```bash
bash /project/cil/home_dirs/rcc_onboard/utils/monitor_partition.sh
```

This shows real-time information about:

- **Node availability** (idle, mixed, allocated)
- **Resource usage** (CPU and memory utilization)
- **Current job queue** (running and pending jobs)
- **Recommendations** for optimal resource requests

Example output when resources are available:
```
================================================================================
                      HPC Cluster Status - Partition: cil
================================================================================

Node Status:
  ● Idle:        2 nodes [##############################] 100%
  ● Mixed:       0 nodes [------------------------------]   0%
  ● Allocated:   0 nodes [------------------------------]   0%

✓ Excellent availability: 2 idle nodes available
  Recommended: Submit your job now for immediate allocation
```

## Basic interactive session

Start a basic interactive session with standard resources:

```bash
sinteractive --account=cil --partition=cil
```

This requests:
- 1 CPU core
- 4GB memory 
- 2 hours of time
- Access to the `cil` partition

## Requesting specific resources

For more intensive work, specify your resource needs:

```bash
sinteractive --account=cil --partition=cil --cpus-per-task=20 --mem=200G --time=07:00:00
```

**Resource parameters:**
- `--cpus-per-task=20` - Request 20 CPU cores
- `--mem=200G` - Request 200GB of memory
- `--time=07:00:00` - Request 7 hours (format: HH:MM:SS)
- `--account=cil` - Use CIL computing allocation
- `--partition=cil` - Use CIL partition nodes

!!! tip "Resource planning"
    Start with smaller requests while testing, then scale up for production work. You can always start a new session with more resources if needed.

## Requesting specific nodes

If the monitoring script shows specific nodes are idle, you can request them directly:

```bash
sinteractive --nodelist=midway3-0430 --account=cil --partition=cil --cpus-per-task=20 --mem=200G --time=07:00:00
```

This is useful when:
- You need consistent hardware across multiple sessions
- A specific node has the exact resources you need
- You want to avoid potential node differences

## Understanding the allocation process

When you run `sinteractive`, Slurm:

1. **Evaluates your request** against available resources
2. **Finds matching nodes** in the specified partition
3. **Allocates resources** and starts your session
4. **Connects you** to the allocated compute node

You'll see output like:
```
salloc: Granted job allocation 12345678
salloc: Waiting for resource configuration
salloc: Nodes midway3-0430 are ready for job
```

## Working in your session

Once connected, you're on a compute node with your allocated resources:

```bash
# Check your current resources
echo "Node: $(hostname)"
echo "CPUs available: $(nproc)"
echo "Memory available: $(free -h)"

# Load modules for your work
module load python/3.9

# Run your analysis
python my_analysis_script.py
```

## Best practices

**Resource sizing:**
- Request what you actually need - over-requesting wastes resources
- Monitor usage with `htop` or `free -h` during your session
- For memory-intensive work, request 20-30% more than you think you need

**Time management:**
- Interactive sessions have time limits (default 2 hours, max typically 8-12 hours)
- Save your work frequently
- Use `tmux` or `screen` for long-running processes that might outlast your SSH connection

**Partition etiquette:**
- Use the monitoring script to check availability before large requests
- Release sessions when done (exit or Ctrl+D)
- Consider batch jobs for work that doesn't need interaction

## Session management

**Check your current session:**
```bash
squeue -u $USER
```

**Exit your session:**
```bash
exit
# or Ctrl+D
```

**If your session disconnects:**
Your job continues running. Reconnect with:
```bash
ssh midway3  # reconnect to RCC
squeue -u $USER  # find your job ID
# Your session should still be active on the allocated node
```

## Common issues and solutions

**"Resources currently not available":**
- Check the monitoring script for current availability
- Try requesting fewer resources or shorter time
- Consider using a different partition if urgent

**Session killed unexpectedly:**
- Check if you exceeded memory limits with `sacct -j JOBID`
- Review time limits with `scontrol show job JOBID`
- Monitor resource usage during future sessions

**Cannot connect after allocation:**
- Wait a few moments for node setup to complete
- Check SSH connection to RCC itself
- Contact RCC support if persistent

## Next steps

Once you're comfortable with interactive sessions, learn how to submit [batch jobs](first-slurm-job.md) for automated, longer-running computations.