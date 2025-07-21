# Explore Mortality Data

Learn how to explore multidimensional climate impact datasets using Python and xarray. This tutorial uses real mortality projection data from CIL's research outputs.

## Dataset overview

We'll work with mortality projections from the CIL research pipeline. The dataset contains:

- **Temporal coverage**: 120 years of projections (typically 2000-2119)
- **Spatial coverage**: 5,716 regions globally
- **Variables**: Multiple mortality impact measures
- **Format**: NetCDF4 with hierarchical data structure

**Dataset location**: `/project/cil/home_dirs/rcc_onboard/data/mortality/montecarlo/`

**File**: `Agespec_interaction_response-combined-incadapt-aggregated.nc4`

## Key variables in the dataset

- `rebased` - Mortality impacts relative to 2005 baseline (deaths/person/year)
- `response` - Direct marginal mortality response (deaths per 100,000)
- `transformed` - Raw mortality rate conversions
- `year` - Impact years (2000-2119)
- `regions` - Hierarchical region identifiers

We'll focus on the `rebased` variable, which shows climate-driven mortality changes relative to a 2005 baseline.

## Set up your workspace

Navigate to your tutorial workspace and copy the analysis script:

```bash
cd /project/cil/home_dirs/youruser/rcc_onboarding

# Create a subfolder for the tutorial
mkdir hands_on_tutorial
cd hands_on_tutorial

# Copy the analysis script from the shared resources
cp /project/cil/home_dirs/rcc_onboard/scripts/explore_mortality_data.py .
```

## Option 1: Exploratory analysis via Slurm job

Create a batch job to analyze the mortality data and generate summary statistics and plots.

### Create the analysis script

The analysis script is available in the shared resources:

```bash
# View the script (it's already created in the shared location)
less /project/cil/home_dirs/rcc_onboard/scripts/explore_mortality_data.py
```

The script contains the following analysis workflow:

