---
layout: episode
title: "Explore the CMIP data and ESMVALTool"
teaching: 60
exercises: 30
questions:
  - "What is the strcutures of the CMIP datasets?"
  - "How to add additional CMIP model datasets?"
  - "How to add a new data path of my own?"
  - "How to invoke and run the ESMValTool?"
  - "What is the structure of output files?"
#objectives:
  #- " xxx "
#keypoints:
  #- " xxx "

---

## Explore CMIP datasets by NorESM and other ESMs
logon in the ipcc node of NIRD:
```bash
ssh -l <user_name> ipcc.nird.sigma2.no
```
Explore CMIP data by NorESM and other ESMs at:
```bash
/projects/NS9034K/CMIP5
/projects/NS9034K/CMIP6
/projects/NS9560K-datalake/ESGF
```

> ## Exercise 1
* Find how many MIPs NorESM contributes to CMIP6?
* How many `table_id (mip)` does `NorESM2-LM` has for the `historical` exeriment, ensemble `r1i1p1`?
* Where is the monthly `sea_surface_temperature` data for the above experiment?
* How many different model data are available for the monthly `tos` data for the `SSP585` experiments?
>
>>## Solution
> * NorESM contributes 13 MIPs in CMIP6
> ```bash
> $ ls /projects/NS9034K/CMIP6/ |wc -l
> 13
> ```
> * `NorESM2-LM` `historical` `r1i1p1f1` has 23 `table_id (mip)`.
> ```bash
> $ cd /projects/NS9034K/CMIP6/CMIP/NCC/NorESM2-LM/historical/r1i1p1f1
> $ ls |wc -l
> 23
> ```
> * The `sea_surface_temperature` is under:
> ```bash
> /projects/NS9034K/CMIP6/CMIP/NCC/NorESM2-LM/historical/r1i1p1f1/Omon/tos/gn/v20190815
> tos_Omon_NorESM2-LM_historical_r1i1p1f1_gn_186001-186912.nc
> ...
> ```
> * There are 35 model (including ensemble) `tos` for `SSP585`:
> ```bash
> $ ls /projects/NS9560K-datalake/ESGF/CMIP6/ScenarioMIP/*/*/ssp585/*/Omon/tos |grep 'tos' |wc -l
> 35
> ```
> {: .solution}
{: .challenge}

