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

---
### Add observatioinal dataset

Start wit the downloaded data:
```bash
$ ll /projects/NS9560K-datalake/ESGF/obsdata/Tier1/HadCRUT5/HadCRUT.5.0.1.0.analysis.anomalies.ensemble_mean.nc
-rw-rw-r-- 1 evelievd ns9560k 31M Nov  3  2021 /projects/NS9560K-datalake/ESGF/obsdata/Tier1/HadCRUT5/HadCRUT.5.0.1.0.analysis.anomalies.ensemble_mean.nc
```

This dataset seems not to be properly CMORized.

> ## Two tasks:
1. CMORize it
2. Add additonal dataset entry in the recipe
{: .challenge}


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
