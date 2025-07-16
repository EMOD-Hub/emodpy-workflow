# How-tos
The following list provides helpful snippets on how to do things with emodpy-workflow.

## **Make a new project**

To make a new project (directory), execute the following:

```bash
python -m emodpy_workflow.scripts.new_project -d DIRECTORY
```

... where DIRECTORY is the path to the project directory to create.

---

## **Make a new frame**

To make a new frame of an existing emodpy-hiv country model, execute the following:

```bash
python -m emodpy_workflow.scripts.new_frame --country COUNTRY --dest FRAME
```

... where COUNTRY is the country model class name and FRAME is the desired name of the frame to create. The created
frame will be in the &lt;project_directory&gt;/frames/FRAME directory.

---

## **Extend an existing frame**

To extend an existing frame, which imports an existing frame as the starting point of a new frame, execute the 
following:

```bash
python -m emodpy_workflow.scripts.extend_frame --source SOURCE --dest FRAME
```

... where SOURCE is the name of the frame to be extended and FRAME is the name of the frame to create.

---

## **Add additional elements to a frame**

### Config

To add a new hyperparameter to the config, one needs to add a new ParameterizedCall to config.py in the chosen frame.

For example, to add a hyperparameter named **Base_Infectivity** that allows modification to the same-named EMOD-HIV 
config parameter in a frame named **baseline**, add the following new function to frames/baseline/config.py:

```python
def modify_base_infectivity(config, Base_Infectivity: int = None):
    if Base_Infectivity is not None:
        config.parameters.Base_Infectivity = Base_Infectivity
```

... then create a ParameterizedCall object utilizng the new function and add it to the list of config 
ParameterizedCalls. Modify function **get_config_parameterized_calls** in frames/baseline/config.py to be the following:

```python
def get_config_parameterized_calls(config: ReadOnlyDict) -> List[ParameterizedCall]:
    parameterized_calls = country_model.get_config_parameterized_calls(config=config)
    # Add any additional ParameterizedCalls here
    pc = ParameterizedCall(func=modify_base_infectivity, hyperparameters={'Base_Infectivity': None})
    parameterized_calls.append(pc)
    return parameterized_calls
```

The hyperparameter named **Base_Infectivity** will now be available for use.

### Campaign

To add a new campaign element (often called an intervention), one needs to create and add an appropriate intervention 
object in a function and then a new ParameterizedCall utilizing it to campaign.py in the chosen frame.

For example, to add an HIV vaccine to a frame named **hiv_vaccine** that was created with the **extend_frame** command,
add the following new function creating a vaccine intervention to frames/hiv_vaccine/campaign.py:

```python
from emodpy_hiv.campaign.individual_intervention import ControlledVaccine
from emodpy_hiv.campaign.distributor import add_intervention_triggered
from emodpy_hiv.campaign.waning_config import Constant

def add_hiv_vaccine(campaign: emod_api.campaign, vaccine_efficacy: float = 1.0):
    hiv_vaccine = ControlledVaccine(campaign=campaign,
                                    waning_config=Constant(constant_effect=vaccine_efficacy))
    add_intervention_triggered(campaign=campaign,
                               intervention_list=[hiv_vaccine],
                               triggers_list=["STIDebut"],
                               start_year=2030)
    return campaign
```

... then create a ParameterizedCall object utilizng the new function and add it to the list of campaign 
ParameterizedCalls. Modify function **get_campaign_parameterized_calls** in frames/hiv_vaccine/campaign.py to be the 
following:

```python
def get_campaign_parameterized_calls(campaign: emod_api.campaign) -> List[ParameterizedCall]:
    parameterized_calls = source_frame.model.campaign_parameterizer(campaign=campaign)
    # Add any additional ParameterizedCalls here
    pc = ParameterizedCall(func=add_hiv_vaccine, hyperparameters={'vaccine_efficacy': None})
    parameterized_calls.append(pc)
    return parameterized_calls
```

The hyperparameter named **vaccine_efficacy** will now be available for use.