```python
#!/usr/bin/env python3
"""
Mortality Data Exploration Script
Analyzes climate-driven mortality projections from CIL research outputs
"""

import xarray as xr
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
from pathlib import Path
import sys

def load_mortality_data():
    """Load the mortality dataset from CIL research outputs."""
    # Dataset path - using the shared onboarding data copy
    data_path = "/project/cil/home_dirs/rcc_onboard/data/mortality/montecarlo/batch0/rcp45/ACCESS1-0/high/SSP3/Agespec_interaction_response-combined-incadapt-aggregated.nc4"
    
    print(f"Loading dataset: {data_path}")
    
    try:
        ds = xr.open_dataset(data_path)
        print("✓ Dataset loaded successfully")
        return ds
    except Exception as e:
        print(f"✗ Error loading dataset: {e}")
        sys.exit(1)

def explore_dataset_structure(ds):
    """Examine the basic structure of the dataset."""
    print("\n" + "="*60)
    print("DATASET STRUCTURE")
    print("="*60)
    
    print(f"Dimensions: {dict(ds.dims)}")
    print(f"Variables: {list(ds.data_vars)}")
    print(f"Coordinates: {list(ds.coords)}")
    
    # Focus on the rebased variable
    rebased = ds['rebased']
    print(f"\nRebased variable info:")
    print(f"  Shape: {rebased.shape}")
    print(f"  Units: {rebased.attrs.get('units', 'Not specified')}")
    print(f"  Description: {rebased.attrs.get('long_title', 'Not specified')}")
    
    return rebased

def analyze_temporal_trends(rebased):
    """Analyze how mortality impacts change over time."""
    print("\n" + "="*60)
    print("TEMPORAL ANALYSIS")
    print("="*60)
    
    # Calculate global mean mortality impact by year
    global_mean = rebased.mean(dim='region')
    
    # Get years for analysis
    years = rebased.year.values
    
    print(f"Year range: {years.min()} to {years.max()}")
    print(f"Global mean mortality impact in {years[0]}: {global_mean.values[0]:.6f} deaths/person/year")
    print(f"Global mean mortality impact in {years[-1]}: {global_mean.values[-1]:.6f} deaths/person/year")
    
    # Calculate decade averages
    decades = []
    decade_means = []
    
    for decade_start in range(int(years[0]), int(years[-1]), 10):
        decade_end = decade_start + 9
        decade_mask = (years >= decade_start) & (years <= decade_end)
        if decade_mask.any():
            decade_mean = global_mean[decade_mask].mean().values
            decades.append(f"{decade_start}s")
            decade_means.append(decade_mean)
            print(f"  {decade_start}s average: {decade_mean:.6f} deaths/person/year")
    
    return global_mean, decades, decade_means

def analyze_spatial_patterns(rebased):
    """Analyze spatial distribution of mortality impacts."""
    print("\n" + "="*60)
    print("SPATIAL ANALYSIS")
    print("="*60)
    
    # Focus on mid-century (around 2050)
    mid_century_year = 2050
    if mid_century_year in rebased.year.values:
        mid_century_data = rebased.sel(year=mid_century_year)
    else:
        # Find closest year
        closest_year = rebased.year.values[np.argmin(np.abs(rebased.year.values - mid_century_year))]
        mid_century_data = rebased.sel(year=closest_year)
        print(f"Using year {closest_year} (closest to {mid_century_year})")
    
    # Calculate statistics
    mean_impact = mid_century_data.mean().values
    std_impact = mid_century_data.std().values
    min_impact = mid_century_data.min().values
    max_impact = mid_century_data.max().values
    
    print(f"Mortality impacts for year {mid_century_data.year.values}:")
    print(f"  Mean: {mean_impact:.6f} deaths/person/year")
    print(f"  Std Dev: {std_impact:.6f}")
    print(f"  Range: {min_impact:.6f} to {max_impact:.6f}")
    
    # Find regions with highest and lowest impacts
    max_region_idx = mid_century_data.argmax().values
    min_region_idx = mid_century_data.argmin().values
    
    regions = rebased.regions.values
    print(f"  Highest impact region: {regions[max_region_idx]} ({max_impact:.6f})")
    print(f"  Lowest impact region: {regions[min_region_idx]} ({min_impact:.6f})")
    
    return mid_century_data

def create_visualizations(rebased, global_mean, mid_century_data):
    """Create summary plots of the mortality data."""
    print("\n" + "="*60)
    print("CREATING VISUALIZATIONS")
    print("="*60)
    
    # Create output directory if it doesn't exist
    output_dir = "/project/cil/home_dirs/rcc_onboard/outputs/mortality"
    Path(output_dir).mkdir(parents=True, exist_ok=True)
    
    # Set up the plotting
    fig, axes = plt.subplots(2, 2, figsize=(15, 12))
    fig.suptitle('Climate-Driven Mortality Projections Analysis', fontsize=16, fontweight='bold')
    
    # Plot 1: Time series of global mean
    ax1 = axes[0, 0]
    years = rebased.year.values
    ax1.plot(years, global_mean.values, linewidth=2, color='darkred')
    ax1.set_title('Global Mean Mortality Impact Over Time')
    ax1.set_xlabel('Year')
    ax1.set_ylabel('Deaths/Person/Year')
    ax1.grid(True, alpha=0.3)
    ax1.ticklabel_format(style='scientific', axis='y', scilimits=(0,0))
    
    # Plot 2: Histogram of spatial distribution at mid-century
    ax2 = axes[0, 1]
    # Remove extreme outliers for better visualization
    data_clean = mid_century_data.values
    q1, q99 = np.percentile(data_clean[~np.isnan(data_clean)], [1, 99])
    data_clean = data_clean[(data_clean >= q1) & (data_clean <= q99)]
    
    ax2.hist(data_clean, bins=50, alpha=0.7, color='darkblue', edgecolor='black', linewidth=0.5)
    ax2.set_title(f'Distribution of Regional Impacts ({mid_century_data.year.values})')
    ax2.set_xlabel('Deaths/Person/Year')
    ax2.set_ylabel('Number of Regions')
    ax2.ticklabel_format(style='scientific', axis='x', scilimits=(0,0))
    
    # Plot 3: Decadal trends
    ax3 = axes[1, 0]
    decade_starts = range(int(years[0]), int(years[-1]), 10)
    decade_means = []
    decade_stds = []
    
    for decade_start in decade_starts:
        decade_end = decade_start + 9
        decade_mask = (years >= decade_start) & (years <= decade_end)
        if decade_mask.any():
            decade_data = global_mean[decade_mask]
            decade_means.append(decade_data.mean().values)
            decade_stds.append(decade_data.std().values)
    
    decade_labels = [f"{d}s" for d in decade_starts[:len(decade_means)]]
    
    ax3.errorbar(range(len(decade_means)), decade_means, yerr=decade_stds, 
                marker='o', linewidth=2, markersize=8, capsize=5, color='darkgreen')
    ax3.set_title('Decadal Average Mortality Impacts')
    ax3.set_xlabel('Decade')
    ax3.set_ylabel('Deaths/Person/Year')
    ax3.set_xticks(range(len(decade_labels)))
    ax3.set_xticklabels(decade_labels, rotation=45)
    ax3.grid(True, alpha=0.3)
    ax3.ticklabel_format(style='scientific', axis='y', scilimits=(0,0))
    
    # Plot 4: Regional impact ranking (top 20 highest impacts)
    ax4 = axes[1, 1]
    sorted_impacts = mid_century_data.sortby(mid_century_data, ascending=False)
    top_20 = sorted_impacts[:20]
    regions_top20 = rebased.regions.values[top_20.argsort()[::-1][:20]]
    
    y_pos = range(20)
    ax4.barh(y_pos, top_20.values[:20], color='darkred', alpha=0.7)
    ax4.set_title(f'Top 20 Regions by Mortality Impact ({mid_century_data.year.values})')
    ax4.set_xlabel('Deaths/Person/Year')
    ax4.set_ylabel('Region Rank')
    ax4.ticklabel_format(style='scientific', axis='x', scilimits=(0,0))
    
    # Adjust layout and save
    plt.tight_layout()
    
    output_file = f"{output_dir}/mortality_analysis_summary.png"
    plt.savefig(output_file, dpi=300, bbox_inches='tight')
    print(f"✓ Saved visualization: {output_file}")
    
    plt.close()

def save_summary_statistics(rebased, global_mean):
    """Save key statistics to CSV files."""
    print("\n" + "="*60)
    print("SAVING SUMMARY DATA")
    print("="*60)
    
    # Create output directory if it doesn't exist
    output_dir = "/project/cil/home_dirs/rcc_onboard/outputs/mortality"
    Path(output_dir).mkdir(parents=True, exist_ok=True)
    
    # Save time series data
    years = rebased.year.values
    time_series_df = pd.DataFrame({
        'year': years,
        'global_mean_mortality': global_mean.values
    })
    
    time_series_file = f"{output_dir}/mortality_time_series.csv"
    time_series_df.to_csv(time_series_file, index=False)
    print(f"✓ Saved time series: {time_series_file}")
    
    # Save regional summary for mid-century
    mid_century_year = 2050
    if mid_century_year in rebased.year.values:
        mid_century_data = rebased.sel(year=mid_century_year)
    else:
        closest_year = rebased.year.values[np.argmin(np.abs(rebased.year.values - mid_century_year))]
        mid_century_data = rebased.sel(year=closest_year)
    
    regions_df = pd.DataFrame({
        'region': rebased.regions.values,
        'mortality_impact_2050': mid_century_data.values
    })
    
    regions_file = f"{output_dir}/mortality_regional_2050.csv"
    regions_df.to_csv(regions_file, index=False)
    print(f"✓ Saved regional data: {regions_file}")

def main():
    """Main analysis workflow."""
    print("Starting Mortality Data Exploration")
    print("="*60)
    
    # Load and explore data
    ds = load_mortality_data()
    rebased = explore_dataset_structure(ds)
    
    # Perform analyses
    global_mean, decades, decade_means = analyze_temporal_trends(rebased)
    mid_century_data = analyze_spatial_patterns(rebased)
    
    # Create visualizations
    create_visualizations(rebased, global_mean, mid_century_data)
    
    # Save summary data
    save_summary_statistics(rebased, global_mean)
    
    print("\n" + "="*60)
    print("✓ ANALYSIS COMPLETE")
    print("="*60)
    print("Generated files in /project/cil/home_dirs/rcc_onboard/outputs/mortality/:")
    print("  - mortality_analysis_summary.png")
    print("  - mortality_time_series.csv") 
    print("  - mortality_regional_2050.csv")

if __name__ == "__main__":
    main()
```

