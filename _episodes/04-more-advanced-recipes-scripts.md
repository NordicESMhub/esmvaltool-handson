---
layout: episode
title: "More on writing diagnostic scripts"
teaching: 60
exercises: 120
questions:
  - " "
#keypoints:
  #- " "

---
## Add observational datasets, save provenance records of data and plots

We continue working with the `recipe_mydiag.yml` and `mydiag.py`.

Here are two tasks:
1. add an observational dataset
2. save the data and plots provenance record

> ## full recipe_mydiag.yml
```yaml
> # ESMValTool
> # recipe_mydiag.yml
> 
> documentation:
>   title:
>     An example recipe
>   description:
>     An example recipe created for the ESMValTool Workshop
>   authors:
>     - unmaintained
>   maintainer:
>     - unmaintained
> 
> datasets:
> #- {dataset: bcc-csm1-1, project: CMIP5, mip: Amon, exp: historical, ensemble: r2i1p1, grid: gn}
> #- {dataset: CanESM2, project: CMIP5, mip: Amon, exp: historical, ensemble: r1i1p1, grid: gn}
> #- {dataset: BCC-ESM1, project: CMIP6, mip: Amon, exp: historical, ensemble: r1i1p1f1, grid: gn}
>  - {dataset: NorESM2-LM, project: CMIP6, mip: Amon, exp: historical, ensemble: r1i1p1f1, grid: gn}
>  - {dataset: EC-Earth3, project: CMIP6, mip: Amon, exp: historical, ensemble: r1i1p1f1, grid: gr}
> 
> preprocessors:
>   mypp_map:
>     regrid: &regrid_setting
>       target_grid: 1x1
>       scheme: linear
>     climate_statistics:
>       operator: mean
>       period: full
>     convert_units:
>       units: degrees_C
>   mypp_ts:
>     regrid:
>       <<: *regrid_setting
>       #target_grid: 1x1
>       #scheme: linear  
>     area_statistics:
>       operator: mean
>     annual_statistics:
>       operator: mean
>     convert_units:
>       units: degrees_C
> 
> diagnostics:
>   map:
>     description: "my test diagnostic script"
>     variables:
>       tas_map:
>         short_name: tas
>         preprocessor: mypp_map
>         mip: Amon
>         start_year: 2000
>         end_year: 2014
>     scripts:
>       myscript:
>         script: ~/esmvaltool_workshop/mydiag.py
>   timeseries:
>     description: "my test diagnostic script"
>     variables:
>       tas_ts:
>         short_name: tas
>         preprocessor: mypp_ts
>         mip: Amon
>         start_year: 1970
>         end_year: 2014
>     scripts:
>       myscript:
>         script: ~/esmvaltool_workshop/mydiag.py
> 
```
{: .solution}

> ## full mydiag.py
> ```python
> # import libraries for logging
> import logging
> from pathlib import Path
> from pprint import pformat
> 
> from esmvaltool.diag_scripts.shared import run_diagnostic, group_metadata
> 
> import iris
> import iris.quickplot
> import matplotlib.pyplot as plt
> 
> #from esmvaltool.diag_scripts.shared.plot import quickplot
> 
> logger = logging.getLogger(Path(__file__).stem)
> 
> def main(cfg):
>     """Compute the time average for each input dataset."""
>     # Get a description of the preprocessed data that we will use as input.
>     input_data = cfg['input_data'].values()
> 
>     #selections = select_metadata(input_data, short_name='tas', project='CMIP6')
>     #logger.info("Example of how to select only CMIP6 temperature data:\n%s",
>                 #pformat(selections))
> 
>     # Example of how to loop over dervied variables
>     groups = group_metadata(input_data, 'variable_group', sort='dataset')
>     for group_name in groups:
>         logger.info("Processing variable %s", group_name)
>         for attributes in groups[group_name]:
>             logger.info("Processing dataset %s", attributes['dataset'])
>             input_file = attributes['filename']
>             var_name   = attributes['short_name']
>             cube = iris.load_cube(input_file)
> 
>             basename = Path(input_file).stem
>             plot_dir = cfg['plot_dir']
>             file_type = cfg['output_file_type']
> 
>             plt.figure()
>             if "tas_map" in group_name:
>                 iris.quickplot.pcolormesh(cube,cmap='RdBu')
>                 plt.gca().coastlines()
>                 plt.colorbar()
> 
>             if "tas_ts" in group_name:
>                 plot = iris.quickplot.plot(cube)
> 
>             plt.savefig(plot_dir+'/'+group_name+'_'+basename+'.'+file_type)
> 
>             """
>             if cfg.get('quickplot'):
>                 #Create the plot
>                 quickplot(cube, **cfg['quickplot'])
>                 #And save the plot
>                 save_figure(basename, provenance_record, cfg)
>             """
> 
> 
> if __name__ == '__main__':
> 
>     with run_diagnostic() as config:
>         main(config)
> 
```
{: .solution}

