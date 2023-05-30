---
layout: episode
title: "Write a simple recipe and diagnostic script"
teaching: 60
exercises: 90
questions:
  - " "
keypoints:
  #- " "

---
## Building a recipe from scratch

The basic structure of a recipe:
* [documentation](https://docs.esmvaltool.org/projects/esmvalcore/en/latest/recipe/overview.html#recipe-section-documentation)
* [datasets](https://docs.esmvaltool.org/projects/esmvalcore/en/latest/recipe/overview.html#recipe-section-datasets)
* [preprocessors](https://docs.esmvaltool.org/projects/esmvalcore/en/latest/recipe/overview.html#recipe-section-preprocessors)
* [diagnostics](https://docs.esmvaltool.org/projects/esmvalcore/en/latest/recipe/overview.html#recipe-section-diagnostics)

---
### The Task
The task is to write a recipe and diagnostic script, to calculate the averaged global `surface_temperature` in recent years, and also the time series for the recent decades, plot the results, save the figures and the calculated data (e.g., the climatology and timeseries)

We start from scratch (using the `examples/recipe_my_personal_diagnostic.yml` as template):


First, open a new file called `recipe_mydiag.yml`:

```bash
cd ~/esmvaltool_workshop
vi recipe_mydiag.yml
```

> ## Exercise: write a minimum recipe
> Let's add a minimum `documentation` part and `diagnostic` parts, as a start. We don't add actual diagnostic script, but use `null` as the placeholder for the diagnostic script.
>  ```yaml
>  # ESMValTool
>  # recipe_mydiag.yml
>  
>  documentation:
>    title:
>      <your title>
>    description:
>      <description>
>    authors:
>      - <author name>
>    maintainer:
>      - <maintainer name>
>  
>  diagnostics:
>    <diag_anme>:
>      description: <desc>
>      scripts:
        ...
>  ```
>
> > ## solution:
> > ```yaml
> > # ESMValTool
> > # recipe_mydiag.yml
> > 
> > documentation:
> >   title:
> >     An example recipe
> >   description:
> >     An example recipe created for the ESMValTool Workshop
> >   authors:
> >     - unmaintained
> >   maintainer:
> >     - unmaintained
> > 
> > diagnostics:
> >   map:
> >     description: "my test diagnostic script"
> >     scripts: null
> > ```
> {: .solution}
{: .challenge}

Notice that `yaml` always requires 2 spaces indentation between the different
levels. 

We will try to run the recipe after every modification we make, to see if it (still) works.

> ## About the authors
>
> Note, here `unmaintained` is use for the `authors` and `maintainers`.
>
> The author information must already contained in [config_references.yml](https://github.com/ESMValGroup/ESMValTool/blob/main/esmvaltool/config-references.yml).
> There is not easy way to make customized author list.
> Therefore, we use the general name `unmaintained` as the author.
{: .callout}


Try to run the recipe each time when something is changed, to see if it (still) works.
```bash
esmvaltool run recipe_mydiag.yml
```

In this case, it gives an error. Below you see the last few lines of the error message.
```
...
2023-05-29 21:53:35,903 UTC [121412] INFO    These tasks will be executed:
2023-05-29 21:53:36,260 UTC [121412] INFO    Maximum memory used (estimate): 0.0 GB
2023-05-29 21:53:36,261 UTC [121412] INFO    Sampled every second. It may be inaccurate if short but high spikes in memory consumption occur.
2023-05-29 21:53:36,261 UTC [121412] ERROR   No tasks to run!
```
{: .error}

Although there is no actual error in the recipe, ESMValTool assumes you mistakenly 
left out a variable name to process and alerts you with this error message.

We will add `variable` entry after we add the `datasets` section.

## Adding a dataset section

Let's add a datasets section. 

We have configured and explored the CMIP datasets on NIRD in the previous session [Find data and configure esmvaltool]({{site.baseurl}}/01-find-data-config-esmvaltool)
Try to pick up at least two dataset, one from NorESM and one orESM2-LM from CMIP6, and one from other model.

> ## Filling in the dataset keys
>
> Use the paths `rootpath` specified in the configuration file (`config_user.yml`) to explore the data directory, 
> and look at the explanation of the dataset section  in the [ESMValTool documentation](https://docs.esmvaltool.org/projects/ESMValCore/en/latest/recipe/overview.html#recipe-section-datasets)
>
> For both the datasets, write down the following properties:
>
> - project
> - variable (short name)
> - CMIP table
> - dataset (model name or obs/reanalysis dataset)
> - experiment
> - ensemble member
> - grid
> - start year
> - end year
>
> > ## Answers
> >
> > The following command might be used to help you list the avaiable datasets:
> > ```bash
> > $ ls /projects/NS9560K-datalake/ESGF/CMIP6/CMIP/*/*/historical/*/Amon/tas
> > $ ls /projects/NS9034K/CMIP6/CMIP/NCC/NorESM2-LM/historical/r1i1p1f1/Amon/tas/gn/v20190815/
> > $ ls /projects/NS9560K-datalake/ESGF/cmip5/output1/*/*/historical/mon/atmos/Amon/*/v1/tas
> > ```
> >
> > | **key**   | **file 1**| **file 2**|
> > | project   | CMIP6     | CMIP5     |
> > | short name| tas       | tas       |
> > | CMIP table| Amon      | Amon      |
> > | dataset   | BCC-ESM1  | CanESM2   |
> > | experiment| historical| historical|
> > | ensemble  | r1i1p1f1  | r1i1p1    |
> > | grid      | gn (native grid)| N/A |
> > | start year| 1850      | 1850      |
> > | end year  | 2014      | 2005      |
> >
> > Note that the grid key is only required for CMIP6 data, and that the extent
> > of the historical period has changed between CMIP5 and CMIP6.
> >
> {: .solution}
{: .challenge}

We start with the BCC-ESM1 dataset and add a datasets section to the recipe,
listing a single dataset, as shown below. Note that key fields such 
as `mip` or `start_year` are included in the `datasets` section here but are part 
of the `diagnostic` section in the recipe example seen in 

```yaml
datasets:
  - {dataset: BCC-ESM1, project: CMIP6, mip: Amon, exp: historical, ensemble: r1i1p1f1, grid: gn}
  - {dataset: NorESM2-LM, project: CMIP6, mip: Amon, exp: historical, ensemble: r1i1p1f1, grid: gn}
  - {dataset: EC-Earth3, project: CMIP6, mip: Amon, exp: historical, ensemble: r1i1p1f1, grid: gr}
```

The recipe should run but produce the same message as in the previous case since we
still have not included a variable to actually process.  We have not included the 
short name of the variable in this dataset section because this allows 
us to reuse this dataset entry with different variable names later on. 
This is not really necessary for our simple use case, but it is common practice 
in ESMValTool.

## Adding the preprocessor section

In the preprocessor, we will need to define preprocessors to do two tasks.
1. Make long-term average the global gridded temperature data to get a climatology
2. Make annual average and convert the global data to a timeseries of temperature.

> ## Defining the preprocessor
>
> Have a look at the available preprocessors in the
> [documentation](https://docs.esmvaltool.org/projects/esmvalcore/en/latest
/recipe/preprocessor.html).
> Write down
>
> - Which preprocessor functions do you think we should use?
> - What are the parameters that we can pass to these functions?
> - What do you think should be the order of the preprocessors?
> - A suitable name for the overall preprocessor
>
> > ## Solution
> >
> > We need to calculate global means and to calculate the timeseries.
> > We use `regrid` to remap the data to 1x1. 
> > `climate_statistics` preprocessor, to calculate long-term mean.
> > `area_statistics` preprocessor to calculate global mean.
> >
> > The default order in which these preprocessors are applied can be seen
> > [here](https://docs.esmvaltool.org/projects/esmvalcore/en/latest/api/
esmvalcore.preprocessor.html#preprocessor-functions):
> > `area_statistics` comes before `anomalies`. If you want to change this, you
> > can use the `custom_order` preprocessor. We will keep it like this.
> >
> > Let's name our preprocessor `global_anomalies`.
> {: .solution}
{: .challenge}

Add the following block to your recipe file:

```yaml
preprocessors:
  mypp_map:
    regrid:
      target_grid: 1x1
      scheme: linear
    climate_statistics:
      operator: mean
      period: full
    convert_units:
      units: degrees_C
  mypp_ts:
    regrid:
      target_grid: 1x1
      scheme: linear
    area_statistics:
      operator: mean
    annual_statistics:
      operator: mean
    convert_units:
      units: degrees_C
```

You can reuse parameters from one script to another, more [documentation](https://docs.esmvaltool.org/projects/ESMValCore/en/latest/recipe/overview.html#re-using-parameters-from-one-script-to-another)

```yaml
  mypp1:
    regrid: &regrid_setting
      target_grid: 1x1
      scheme: linear
  mypp2:
    regrid:
      <<: *regrid_setting
      #target_grid: 1x1
      #scheme: linear
```
then for `mypp2`, it will use the same configuration for the `regrid` preprocessor.

## Completing the diagnostics section

Now we are ready to finish our diagnostics section. Remember that we want to
make two tasks: a preprocessor task, and a diagnostic task. To illustrate that
we can also pass settings to the diagnostic script, we add the option to specify
a custom colormap.

> ## Fill in the blanks
>
> Extend the diagnostics section in your recipe by filling in the blanks in the
> following template:
>
> ```yaml
> diagnostics:
>   <... (suitable name for our diagnostic)>:
>     description: <...>
>     variables:
>       <... (suitable name for the preprocessed variable)>:
>         short_name: <...>
>         preprocessor: <...>
>     scripts:
>       <... (suitable name for our python script)>:
>         script: <full path to python script>
>         colormap: <... choose from matplotlib colormaps>
> ```
>
> > ## Solution
> >
> > ```yaml
> > diagnostics:
> >  diag_name:
> >    description: "my test diagnostic script"
> >    scripts:
> >      myscript:
> >        script: null
> > ```
> {: .solution}
{: .challenge}

We have not actully written the diagnostic script yet, so we put `null` in the script part.

Let's  start writing the recipe in the next exercise.

---
## Understanding an existing Python diagnostic

If you clone the ESMValTool repository, a folder called `ESMValTool` is created in your home/working directory
```bash
cd ~/
git clone https://github.com/ESMValGroup/ESMValTool.git
cd ESMValTool
```

The folder ``ESMValTool`` contains the source code of the tool. We can find the
recipe ``recipe_python.yml`` and the python script ``diagnostic.py`` in these
directories:
```bash
- ~/ESMValTool/esmvaltool/recipes/examples/recipe_python.yml
- ~/ESMValTool/esmvaltool/diag_scripts/examples/diagnostic.py
```

Let's have look at the code in  ``diagnostic.py``.
For reference, we show the diagnostic code in the dropdown box below.
There are four main sections in the script:

- A description i.e. the ``docstring`` (line 1).
- Import statements (line 2-16).
- Functions that implement our analysis (line 21-101).
- A typical Python top-level script i.e. ``if __name__ == '__main__'`` (line
  102-107).

> ## diagnostic.py
>
>~~~python
>  1:  """Python example diagnostic."""
>  2:  import logging
>  3:  from pathlib import Path
>  4:  from pprint import pformat
>  5:
>  6:  import iris
>  7:
>  8:  from esmvaltool.diag_scripts.shared import (
>  9:      group_metadata,
> 10:      run_diagnostic,
> 11:      save_data,
> 12:      save_figure,
> 13:      select_metadata,
> 14:      sorted_metadata,
> 15:  )
> 16:  from esmvaltool.diag_scripts.shared.plot import quickplot
> 17:
> 18:  logger = logging.getLogger(Path(__file__).stem)
> 19:
> 20:
> 21:  def get_provenance_record(attributes, ancestor_files):
> 22:      """Create a provenance record describing the diagnostic data and plot."""
> 23:      caption = ("Average {long_name} between {start_year} and {end_year} "
> 24:                 "according to {dataset}.".format(**attributes))
> 25:
> 26:      record = {
> 27:          'caption': caption,
> 28:          'statistics': ['mean'],
> 29:          'domains': ['global'],
> 30:          'plot_types': ['zonal'],
> 31:          'authors': [
> 32:              'andela_bouwe',
> 33:              'righi_mattia',
> 34:          ],
> 35:          'references': [
> 36:              'acknow_project',
> 37:          ],
> 38:          'ancestors': ancestor_files,
> 39:      }
> 40:      return record
> 41:
> 42:
> 43:  def compute_diagnostic(filename):
> 44:      """Compute an example diagnostic."""
> 45:      logger.debug("Loading %s", filename)
> 46:      cube = iris.load_cube(filename)
> 47:
> 48:      logger.debug("Running example computation")
> 49:      cube = iris.util.squeeze(cube)
> 50:      return cube
> 51:
> 52:
> 53:  def plot_diagnostic(cube, basename, provenance_record, cfg):
> 54:      """Create diagnostic data and plot it."""
> 55:
> 56:      # Save the data used for the plot
> 57:      save_data(basename, provenance_record, cfg, cube)
> 58:
> 59:      if cfg.get('quickplot'):
> 60:          # Create the plot
> 61:          quickplot(cube, **cfg['quickplot'])
> 62:          # And save the plot
> 63:          save_figure(basename, provenance_record, cfg)
> 64:
> 65:
> 66:  def main(cfg):
> 67:      """Compute the time average for each input dataset."""
> 68:      # Get a description of the preprocessed data that we will use as input.
> 69:      input_data = cfg['input_data'].values()
> 70:
> 71:      # Demonstrate use of metadata access convenience functions.
> 72:      selection = select_metadata(input_data, short_name='tas', project='CMIP5')
> 73:      logger.info("Example of how to select only CMIP5 temperature data:\n%s",
> 74:                  pformat(selection))
> 75:
> 76:      selection = sorted_metadata(selection, sort='dataset')
> 77:      logger.info("Example of how to sort this selection by dataset:\n%s",
> 78:                  pformat(selection))
> 79:
> 80:      grouped_input_data = group_metadata(input_data,
> 81:                                          'variable_group',
> 82:                                          sort='dataset')
> 83:      logger.info(
> 84:          "Example of how to group and sort input data by variable groups from "
> 85:          "the recipe:\n%s", pformat(grouped_input_data))
> 86:
> 87:      # Example of how to loop over variables/datasets in alphabetical order
> 88:      groups = group_metadata(input_data, 'variable_group', sort='dataset')
> 89:      for group_name in groups:
> 90:          logger.info("Processing variable %s", group_name)
> 91:          for attributes in groups[group_name]:
> 92:              logger.info("Processing dataset %s", attributes['dataset'])
> 93:              input_file = attributes['filename']
> 94:              cube = compute_diagnostic(input_file)
> 95:
> 96:              output_basename = Path(input_file).stem
> 97:              if group_name != attributes['short_name']:
> 98:                  output_basename = group_name + '_' + output_basename
> 99:              provenance_record = get_provenance_record(
>100:                  attributes, ancestor_files=[input_file])
>101:              plot_diagnostic(cube, output_basename, provenance_record, cfg)
>102:
>103:
>104:  if __name__ == '__main__':
>105:
>106:      with run_diagnostic() as config:
>107:          main(config)
>108:
>~~~
>
{:.solution}


> ## What is the starting point of a diagnostic?
>
> 1. Can you spot a function called ``main`` in the code above?
> 2. What are its input arguments?
> 3. How many times is this function mentioned?
>
>> ## Answer
>>
>> 1. The ``main`` function is defined in line 66 as ``main(cfg)``.
>> 2. The input argument to this function is the variable ``cfg``, a Python dictionary
>> that holds all the necessary
>> information needed to run the diagnostic script such as the location of input
>> data and various settings. We will next parse this ``cfg`` variable
>> in the  ``main`` function and extract information as needed
>> to do our analyses (e.g. in line 69).
>> 3. The ``main`` function is called near the very end on line 107. So, it is mentioned
>> twice in our code - once where it is called by the top-level Python script and
>> second where it is defined.
> {: .solution}
{: .challenge}

> ## The function run_diagnostic
>
> The function ``run_diagnostic`` (line 106) is called a context manager
> provided with ESMValTool and is the main entry point for most Python
> diagnostics.
>
{: .callout}

---

> ## Exercise: Write a simplist diagnostic script to interface with the preprocessed data from the recipe.
>
> 1. write a `if __name__ == '__main__'` and include `run_diagnostic()` as entry point.
> 2. write a `main()` to call the `run_diagnostic()`
>
>> ## Answer
>>
>> write the python diagnostic script:
>> ```bash
>> cd ~/esmvaltool_workshop
>> vi mydiag.py
>> ```
>> and put the following script snippet:
>> ```python
>> """ mydiag.py """
>> # import libraries for logging
>> from esmvaltool.diag_scripts.shared import run_diagnostic
>> 
>> def main(cfg):
>>     # Get a description of the preprocessed data that we will use as input.
>>     input_data = cfg['input_data'].values()
>>     print(input_data)
>> 
>> if __name__ == '__main__':
>> 
>>     with run_diagnostic() as config:
>>         main(config)
>> ```
>> The `print(input_data)` statment will print the information of the preprocessed data that the ESMValCore will pass to the diagnostic script.
>> 
>> Also update the recipe file `recipe_mydiag.yml` to include this script:
>> ```yaml
>>diagnostics:
>>  diag_name:
>>    description: "my test diagnostic script"
>>    scripts:
>>      myscript:
>>        script: ~/esmvaltool_workshop/mydiag.py
>> ```
> {: .solution}
{: .challenge}

Now, run the recipe with implemented diagnostic script:
```
cd ~/esmvaltool_workshop
esmvaltool run ./recipe_mydiag.yml
```
The log in the terminal (also found in the `main_log.txt`, e.g in `http://ns9560k.web.sigma2.no/diagnostics/esmvaltool/<your_user>/tmp/recipe_mydiag_YYYYMMDD_HHMMSS/run/main_log.txt`)

```log
INFO    [316548] Writing log to /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_110907/run/diag_name/myscript/log.txt
INFO    [316548] To re-run this diagnostic script, run:
cd /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_110907/run/diag_name/myscript; MPLBACKEND="Agg" /diagnostics/esmvaltool/2.8.0/bin/python /nird/home/yanchun/esmvaltool_workshop/mydiag.py /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_110907/run/diag_name/myscript/settings.yml

```

For example, in the log file shown above `/projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_110907/run/diag_name/myscript/log.txt`, you can find the output like:
```text
2023-05-30 11:09:12,214 [316552] INFO     esmvaltool.diag_scripts.shared._base,511| Starting diagnostic script myscript with configuration:
auxiliary_data_dir: /projects/NS9560K-datalake/ESGF/auxiliary_data
input_data: {}
input_files: []
log_level: info
output_file_type: png
plot_dir: /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_110907/plots/diag_name/myscript
recipe: recipe_mydiag.yml
run_dir: /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_110907/run/diag_name/myscript
script: myscript
version: 2.8.0
work_dir: /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_110907/work/diag_name/myscript

2023-05-30 11:09:12,215 [316552] INFO     esmvaltool.diag_scripts.shared._base,550| Creating /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_110907/work/diag_name/myscript
2023-05-30 11:09:12,215 [316552] INFO     esmvaltool.diag_scripts.shared._base,550| Creating /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_110907/plots/diag_name/myscript
2023-05-30 11:09:12,216 [316552] INFO     esmvaltool.diag_scripts.shared._base,560| End of diagnostic script run.
dict_values([])
```

This is what `print(input_data)` has write out.

And the `input_data` value is actually coming from the file `/projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_110907/run/diag_name/myscript/settings.yml`, which is the main metadata information that ESMValCore pass to the ESMValTool (i.e., the diagnostic script)

---

However, the `input_data` dictionary is empty (i.e., `input_data: {}`). That is because we have not actually added variable in the diagnostic script for preprocessing, and that is whay it is empty.

> ## Let's add some variables in the `diagnostic` section of `recipe_mydiag.yml`.
>
> > ## add in recipe_mydiag.yml
> > ```yaml
> >     variables:
> >       tas_map:
> >         short_name: tas
> >         preprocessor: mypp_map
> >         mip: Amon
> >         start_year: 2000
> >         end_year: 2014
> >     scripts:
> >       myscript:
> >         script: ~/esmvaltool_workshop/mydiag.py
> >   timeseries:
> >     description: "my test diagnostic script"
> >     variables:
> >       tas_ts:
> >         short_name: tas
> >         preprocessor: mypp_ts
> >         mip: Amon
> >         start_year: 1970
> >         end_year: 2014
> > 
> > ```
> {: .solution}
{: .challenge}

---
Run the recipe again, and you will get some real output, e.g., [http://ns9560k.web.sigma2.no/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/run/map/myscript/log.txt](http://ns9560k.web.sigma2.no/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/run/map/myscript/log.txt) and [http://ns9560k.web.sigma2.no/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/run/timeseries/myscript/log.txt](http://ns9560k.web.sigma2.no/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/run/timeseries/myscript/log.txt)

> ## more detailed output
```
2023-05-30 12:40:32,684 [355246] INFO     esmvaltool.diag_scripts.shared._base,511	Starting diagnostic script myscript with configuration:
auxiliary_data_dir: /projects/NS9560K-datalake/ESGF/auxiliary_data
input_data:
  ? /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/preproc/map/tas_map/CMIP6_EC-Earth3_Amon_historical_r1i1p1f1_tas_gr_2000-2014.nc
  : activity: CMIP
    alias: EC-Earth3
    dataset: EC-Earth3
    diagnostic: map
    end_year: 2014
    ensemble: r1i1p1f1
    exp: historical
    filename: /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/preproc/map/tas_map/CMIP6_EC-Earth3_Amon_historical_r1i1p1f1_tas_gr_2000-2014.nc
    frequency: mon
    grid: gr
    institute:
    - EC-Earth-Consortium
    long_name: Near-Surface Air Temperature
    mip: Amon
    modeling_realm:
    - atmos
    preprocessor: mypp_map
    project: CMIP6
    recipe_dataset_index: 1
    short_name: tas
    standard_name: air_temperature
    start_year: 2000
    timerange: 2000/2014
    units: degrees_C
    variable_group: tas_map
    version: v20200310
  ? /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/preproc/map/tas_map/CMIP6_NorESM2-LM_Amon_historical_r1i1p1f1_tas_gn_2000-2014.nc
  : activity: CMIP
    alias: NorESM2-LM
    dataset: NorESM2-LM
    diagnostic: map
    end_year: 2014
    ensemble: r1i1p1f1
    exp: historical
    filename: /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/preproc/map/tas_map/CMIP6_NorESM2-LM_Amon_historical_r1i1p1f1_tas_gn_2000-2014.nc
    frequency: mon
    grid: gn
    institute:
    - NCC
    long_name: Near-Surface Air Temperature
    mip: Amon
    modeling_realm:
    - atmos
    preprocessor: mypp_map
    project: CMIP6
    recipe_dataset_index: 0
    short_name: tas
    standard_name: air_temperature
    start_year: 2000
    timerange: 2000/2014
    units: degrees_C
    variable_group: tas_map
    version: v20190815
input_files:
- /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/preproc/map/tas_map/metadata.yml
log_level: info
output_file_type: png
plot_dir: /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/plots/map/myscript
recipe: recipe_mydiag.yml
run_dir: /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/run/map/myscript
script: myscript
version: 2.8.0
work_dir: /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/work/map/myscript
2023-05-30 12:40:32,685 [355246] INFO     esmvaltool.diag_scripts.shared._base,550	Creating /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/work/map/myscript
2023-05-30 12:40:32,686 [355246] INFO     esmvaltool.diag_scripts.shared._base,550	Creating /projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/plots/map/myscript
2023-05-30 12:40:32,686 [355246] INFO     esmvaltool.diag_scripts.shared._base,560	End of diagnostic script run.
dict_values([{'activity': 'CMIP', 'alias': 'EC-Earth3', 'dataset': 'EC-Earth3', 'diagnostic': 'map', 'end_year': 2014, 'ensemble': 'r1i1p1f1', 'exp': 'historical', 'filename': '/projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/preproc/map/tas_map/CMIP6_EC-Earth3_Amon_historical_r1i1p1f1_tas_gr_2000-2014.nc', 'frequency': 'mon', 'grid': 'gr', 'institute': ['EC-Earth-Consortium'], 'long_name': 'Near-Surface Air Temperature', 'mip': 'Amon', 'modeling_realm': ['atmos'], 'preprocessor': 'mypp_map', 'project': 'CMIP6', 'recipe_dataset_index': 1, 'short_name': 'tas', 'standard_name': 'air_temperature', 'start_year': 2000, 'timerange': '2000/2014', 'units': 'degrees_C', 'variable_group': 'tas_map', 'version': 'v20200310'}, {'activity': 'CMIP', 'alias': 'NorESM2-LM', 'dataset': 'NorESM2-LM', 'diagnostic': 'map', 'end_year': 2014, 'ensemble': 'r1i1p1f1', 'exp': 'historical', 'filename': '/projects/NS9560K/www/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_124004/preproc/map/tas_map/CMIP6_NorESM2-LM_Amon_historical_r1i1p1f1_tas_gn_2000-2014.nc', 'frequency': 'mon', 'grid': 'gn', 'institute': ['NCC'], 'long_name': 'Near-Surface Air Temperature', 'mip': 'Amon', 'modeling_realm': ['atmos'], 'preprocessor': 'mypp_map', 'project': 'CMIP6', 'recipe_dataset_index': 0, 'short_name': 'tas', 'standard_name': 'air_temperature', 'start_year': 2000, 'timerange': '2000/2014', 'units': 'degrees_C', 'variable_group': 'tas_map', 'version': 'v20190815'}])
```
{: .solution}

> ## complete the diagnostic script
Finally, we expand the diagnostic script to do some really anaanalysis, plot the figure, and save the figures. To do so, we need:
* Import some additional pythyon libraries (iris, matploblib, etc) to help with ploting, etc
```python
import iris
import iris.quickplot
import matplotlib.pyplot as plt
```
* Import some additional ESMValTool diagnostic interfaces for operating with the meta data information from the preprocessor
``` python
from esmvaltool.diag_scripts.shared import run_diagnostic, group_metadata
```
* Import some libraries to help with logging the results (only help to makes the logging looks better)
```python
# import libraries for logging
import logging
from pathlib import Path
from pprint import pformat
```
* And do the actuall analysis, plotting, saving in the main prgram.
```python
def main(cfg)
	...
    # Example of how to loop over dervied variables
    groups = group_metadata(input_data, 'variable_group', sort='dataset')
	for loop to loop trough dataset
		load data
		...
		plot the data
		...
		save the figure
```
>
> > ## Final diagnostic script
> >
> > ```python
> > # import libraries for logging
> > import logging
> > from pathlib import Path
> > from pprint import pformat
> > 
> > from esmvaltool.diag_scripts.shared import run_diagnostic, group_metadata
> > 
> > import iris
> > import iris.quickplot
> > import matplotlib.pyplot as plt
> > 
> > #from esmvaltool.diag_scripts.shared.plot import quickplot
> > 
> > logger = logging.getLogger(Path(__file__).stem)
> > 
> > def main(cfg):
> >     """Compute the time average for each input dataset."""
> >     # Get a description of the preprocessed data that we will use as input.
> >     input_data = cfg['input_data'].values()
> > 
> >     #selections = select_metadata(input_data, short_name='tas', project='CMIP6')
> >     #logger.info("Example of how to select only CMIP6 temperature data:\n%s",
> >                 #pformat(selections))
> > 
> >     # Example of how to loop over dervied variables
> >     groups = group_metadata(input_data, 'variable_group', sort='dataset')
> >     for group_name in groups:
> >         logger.info("Processing variable %s", group_name)
> >         for attributes in groups[group_name]:
> >             logger.info("Processing dataset %s", attributes['dataset'])
> >             input_file = attributes['filename']
> >             var_name   = attributes['short_name']
> >             cube = iris.load_cube(input_file)
> > 
> >             basename = Path(input_file).stem
> >             plot_dir = cfg['plot_dir']
> >             file_type = cfg['output_file_type']
> > 
> >             plt.figure()
> >             if "tas_map" in group_name:
> >                 iris.quickplot.pcolormesh(cube,cmap='RdBu')
> >                 plt.gca().coastlines()
> >                 plt.colorbar()
> > 
> >             if "tas_ts" in group_name:
> >                 plot = iris.quickplot.plot(cube)
> > 
> >             plt.savefig(plot_dir+'/'+group_name+'_'+basename+'.'+file_type)
> > 
> >             """
> >             if cfg.get('quickplot'):
> >                 #Create the plot
> >                 quickplot(cube, **cfg['quickplot'])
> >                 #And save the plot
> >                 save_figure(basename, provenance_record, cfg)
> >             """
> > 
> > 
> > if __name__ == '__main__':
> > 
> >     with run_diagnostic() as config:
> >         main(config)
> > ```
> {: .solution}
{: .challenge}

---
## Browser the final result at:
[http://ns9560k.web.sigma2.no/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_140124/](http://ns9560k.web.sigma2.no/diagnostics/esmvaltool/yanchun/tmp/recipe_mydiag_20230530_140124/)

**Note**, there is no figure shown in the `index` page, probablly because we don't use the esmvaltool `save_figure()` function. And we lack the `provenance_record` for the generated data and plots, as it is not properly done yet in the diagnostic script.