---

## **Replace an element of a frame**

### Campaign or Demographics

Overriding an element of an EMOD-HIV campaign or demographics involves creating a Python subclass of the country model 
you wish to use and adding overriding function(s) of the same name(s) of the country model function(s) you wish to 
replace.

For example, creating a Python country model subclass named **ZambiaModified** using an alternate 
**add_state_TestingOnChild6w** function for the Zambia model campaign in frame **zambia_modified**. 

First, use the **new_frame** command to create a fresh Zambia country model starting point
(see [Make a new frame](#how-to-make-a-new-frame)):

```bash
python -m emodpy_workflow.scripts.new_frame --country Zambia --dest zambia_modified
```

Then update the original Zambia country model import near the top:

```python
# Old line here
from emodpy_hiv.countries import Zambia as country_model

# Use this line instead
from emodpy_hiv.countries import Zambia
```

Then add the following new Zambia country model Python subclass in frames/zambia_modified/campaign.py containing the
desired override/replacement. The additional naming line at the end updates the rest of the file to use the new country
model.

In this example, we are copy/pasting the original Zambia **add_state_TestingOnChild6w** function and modifying the 
internal **child_testing_time_value_map** values to be two years earlier than the original:

```python
class ZambiaModified(Zambia):
    @classmethod
    def add_state_TestingOnChild6w(cls, campaign: emod_api.campaign, node_ids: Union[List[int], None] = None):
        child_testing_start_year = 2002
        child_testing_time_value_map = {"Times": [2002, 2003, 2004, 2006, 2007],
                                        "Values": [0, 0.03, 0.1, 0.2, 0.3365]}
        disqualifying_properties = [coc.CascadeState.LOST_FOREVER,
                                    coc.CascadeState.ON_ART,
                                    coc.CascadeState.LINKING_TO_ART,
                                    coc.CascadeState.ON_PRE_ART,
                                    coc.CascadeState.LINKING_TO_PRE_ART,
                                    coc.CascadeState.ART_STAGING,
                                    coc.CascadeState.TESTING_ON_SYMPTOMATIC]
        property_restrictions = 'Accessibility:Yes'
        coc.add_state_TestingOnChild6w(campaign=campaign,
                                       disqualifying_properties=disqualifying_properties,
                                       time_value_map=child_testing_time_value_map,
                                       node_ids=node_ids,
                                       property_restrictions=property_restrictions,
                                       start_year=child_testing_start_year)

country_model = ZambiaModified
```

The frame **zambia_modified** now is identical to a Zambia country model frame but with the targeted section of the
campaign replaced.

The process for replacing demographics elements is identical, but frames/zambia_modified/demographics.py is edited 
instead.

---

## **Specify an ingest form for a frame**

The way to specify the ingest form to use for **all** frames in a project is by editing the **ingest_filename** attribute in its
**manifest.py** file.

e.g.

```python
ingest_forms_dir = os.path.join(os.getcwd(), 'calibration', 'ingest_forms')
ingest_filename = os.path.join(ingest_forms_dir, 'Zambia_calibration_ingest_form_2022-05-19__source__edited_for_calibration_testing--ALL_NODE.xlsm')
```

... sets the ingest file to use to be the file at path:

&lt;project_directory&gt;/calibration/ingest_forms/Zambia_calibration_ingest_form_2022-05-19__source__edited_for_calibration_testing--ALL_NODE.xlsm

To **override** this ingest_filename path **for a specific frame only** requires an edit to the chosen frame's EMOD_HIV
object specification in its \_\_init__.py file. Change "manifest.ingest_filename" below in your chosen frame to the
desired path:

```python
model = EMOD_HIV(
    manifest=manifest,
    config_initializer=config.initialize_config,
    config_parameterizer=config.get_config_parameterized_calls,
    demographics_initializer=demographics.initialize_demographics,
    demographics_parameterizer=demographics.get_demographics_parameterized_calls,
    campaign_initializer=campaign.initialize_campaign,
    campaign_parameterizer=campaign.get_campaign_parameterized_calls,
    ingest_form_path=manifest.ingest_filename,  # <-- change the value here for single-frame update only
    build_reports=config.build_reports
)
```

---

## **List available hyperparameters in a frame**

To find all hyperparameters that are available for use (e.g. calibration, scenario design), execute the following:

```bash
python -m emodpy_workflow.scripts.available_parameters -F FRAME
```

... where FRAME is the name of the frame to be inspected.

---

## **List duplicative hyperparameters in a frame (intentional or not)**

To find all hyperparameters that are used **more than once** in a frame, execute the following:

```bash
python -m emodpy_workflow.scripts.available_parameters -F FRAME
```

... where FRAME is the name of the frame to be inspected. Duplicated hyperparameters are listed at the bottom of the
result.

Duplicated hyperparameters can be **intentional** if a
hyperparameter is intended to modify more than one ParameterizedCall value. They can also be **unintentional** if an
existing hyperparameter name and label has been inadvertently reused. It is important to confirm that only intentional
duplication exists to ensure model hyperparameter values are set as expected.

---

## **Calibrate a frame**

The basic command to calibrate a frame to reference data in an ingest form using the optim_tool algorithm is:

```bash
python -m emodpy_workflow.scripts.calibrate -F FRAME -p PLATFORM optim_tool
```

... where FRAME is the name of the frame to calibrate and PLATFORM is the idmtools.ini platform name to run on.

There are numerous additional parameters that can be set to control the behavior of the process. They are specified
**before** the algorithm name (e.g. optim_tool) on the command line. 

To see parameters for controlling the calibration process, run:

```bash
python -m emodpy_workflow.scripts.calibrate -h
```

Each optimization algorithm (e.g. optim_tool) has its own parameters that are specified **after** the algorithm name. To 
see parameters for controlling the details of the chosen optimization algorithm, run:

```bash
python -m emodpy_workflow.scripts.calibrate optim_tool -h
```

---

## **Resample a calibration**

Resampling a calibration is the process of selecting one or more sets of parameters from a calibration. 
These parameter sets are effectively "a model calibration", the end result of a calibration process. They also contain
the random seed (Run_Number) to allow exact recreation of a simulation.

To Resample a calibration, run the following:

```bash
python -m emodpy_workflow.scripts.resample -d CALIBRATION_DIR -m METHOD -n NUMBER -o FILE
```

... where CALIBRATION_DIR is the directory path of a calibration process that has been performed, METHOD is the
sampling algorithm, NUMBER is the count of parameter sets to select, and FILE is the file path to write csv results to.

---

## **Create a scenario**

A scenario is a "what if" variation applied on top of a frame configuration. To create a scenario, first create a sweep
file to define a set of hyperparameter value overrides for the scenario.

For example, if one wishes to define a scenario modifying the **Base_Infectivity** and 
**condom_usage_max--COMMERCIAL-ALL-NODES** hyperparameter values of a frame named **baseline**, the following will
create 1 suite containing 1 experiment of 1 simulation, applying the specified overrides to the baseline frame
configuration.

Create a file named: **sweeps.py** ... containing the following:
```python
frame_name = 'baseline'

parameter_sets = {
    frame_name: {
        'sweeps': [{'experiment_name': "Base_Infectivity_and_condom_usage_max_scenario",
                    'Base_Infectivity': 0.0008,
                    'condom_usage_max--COMMERCIAL-ALL-NODES': 0.5}]
    }
}
```

Then run:

```bash
python -m emodpy_workflow.scripts.run -F baseline -p PLATFORM -o OUTPUT -N BaseInfectivityAndCondomMaxScenario -S sweeps.py
```

... where PLATFORM is the idmtools.ini platform name to run on and  OUTPUT is the directory for storing the run receipt 
file.

---

## **Create a set of scenarios**

First, please see [Create a scenario](#create-a-scenario). Then use the following for sweeps.py instead, which 
will create 1 suite containing 3 experiments of 1 simulation, applying the specified overrides to the baseline frame
configuration.

```python
frame_name = 'baseline'

parameter_sets = {
    frame_name: {
        'sweeps': [{'experiment_name': "Base_Infectivity_and_condom_usage_max_scenario_1",
                    'Base_Infectivity': 0.0008,
                    'condom_usage_max--COMMERCIAL-ALL-NODES': 0.5},
                   {'experiment_name': "Base_Infectivity_and_condom_usage_max_scenario_2",
                    'Base_Infectivity': 0.0006,
                    'condom_usage_max--COMMERCIAL-ALL-NODES': 0.7},
                   {'experiment_name': "Base_Infectivity_and_condom_usage_max_scenario_3",
                    'Base_Infectivity': 0.0004,
                    'condom_usage_max--COMMERCIAL-ALL-NODES': 0.9}
                   ]
    }
}
```

Then run:

```bash
python -m emodpy_workflow.scripts.run -F baseline -p PLATFORM -o OUTPUT -N BaseInfectivityAndCondomMaxScenarios3 -S sweeps.py
```

... where PLATFORM is the idmtools.ini platform name to run on and  OUTPUT is the directory for storing the run receipt 
file.

---

## **Explore the default behavior of a country model frame**

A single simulation of a default county model configuration can be run as follows:

```bash
python -m emodpy_workflow.scripts.run -F FRAME -p PLATFORM -o OUTPUT -N SUITE_NAME
```

... where FRAME is the name of a frame **created with the "new_frame" command and has not been further modified**, 
PLATFORM is the idmtools.ini platform name to run on, OUTPUT is the directory for storing the run receipt file, and
SUITE_NAME is a meaningful name/description of suite (of one experiment, of one simulation) for identification.

---

## **Explore hyperparameter sensitivity of a frame**

To create a set of simulations that differ only by the value of a single hyperparameter, first create a sweep file to 
designate a set of values for the chosen hyperparameter.

For example, if one wishes to explore the sensitivity of the hyperparameter **formation_rate--INFORMAL** in a frame
named **baseline** with 5 different values, the following will create 1 suite containing 5 experiments with 1
simulation each, utilizing 5 different formation_rate--INFORMAL values.

Create a file named: **formation_rate_sweeps.py** ... containing the following (first three rows can be adjusted as needed):

```python
frame_name = 'baseline'
experiment_name_template = "formation_rate--INFORMAL_sensitivity_%g"
formation_rates = [0.0001, 0.0002, 0.0003, 0.0004, 0.0005]

parameter_sets = {
    frame_name: {
        'sweeps': [{'experiment_name': experiment_name_template % rate, 'formation_rate--INFORMAL': rate}
                   for rate in formation_rates]
    }
}
```

Then run:

```bash
python -m emodpy_workflow.scripts.run -F baseline -p PLATFORM -o OUTPUT -N BaselineInformalFormationRate5 -S formation_rate_sweeps.py
```

... where PLATFORM is the idmtools.ini platform name to run on and  OUTPUT is the directory for storing the run receipt 
file.

---

## **Explore internal variability of a frame**

Internal model variability is the variation due solely to the random number sequence used. To create a set of
simulations with identical configuration but with varied random number sequences (Run_Number hyperparameter), first 
create a sweep file to designate a set of Run_Number values to utilize, one per simulation.

For example, if one wishes to explore the internal variability of a frame named **baseline** with **25** simulations, 
the following will create 1 suite containing 25 experiments with 1 simulation each, utilizing 25 different Run_Number 
values.

Create a file named: **run_number_sweeps.py** ... containing the following (first three rows can be adjusted as needed):

```python
frame_name = 'baseline'
n_simulations = 25
experiment_name_template = "internal_variability_run_number_%d"

run_numbers = range(n_simulations)
parameter_sets = {
    frame_name: {
        'sweeps': [{'experiment_name': experiment_name_template % rn, 'Run_Number': rn} 
                   for rn in run_numbers]
    }
}
```

Then run:

```bash
python -m emodpy_workflow.scripts.run -F baseline -p PLATFORM -o OUTPUT -N BaselineInternalVariability25 -S run_number_sweeps.py
```

... where PLATFORM is the idmtools.ini platform name to run on and  OUTPUT is the directory for storing the run receipt 
file.

---

## **Run scenarios with a calibrated frame**

Running scenarios on top of a calibrated frame requires using the **run** command with both a resampled csv file (from
the **resample** command) and a scenario sweep file as input. Every parameter set in the resampled csv file will form 
the basis for a simulation in every scenario.

For example, to run a set of three scenarios varying a couple of coinfection hyperparameters (including one no-change 
scenario) using the **baseline** frame on the ContainerPlatform:

Assume **resampled_parameter_sets.csv** file exists with 200 calibrated parameter sets.

With the following **sweeps.py** file:
```python
parameter_sets = {
    'baseline': {
        'sweeps': [
            {'experiment_name': 'higher_coinfection', 'coinfection_coverage_HIGH': 0.5, 'coinfection_coverage_LOW': 0.4},
            {'experiment_name': 'lower_coinfection', 'coinfection_coverage_HIGH': 0.2, 'coinfection_coverage_LOW': 0.02},
            {},  # This runs with no modifications over the baseline and is auto-named "baseline"
        ]
    }
}
```

Run:

```bash
python -m emodpy_workflow.scripts.run -F baseline -p ContainerPlatform -o OUTPUT -N SUITE_NAME -s resampled_parameter_sets.csv -S sweeps.py
```

... where OUTPUT is the directory for storing the run receipt file and
SUITE_NAME is a meaningful name/description of the suite created. The result will be 1 suite of 3 experiments each with
200 simulations (600 simulations in total). Each simulation in a given experiment will use a different calibrated 
parameter set, overridden by the specific parameters in the corresponding sweeps.py experiment entry..

---

## **Download a file per simulation**

There is a single download command to download files from simulations. However, there are multiple ways to specify
**which** simulations to download files from.

The following examples assume one wishes to download file: output/ReportHIVByAgeAndGender.csv ... on platform: 
ContainerPlatform .

### To download a file from all simulations in an experiment:

```bash
python -m emodpy_workflow.scripts.download -f output/ReportHIVByAgeAndGender.csv -p ContainerPlatform --exp-id EXP_ID -o OUTPUT_DIR
```

... where EXP_ID is the unique id of an experiment and OUTPUT_DIR is the directory to store downloaded files.

### To download a file from all simulations in a suite of experiments:

```bash
 python -m emodpy_workflow.scripts.download -f output/ReportHIVByAgeAndGender.csv -p ContainerPlatform --suite-id SUITE_ID -o OUTPUT_DIR
 ```

... where SUITE_ID is the unique id of a suite and OUTPUT_DIR is the directory to store downloaded files.

### To download a file from simulations specified in a resampled parameter sets csv file:

```bash
python -m emodpy_workflow.scripts.download -f output/ReportHIVByAgeAndGender.csv -p ContainerPlatform -o OUTPUT_DIR -s RESAMPLE_FILE
```

... where RESAMPLE_FILE is the path of a **resample** command result and OUTPUT_DIR is the directory to store downloaded 
files.

### To download a file from simulations specified in a run receipt file:

```bash
python -m emodpy_workflow.scripts.download -f output/ReportHIVByAgeAndGender.csv -p ContainerPlatform -r RECEIPT_FILE
```

... where RECEIPT_FILE is the path of a receipt created by a prior **run** command. Output will be stored in the
directory containing the receipt.

---

## **Download multiple files per simulation**

Downloading more than one file from simulations is a small modification of downloading a single file. For downloading a
single file, see: [Download a file per simulation](#download-a-file-per-simulation).

To specify more than one file for download, one specifies **all** files together as the value of the **-f flag**, 
separating them by a **comma with no spaces**.

E.g. to download two files:

```bash
-f output/ReportHIVByAgeAndGender.csv,output/InsetChart.json
```
