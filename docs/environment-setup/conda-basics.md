# Conda Environment Management

Learn how to create and manage reproducible research environments using conda and mamba on RCC systems.

## Why use conda environments?

Conda environments provide:
- **Reproducible workflows** - exact package versions across sessions
- **Isolation** - different projects can use different package versions
- **Easy sharing** - environment files can be shared with collaborators
- **Clean management** - avoid conflicts between packages

## Set up your environment workspace

Create a dedicated folder for storing your environments:

```bash
# Navigate to your personal directory
cd /project/cil/home_dirs/youruser/envs

# Create a subfolder for environment management
mkdir envs
cd envs
```

Your directory structure should now look like:
```
/project/cil/home_dirs/youruser/rcc_onboarding/
├── slurm_jobs/           # Job scripts from previous section
└── envs/                 # Environment files and management
```

## Load required modules

Always start by loading the Python module on RCC:

```bash
module load python
```

This provides access to conda and mamba (a faster conda alternative).

## Using shared environment templates

CIL provides template environment files for common workflows:

```bash
# Copy shared environment templates to your workspace
cp /project/cil/home_dirs/rcc_onboard/envs/*.yml .

# List available templates
ls -la *.yml
```

You should see:
- `r_geospatial.yml` - R environment for general data mapping/
- `python_geospatial.yml` - Python environment for general data analysis

## Create an R environment for geospatial analysis

Use the provided R template to create an environment for the mortality mapping workflow:

```bash
# Create the R environment using mamba (faster than conda)
mamba env create -f r_geospatial.yml

# This creates an environment named 'r_geospatial'
```

The R environment includes packages for:
- Data manipulation (tidyverse, data.table)
- Geospatial analysis (sf, ggplot2)
- Statistical analysis (Hmisc)
- File handling and utilities

## Create a Python environment for climate data

Create a Python environment:

```bash
# Activate a conda env with mamba
conda activate /project/cil/home_dirs/rcc_onboard/envs/python_base
# Create the Python environment
mamba env create --prefix /project/cil/home_dirs/rcc_onboard/envs/python_geospatial --file /project/cil/home_dirs/rcc_onboard/envs/python_geospatial.yml

# This creates an environment named 'python_geospatial'. 
```

The Python environment includes:
- Data handling (xarray, pandas, numpy)
- Parallel computing (dask)
- Geospatial data (geopandas, rasterio, cartopy)
- File formats (zarr, netcdf4, h5py)
- Visualization (matplotlib, seaborn)

## Activate and use environments

**For R work:**
```bash
# Activate a conda env with mamba
conda activate /project/cil/home_dirs/rcc_onboard/envs/python_base
# Activate the R environment
mamba env create --prefix /project/cil/home_dirs/rcc_onboard/envs/r_geospatial --file /project/cil/home_dirs/rcc_onboard/envs/r_geospatial.yml
# Use it with
source activate /project/cil/home_dirs/rcc_onboard/envs/r_geospatial
# Launch R
R
```

**For Python work:**
```bash
# Activate the Python environment
source activate /project/cil/home_dirs/rcc_onboard/envs/python_geospatial
# Launch Python
python
```

## Creating Jupyter Kernels for Conda Environments

After setting up your conda environments, you can create Jupyter kernels to use them in JupyterLab or Jupyter Notebook:

### For Python Environments

```bash
# Activate your Python environment
source activate /project/cil/home_dirs/rcc_onboard/envs/python_geospatial

# Install the ipykernel package if not already installed
conda install -y ipykernel

# Create a kernel for this environment
python -m ipykernel install --user --name rcc_onboarding_python --display-name "rcc_onboarding_python"
```

### For R Environments

```bash
# Activate your R environment
source activate /project/cil/home_dirs/rcc_onboard/envs/r_geospatial

# Install the IRkernel package if not already installed
R -e "install.packages('IRkernel', repos='https://cloud.r-project.org/')"

# Create a kernel for this environment
R -e "IRkernel::installspec(name = 'rcc_onboarding_R', displayname = 'rcc_onboarding_R')"
```

### Using JupyterLab with Your Kernels

To use JupyterLab with your custom kernels:

```bash
# Activate the base Python environment
source activate /project/cil/home_dirs/rcc_onboard/envs/python_base/

# Launch JupyterLab
jupyter lab
```

Once JupyterLab is running, you can create new notebooks and select your custom kernels:
- "rcc_onboarding_python" for your Python environment
- "rcc_onboarding_R" for your R environment

### Managing Jupyter Kernels

**List available kernels:**
```bash
jupyter kernelspec list
```

**Remove a kernel:**
```bash
jupyter kernelspec uninstall rcc_onboarding_python
```

## Environment management commands

**List your environments:**
```bash
conda env list
```

**Check packages in an environment:**
```bash
conda list -p /project/cil/home_dirs/rcc_onboard/envs/r_geospatial
conda list -p /project/cil/home_dirs/rcc_onboard/envs/python_geospatial
```

**Export an environment:**
```bash
conda activate /project/cil/home_dirs/rcc_onboard/envs/r_geospatial
conda env export --from-history > my_r_environment.yml
```

**Remove an environment:**
```bash
conda env remove -n environment_name
```

## Using environments in Slurm jobs

Integrate conda environments into your job scripts:

```bash
#!/bin/bash
#SBATCH --account=cil
#SBATCH --partition=cil
#SBATCH --job-name=r_analysis
#SBATCH --time=02:00:00
#SBATCH --cpus-per-task=4
#SBATCH --mem=32G

# Load modules
module load python

# Activate environment
conda activate activate /project/cil/home_dirs/rcc_onboard/envs/r_geospatial

# Run your R script
Rscript R_script.R
```

## Best practices

**Environment naming:**
- Use descriptive names (e.g., `mortality_analysis`, `energy_modeling`)
- Include version numbers for long-term projects
- Keep environment names consistent across team members

**Resource management:**
- Store environments in your personal space (not shared directories)
- Clean up unused environments periodically
- Document environment purposes in your project README

**Reproducibility:**
- Always export environment files for sharing
- Pin important package versions in YAML files
- Test environments on fresh installations

## Next steps

With your environments set up, you're ready to:

1. Work through the [hands-on tutorial](../hands-on-tutorial/) using these environments
2. Explore specific [project environments](project-environments.md) for different research areas
3. Learn [troubleshooting](troubleshooting.md) techniques for environment issues

Properly managed environments form the foundation for reproducible climate impact research workflows.