### Create the Slurm job script

Copy and customize the job script from shared resources:

```bash
# Copy the job script template
cp /project/cil/home_dirs/rcc_onboard/slurm_jobs/run_mortality_analysis.sh .

# Or create your own version
nano run_mortality_analysis.sh
```

The job script content:

```bash
#!/bin/bash

#SBATCH --account=cil
#SBATCH --partition=cil
#SBATCH --job-name=mortality_analysis
#SBATCH --output=mortality_analysis_%j.out
#SBATCH --error=mortality_analysis_%j.err
#SBATCH --time=01:00:00
#SBATCH --cpus-per-task=4
#SBATCH --mem=32G

# Load modules
module load python

# Activate the Python environment
source activate /project/cil/home_dirs/rcc_onboard/envs/python_geospatial

# Run the analysis script from the shared location
echo "Starting mortality data analysis at: $(date)"
python /project/cil/home_dirs/rcc_onboard/scripts/explore_mortality_data.py
echo "Analysis completed at: $(date)"

# List generated files in the output directory
echo "Generated output files:"
ls -la /project/cil/home_dirs/rcc_onboard/outputs/mortality/
```

### Submit and monitor the job

```bash
# Check cluster availability
bash /project/cil/home_dirs/rcc_onboard/utils/monitor_partition.sh

# Submit the job
sbatch run_mortality_analysis.sh

# Monitor progress
squeue -u $USER
```