> ## Exercise 2
* Download some datasets from one of the ESGF node (e.g., http://esgf-data.dkrz.de) and add to the NS9560K datalake ESGF pool.
* Then move to the `autosort` folder by invoking `move2autosort.sh`.
>
> >## Solution
> Download the data with `wget` script from the ESGF node, to e.g.,
> ```
> /projects/NS9560K-datalake/ESGF/rawdata/model/
> ```
> then 
> ```
> $ cd /projects/NS9560K-datalake/ESGF/rawdata/
> ./move2autosort.sh "/path/to/files*.nc /path/to/folders*.nc"
> ```
> Within 30mins, your data should be sorted to `/projects/NS9560K-datalake/ESGF/`.
> >
> Find out where it is sorted to?
> >
> Check the `failed/` folder and files in the `logs` folder.
> {: .solution}
{: .challenge}


---

## Configure and run the ESMValTool.
Clone the github repository with temporary configuration files for NIRD.
```bash
cd ~/
git clone git@github.com:NorESMhub/noresmvaltool.git
```

Copy the configuration files to your personal folder
```bash
cd ~/noresmvaltool/esmvaltool/config
cp config_user.yml config_developer.yml ~/.esmvaltool/
```
Make a folder for the workshop and activate ESMValTool
```bash
mkdir ~/esmvaltool_workshop
conda activate /diagnostics/esmvaltool/2.8.0
```
Looks at the ESMValtool command line optins:
```bash
esmvaltool -h
```
List the available ESMValTool recipes:
```bash
esmvaltool recipes list
```
These recipes are installed under the current ESMValTool installation:
```
/diagnostics/esmvaltool/2.8.0/lib/python3.10/site-packages/esmvaltool/recipes/
```
Retrieve the recipe `recipe_python.yml`
```bash
esmvaltool recipes get examples/recipe_python.yml
```
Run the recipe `recipe_python.yml`
```bash
# use the retrieved recipe at current location
esmvaltool run ./recipe_python.yml
# or use the recipe from the installation.
esmvaltool run recipe_python.yml    
# or specify the config-user.yml from a non-default location.
esmvaltool run --config_file ~/noresmvaltool/esmvaltool/config/config-user.yml ./recipe_python.yml
```
>## Example standard output
```log
______________________________________________________________________
          _____ ____  __  ____     __    _ _____           _
         | ____/ ___||  \/  \ \   / /_ _| |_   _|__   ___ | |
         |  _| \___ \| |\/| |\ \ / / _` | | | |/ _ \ / _ \| |
         | |___ ___) | |  | | \ V / (_| | | | | (_) | (_) | |
         |_____|____/|_|  |_|  \_/ \__,_|_| |_|\___/ \___/|_|
______________________________________________________________________
>
ESMValTool - Earth System Model Evaluation Tool.
>
http://www.esmvaltool.org
...
...
2023-05-27 19:47:54,820 UTC [793340] INFO    Package versions
2023-05-27 19:47:54,820 UTC [793340] INFO    ----------------
2023-05-27 19:47:54,820 UTC [793340] INFO    ESMValCore: 2.8.0
2023-05-27 19:47:54,820 UTC [793340] INFO    ESMValTool: 2.8.0
2023-05-27 19:47:54,820 UTC [793340] INFO    ----------------
2023-05-27 19:47:54,820 UTC [793340] INFO    Using config file /nird/home/yanchun/.esmvaltool/config-user.yml
2023-05-27 19:47:54,821 UTC [793340] INFO    Writing program log files to:
/projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_python_20230527_194754/run/main_log.txt
/projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_python_20230527_194754/run/main_log_debug.txt
...
...
2023-05-27 19:48:22,544 UTC [793340] INFO    Sampled every second. It may be inaccurate if short but high spikes in memory consumption occur.
2023-05-27 19:48:22,545 UTC [793340] INFO    Run was successful
```
{: .solution}

---

## Explore the output of the ESMValTool
If the run is a success, you should be able find an esmvaltool output under, giving the default `output_dir` in the template of `config_user.yml`:
```
http://ns9560k.web.sigma2.no/diagnostics/esmvaltool/
```
with subfolders of your user name:
```
[your_user_name]/tmp/recipe_python_YYYYMMDD_HHMMSS/
```
Explore these (sub)folders.

>## Note
During the phase of testing and debuging your recipes, it will likely produce a number of stamped recipe diagnostic output.
>
A `tmp/` folder is therefore used to store these temporary outputs. When you get the right results, you can move these output to its parent folder.
>
You can change this by modifying the `output_dir` in the `config_user.yml` file.
{: .discussion}

> ## Output files and directories
> Explore the output directory and files, find out:
>
> 1. Did ESMValTool use the right config file?
> 1. What is the path to the example recipe?
> 1. Can you guess what the different output directories are for?
> 1. ESMValTool creates two log files. What is the difference?
>
> (These quesitons are from the [ESMValTool Tutorial](https://esmvalgroup.github.io/ESMValTool_Tutorial).)
>
> > ## Answers
> >
> > 1. The config file should be the one we edited in the previous episode,
> >    something like `/home/<username>/.esmvaltool/config-user.yml` or
 `~/noresmvaltool/esmvaltool/config/config-user.yml`.
> > 1. ESMValTool found the recipe in its current directory or instatllation directory, for example:
> >    `~/noresmvaltool_workshop/recipe_python.yml`
>> or `/diagnostics/esmvaltool/2.8.0/lib/python3.10/site-packages/esmvaltool/recipes/`
> > 1. There should be four output folders:
> >    - `plots/`: this is where output figures are stored.
> >    - `preproc/`: this is where pre-processed data are stored.
> >    - `run/`: this is where esmvaltool stores general information about the
> >      run, such as log messages and a copy of the recipe file.
> >    - `work/`: this is where output files (not figures) are stored.
> > 1. The log files are:
> >    - `main_log.txt` is a copy of the command-line output
> >    - `main_log_debug.txt` contains more detailed information that may be
> >      useful for debugging.
> >
> {: .solution}
{: .challenge}