---
### Add observatioinal dataset

We get know that the surface air temperature `tas` is available in the observational data `HadCRUT5`, in [supported-datasets-for-which-a-cmorizer-script-is-available](https://docs.esmvaltool.org/en/latest/input.html#supported-datasets-for-which-a-cmorizer-script-is-available)

So we try to add the `HadCRUT5` as an observational data.

---
Let start with downloadeding data:

Before that, we find some has already put the data:
```bash
$ ll /projects/NS9560K-datalake/ESGF/obsdata/Tier1/HadCRUT5/HadCRUT.5.0.1.0.analysis.anomalies.ensemble_mean.nc
-rw-rw-r-- 1 evelievd ns9560k 31M Nov  3  2021 /projects/NS9560K-datalake/ESGF/obsdata/Tier1/HadCRUT5/HadCRUT.5.0.1.0.analysis.anomalies.ensemble_mean.nc
```

> ## Note
```bash
yanchun@ipcc:/projects/NS9560K-datalake/ESGF/obsdata/Tier1
$ ll
total 1.3G
drwxrwsr-x 3 augustinm ns9560k 4.0K Oct 28  2021 AERONET/
drwxrwsr-x 2 tomast    ns9560k 4.0K Feb 10  2021 ATSR/
drwxrwsr-x 2 yanchun   ns9560k 4.0K Jul 30  2020 CERES-EBAF/
drwxrwsr-x 2 yanchun   ns9560k 4.0K Jul 30  2020 GPCP-SG/
drwxrwsr-x 2 evelievd  ns9560k 4.0K Nov  6  2021 HadCRUT5/
drwxrwsr-x 3 augustinm ns9560k 4.0K Oct 28  2021 MODIS/
drwxrwsr-x 3 yanchun   ns9560k 4.0K Jul 30  2020 NASA-GSFC/
drwxrwsr-x 3 yanchun   ns9560k 4.0K Jul 30  2020 NASA-JPL/
drwxrwsr-x 3 yanchun   ns9560k 4.0K May 24 13:30 NASA-LaRC/
-rw-r--r-- 1 evelievd  ns9560k 440M Jul 23  2020 OBS_HadISST_reanaly_1_Amon_ts_187001-201712.nc
-rw-r--r-- 1 evelievd  ns9560k 440M Jul 23  2020 OBS_HadISST_reanaly_1_OImon_sic_187001-201712.nc
-rw-r--r-- 1 evelievd  ns9560k 440M Jul 23  2020 OBS_HadISST_reanaly_1_Omon_tos_187001-201712.nc
lrwxrwxrwx 1 evelievd  ns9560k   91 Aug 23  2021 OBS_HadISST_reanaly_1_SImon_sic_187001-201712.nc -> /projects/NS9252K/obsdata/Tier2/HadISST/OBS_HadISST_reanaly_1_SImon_siconc_187001-201712.nc
```
>
This `HadCRUT` dataset seems not to be properly CMORized.
>
```bash
$ ncdump -h HadCRUT.5.0.1.0.analysis.anomalies.ensemble_mean.nc
netcdf HadCRUT.5.0.1.0.analysis.anomalies.ensemble_mean {
dimensions:
	time = 2061 ;
	latitude = 36 ;
	longitude = 72 ;
	bnds = 2 ;
variables:
	double tas_mean(time, latitude, longitude) ;
		tas_mean:_FillValue = -1.e+30 ;
		tas_mean:long_name = "blended air_temperature_anomaly over land with sea_water_temperature_anomaly" ;
		tas_mean:units = "K" ;
		tas_mean:cell_methods = "area: mean (interval: 5.0 degrees_north 5.0 degrees_east) time: mean (interval: 1 month) realization: mean" ;
		tas_mean:coordinates = "realization" ;
	double time(time) ;
		time:axis = "T" ;
		time:bounds = "time_bnds" ;
		time:units = "days since 1850-01-01 00:00:00" ;
		time:standard_name = "time" ;
		time:long_name = "time" ;
		time:calendar = "gregorian" ;
	double time_bnds(time, bnds) ;
	double latitude(latitude) ;
		latitude:axis = "Y" ;
		latitude:bounds = "latitude_bnds" ;
		latitude:units = "degrees_north" ;
		latitude:standard_name = "latitude" ;
		latitude:long_name = "latitude" ;
	double latitude_bnds(latitude, bnds) ;
	double longitude(longitude) ;
		longitude:axis = "X" ;
		longitude:bounds = "longitude_bnds" ;
		longitude:units = "degrees_east" ;
		longitude:standard_name = "longitude" ;
		longitude:long_name = "longitude" ;
	double longitude_bnds(longitude, bnds) ;
	int64 realization ;
		realization:bounds = "realization_bnds" ;
		realization:units = "1" ;
		realization:standard_name = "realization" ;
	int64 realization_bnds(bnds) ;
```
{: .discussion}

> ## Two tasks:
1. Get the write data, CMORize it, put it to the right place
2. Add additonal dataset entry in the recipe
{: .challenge}

> ## First, we have a peak of supported data list
> 
> > ## Solution
> > ```bash
> > esmvaltool data list
> > ```
> > ```output
> > | Dataset name                   | Tier | Auto-download | Last access |
> > -----------------------------------------------------------------------
> > ...
> > | GRACE                          |    3 | No            |  2020-11-27 |
> > | HadCRUT3                       |    2 | Yes           |  2019-02-21 |
> > | HadCRUT4                       |    2 | Yes           |  2019-02-08 |
> > | HadCRUT5                       |    2 | Yes           |  2022-03-28 |
> > | HadISST                        |    2 | Yes           |  2019-02-08 |
> > | HALOE                          |    2 | Yes           |  2020-03-11 |
> > ...
> > ```
> {: .solution}
{: .challenge}

> ## look at some details of the dataset
> 
> > ## Solution
> > ```bash
> > $ esmvaltool data info HadCRUT5
> > HadCRUT5
> > 
> > Tier: 2
> > Source: https://crudata.uea.ac.uk/cru/data/temperature
> > Automatic download: Yes
> > 
> > Download the following files:
> >   infilling
> >       [Source]/HadCRUT.5.0.1.0.analysis.anomalies.ensemble_mean.nc
> >   no-infilling
> >       [Source]/HadCRUT.5.0.1.0.anomalies.ensemble_mean.nc
> >   climatology
> >       [Source]/absolute_v5.nc
> > ```
> {: .solution}
{: .challenge}

> ## Then, we download the data
> 
> > ## Solution
> > ```
> > esmvaltool data download HadCRUT5
> > ...
> > 2023-05-31 11:14:24 (17.7 MB/s) - ‘/projects/NS9560K-datalake/ESGF/rawdata/obs/Tier2/HadCRUT5/HadCRUT.5.0.1.0.anomalies.ensemble_mean.nc’ saved [23466672/23466672]
> > ...
> > 2023-05-31 11:14:24 (460 KB/s) - ‘/projects/NS9560K-datalake/ESGF/rawdata/obs/Tier2/HadCRUT5/absolute.nc’ saved [63192/63192]
> > 
> > 2023-05-31 09:14:24,479 UTC [648601] INFO    HadCRUT5 downloaded
> > ```
> {: .solution}
{: .challenge}

One can check the data with `ncdump`.
 
> ## Now, let's reformat it
> 
> > ## Solution
> > ```
> > esmvaltool data format HadCRUT5
> > ```
> > 2023-05-31 09:15:38,166 UTC [649125] INFO    Saving: /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/data_formatting_20230531_091536/Tier2/HadCRUT5/OBS_HadCRUT5_ground_5.0.1.0-noninfilled_Amon_tasa_185001-202303.nc
> > 2023-05-31 09:15:38,167 UTC [649125] INFO    Cube has lazy data [lazy is preferred]
> > 2023-05-31 09:15:38,555 UTC [649125] INFO    CMORization of dataset HadCRUT5 finished!
> > 2023-05-31 09:15:38,555 UTC [649125] INFO    Formatting successful for dataset HadCRUT5
{: .solution}
{: .challenge}

> ## Move the cmorized file to the obs datalake
>
> As a test for everyone, it is better you move this to your own /scratch folder
> > ## Solution
> > ```
> > mkdir -p /sratch/$USER/ESGF/obsdata/Tier2/HadCRUT5
cp /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/data_formatting_20230531_091536/Tier2/HadCRUT5/OBS_HadCRUT5_ground_5.0.1.0-* /sratch/$USER/ESGF/obsdata/Tier2/HadCRUT5
> > ```
> {: .solution}
{: .challenge}



> ## Add an additional_dataset entry in your recipe:
```yaml
    additional_datasets:
      - {dataset: HadCRUT5, project: OBS, type: analysis, mip: Amon, version: 1, tier: 2, start_year: 2000, end_year: 2004}
```
And then run the recipe again
```bash
esmvaltool run ./recipe_mydiag.yml
```
You will get some error again:
```
2023-05-31 09:21:43,268 UTC [652706] ERROR   Looked for files matching
/scratchs/yanchun/ESGF/obsdata/Tier2/HadCRUT5/OBS_HadCRUT5_analysis_1_Amon_tas[_.]*nc
2023-05-31 09:21:43,268 UTC [652706] ERROR   Set 'log_level' to 'debug' to get more information
2023-05-31 09:21:43,268 UTC [652706] ERROR   Could not create all tasks
2023-05-31 09:21:43,269 UTC [652706] ERROR   Missing data for preprocessor map/tas_map:
- Missing data for Dataset: tas, Amon, OBS, HadCRUT5, 1
2023-05-31 09:21:43,269 UTC [652706] ERROR   Missing data for preprocessor timeseries/tas_ts:
- Missing data for Dataset: tas, Amon, OBS, HadCRUT5, 1
```
>
That is the name set by the ESMValTool format is wrong. Try to manually rename it.
```bash
mv OBS_HadCRUT5_ground_5.0.1.0-analysis_Amon_tas_185001-202303.nc OBS_HadCRUT5_analysis_1_Amon_tas_185001-202303.nc
```
{: .solution}

**The recipe should work now! 🎉**

---
### Add data provenance record

Use the diagnostic script as an example:
https://github.com/NordicESMhub/ESMValTool-recipes/blob/main/diag_scripts/cbf/diag_cbfs.py

Check how the following function call the `ProvenanceLogger`, and save the prvenance record.
```python
save_data(basename, provenance_record, cfg, cbf)
save_text(basename, provenance_record, cfg, cbf_perc)
save_figure(basename, provenance_record, cfg)
```
> ## tasks:
1. save the provenance record of the plots
2. save the provenance record of global mean and globally-averaged timeseries of surface air temperature
{: .challenge}
