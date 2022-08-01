# Heatwave based on daily temperature data

## Tools

1. Windows Subsystem for Linux (WSL) - https://docs.microsoft.com/en-us/windows/wsl/install. Since CDO is not available for Windows, the easiest way to use it is via the WSL. If you are on Linux/macOS, then you are good to go.
2. Anaconda or Miniconda
3. Climate Data Operator (CDO) - https://code.mpimet.mpg.de/projects/cdo, installed using `conda install -c conda-forge cdo`. Please create a new environment to install CDO, just to make sure the CDO will not breaking your current python environment.

## Data

1. Global high resolution (30 arc sec ~ 1km) daily maximum temperature data (available as 1 month data per 1 file) from CHELSA - https://chelsa-climate.org/, downloaded from https://data.isimip.org/datasets/92b05291-fbe5-4ed2-b3df-29ff0cded9f2/

## Case

California, USA

## How-to?

1. Clip global data using bounding box

	`for fl in ./00_chelsa_global_tasmax/*.nc; do cdo sellonlatbox,-8.3,-7.6,12.3,12.9 $fl ./01_tasmax_/usa_california_`basename $fl`; done`

2. Merge monthly data into annual

	`for year in {1979..2016}; do cdo mergetime ./01_tasmax/usa_california_chelsa-w5e5v1.0_obsclim_tasmax_30arcsec_global_daily_${year}??.nc ./02_tasmax_annual/usa_california_chelsa_daily_tasmax_${year}.nc; done`

3. Delete data who has 29th Feb

	`for fl in *.nc; do cdo -delete,month=2,day=29 ./02_tasmax_annual/$fl ./03_tasmax_del29dfeb/$fl; done`

	below script also works, using `del29dfeb`

	`for fl in *.nc; do cdo -del29dfeb ./02_tasmax_annual/$fl ./03_tasmax_del29dfeb/$fl; done`

4. Convert Kelvin to degree Celsius and don't forget to change the variable (here tasmax) units, too. Combining operators:

	`for fl in *.nc; do cdo -setattribute,tasmax@units="degC" -addc,-273.15 ./03_tasmax_del29dfeb/$fl ./04_tasmax_celsius/$fl; done`

5. Merge all nc files result from point 4 into single nc.

	`cdo mergetime ./04_tasmax_celsius/usa_*.nc ./05_tasmax_all/usa_california_chelsa_daily_tasmax_1979_2016.nc`

6. Calculate the mean TXnorm of daily maximum temperatures for any period used as reference

	`cdo ydrunmean,5,rm=c ./05_tasmax_all/usa_california_chelsa_daily_tasmax_1979_2016.nc ./06_tasmax_meanofreference/usa_california_chelsa_daily_tasmaxnorm_1979_2016.nc`

7. Calculate heat wave duration index w.r.t mean of reference period

	`for year in {1979..2016}; do cdo eca_hwdi ./04_tasmax_celsius/usa_california_chelsa_daily_tasmax_${year}.nc ./06_tasmax_meanofreference/usa_california_chelsa_daily_tasmaxnorm_1979_2016.nc ./07_hwdi/usa_california_chelsa_daily_hwdi_${year}.nc`


## Result