## Option 2: Interactive exploration with Jupyter

For interactive data exploration, use a Jupyter notebook with these key chunks:

### Setup and data loading

```python
import xarray as xr
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd

# Load the dataset
data_path = "/project/cil/home_dirs/rcc_onboard/data/mortality/montecarlo/batch0/rcp45/ACCESS1-0/high/SSP3/Agespec_interaction_response-combined-incadapt-aggregated.nc4"
ds = xr.open_dataset(data_path)

# Examine the structure
print("Dataset dimensions:", dict(ds.dims))
print("Variables:", list(ds.data_vars))

# Focus on rebased mortality impacts
rebased = ds['rebased']
print(f"\nRebased variable shape: {rebased.shape}")
print(f"Units: {rebased.attrs.get('units')}")
print(f"Description: {rebased.attrs.get('long_title')}")
```

### Quick data exploration

```python
# Basic statistics
print("Global statistics across all years and regions:")
print(f"Mean: {rebased.mean().values:.8f}")
print(f"Standard deviation: {rebased.std().values:.8f}")
print(f"Min: {rebased.min().values:.8f}")
print(f"Max: {rebased.max().values:.8f}")

# Check for missing values
print(f"Missing values: {rebased.isnull().sum().values}")
```

### Temporal trends visualization

```python
# Calculate global mean by year
global_mean = rebased.mean(dim='region')

# Plot time series
plt.figure(figsize=(12, 6))
plt.plot(rebased.year, global_mean, linewidth=2)
plt.title('Global Mean Mortality Impact Over Time')
plt.xlabel('Year')
plt.ylabel('Deaths/Person/Year')
plt.grid(True, alpha=0.3)
plt.ticklabel_format(style='scientific', axis='y', scilimits=(0,0))
plt.show()
```

### Spatial analysis for specific year

```python
# Select data for 2050 (or closest year)
target_year = 2050
if target_year in rebased.year.values:
    year_data = rebased.sel(year=target_year)
else:
    closest_year = rebased.year.values[np.argmin(np.abs(rebased.year.values - target_year))]
    year_data = rebased.sel(year=closest_year)
    print(f"Using year {closest_year} (closest to {target_year})")

# Create histogram of regional impacts
plt.figure(figsize=(10, 6))
# Remove extreme outliers for visualization
data_clean = year_data.values
q1, q99 = np.percentile(data_clean[~np.isnan(data_clean)], [1, 99])
data_clean = data_clean[(data_clean >= q1) & (data_clean <= q99)]

plt.hist(data_clean, bins=50, alpha=0.7, edgecolor='black')
plt.title(f'Distribution of Regional Mortality Impacts ({year_data.year.values})')
plt.xlabel('Deaths/Person/Year')
plt.ylabel('Number of Regions')
plt.ticklabel_format(style='scientific', axis='x', scilimits=(0,0))
plt.show()

# Find extreme regions
max_region_idx = year_data.argmax().values
min_region_idx = year_data.argmin().values
print(f"Highest impact region: {ds.regions.values[max_region_idx]} ({year_data.max().values:.8f})")
print(f"Lowest impact region: {ds.regions.values[min_region_idx]} ({year_data.min().values:.8f})")
```
