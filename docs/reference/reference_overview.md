# Reference Overview
The following items provide reference information on topics specific to using emodpy-workflow:

## **Projects**

<p>
emodpy-workflow is organized around "projects". A "project" is a structured directory containing information related to 
model definition, inputs building, observational data, and scientific scenarios. In other words, it contains all data 
for a related set of model configurations and any output obtained for analysis.

**All commands must be run with a project directory as the current directory.**

The minimum required components of a project are detailed below.
</p>

<a id="manifest-py"></a>
### manifest.py

**&lt;project_dir&gt;/manifest.py** : This file specifies the pathing needed for the model executable to be run by the built-in 
commands as well as ingest form location for calibration. For EMOD-HIV, the required attributes (paths) that must be 
set are:

- **ingest_filename** : Path to a default ingest form to use for calibration of any frame in the project.
- **executable_path** : Path to the EMOD-HIV binary to use for all frames in the project.
- **schema_path** : Path to the schema.json file compatible with the specified EMOD-HIV binary.
- **asset_collection_of_container** : If the platform you intend to run simulations on requires the specification and 
use of a .sif file (for Singularity) then specify the path to an appropriate .sif file here. Otherwise, specify 
**None**.
- **post_processing_path** : The path of a dtk_post_process.py file to use with EMOD-HIV. "standard" means to use a 
built-in post-processing script and **None** means to skip post-processing. Post-processing is only required for 
calibration of EMOD-HIV.
- **pre_processing_path** : The path of a dtk_pre_process.py file to use with EMOD-HIV. "standard" means to use a 
built-in pre-processing script and **None** means to skip pre-processing. There is no current "standard" pre-processor 
for EMOD-HIV.
- **in_processing_path** : The path of a dtk_in_process.py file to use with EMOD-HIV. "standard" means to use a built-in
in-processing script and **None** means to skip in-processing. There is no current "standard" in-processor for 
EMOD-HIV.

<a id="bin-directory"></a>
### bin directory

**&lt;project_dir&gt;/bin** : This is a directory intended to contain model executables and any required files related to their 
basic execution. When running EMOD-HIV, it is automatically created upon simulation creation (as needed) and is the 
directory that your Eradication binary and schema will be automatically copied/installed into. It is also typically the 
directory for a user to put any needed container (.sif) files for execution.

<a id="frames-directory"></a>
### frames directory

**&lt;project_dir&gt;/frames** : This is a directory that contains the frames in a project, where each frame is a subdirectory. 
The frames directory will be created automatically as needed by the command new_frame.

<a id="idmtools-ini"></a>
### idmtools.ini

**&lt;project_dir&gt;/idmtools.ini** : This file configures the idmtools Platform object that manages the creation and running of simulations on 
compute resources. All commands that need to communicate with a compute resource (for execution, obtaining files, etc) 
accepts an idmtools.ini platform block name (e.g. ContainerPlatform) via argument to identify which resource to utilize.

The new_project command will automatically create one that can then be modified. Details of the file 
format can be found <a href="https://docs.idmod.org/projects/idmtools/en/latest/configuration.html">here</a>.

---

## **Commands**

**All commands are intended to be run in a project directory. This is the directory that directly contains your frames directory, manifest.py, and idmtools.ini file.**

All commands can be run via:

```bash
python -m emod_workflow.scripts.COMMAND_NAME_HERE <ARGUMENTS_HERE>
```

Information on command arguments and their usage is available via:

```bash
python -m emod_workflow.scripts.COMMAND_NAME_HERE --help
```

**new_project**

Creates a basic project directory with default files and settings. Platform-specific settings/files may still need to be
set/obtained. 

**new_frame**
 
Creates a new frame that imports and uses a defined emodpy-hiv country model, which can then be altered or extended.

**extend_frame**

Imports the input builders of an existing frame, which can then be altered or extended (without modifying the source 
frame).

**available_parameters**

Lists all currently defined and available hyperparameters of a specified frame. These are available for alteration by a
user during calibration and scenarios if they so choose.

**calibrate**

Calibrates a model specified in a frame to its reference data specified in a "calibration ingest form".

**resample**

Selects one or more parameter sets (samples) from a calibration process. These parameter sets are the model calibration.
They can be used later on in scenarios when using the "run" command.

**run**

Runs model simulations. Calibration samples and/or sweeps files can be provided as input as appropriate. 

**download**

Obtains specified output file(s) from previously run simulations and puts them into a structured local directory.

**plot_sims_with_reference**

Plots model output against reference data to aid in calibration.

---

## **Frames**

A frame is a standard input to emodpy-workflow commands that functions as an interface to model input definition, 
discovery, and execution.

In particular, a frame defines:
- The model to be used (EMOD-HIV)
- Functions that initialize the model inputs
- Functions that define how to build inputs after initialization
- Available hyperparameters and how they modify model input building
- Reference observational data (for calibration)

Frames are designed to make it simple to extend them via code reuse similar to class inheritance in object oriented 
design. This enables a project to contain a "family tree" of frames, automatically propagating updates from "parent" 
frames to their descendants for frame and scenario consistency.

The built-in commands "new_frame" and "extend_frame" are convenience methods for generating new frames for use. They are
not strictly necessary to create a frame (they can be "handmade"). A frame simply needs to have one attribute, 
**model**, in its __init__.py file, where the value of **model** is an object of a descendent class of IModel.

A sample EMOD HIV frame __init__.py generated by the "new_frame" command:

```python
# This frame built via command:
# python -m emodpy_workflow.scripts.new_frame

from emodpy_workflow.lib.models.emod_hiv import EMOD_HIV
from emodpy_workflow.lib.utils.runtime import load_manifest

# The manifest contains input file pathing information for the project
manifest = load_manifest()

# EMOD contains three main configuration objects: config, demographics, and campaign. The related information
# for generating these input objects is placed into concern-specific files in this directory.
from . import config
from . import demographics
from . import campaign

# 'model' is a required attribute of this file. All commands access frames by loading the 'model' attribute.
# The model attribute is assigned a model- and disease-specific object that contains all information regarding
# how to build the inputs for the model and generating its command line for execution.
model = EMOD_HIV(
    manifest=manifest,
    config_initializer=config.initialize_config,
    config_parameterizer=config.get_config_parameterized_calls,
    demographics_initializer=demographics.initialize_demographics,
    demographics_parameterizer=demographics.get_demographics_parameterized_calls,
    campaign_initializer=campaign.initialize_campaign,
    campaign_parameterizer=campaign.get_campaign_parameterized_calls,
    ingest_form_path=manifest.ingest_filename,
    build_reports=config.build_reports
)
```

---

## **Sweep Files**

A sweep file is a Python file that specifies sets of hyperparameter overrides, often referred to as "scenarios". 
Scenarios address specific scientific questions, typically (but not exclusively) related to predicting the outcome
of potential interventions and events.

These overrides are applied to simulation inputs building of specific frame(s) after any other parameter overrides 
(they have the highest precedence).

Sweep file format by example:

```python
# A sweep Python file must contain a 'parameter_sets' attribute, which is a dict with keys being names of
# frames and values being dicts of param_name:value entries OR a generator of such dicts
parameter_sets = {
    # This key indicates the contained information is for building off the 'baseline' frame
    'baseline': {
        # Each dict in 'sweeps' list is a set of param: value overrides to be applied. A scenario.
        # Note that the parameter lists are arbitrary. Each scenario can include as many or few parameters
        # as you wish.
        # One experiment will be created per entry in 'sweeps'
        'sweeps': [
            # Optional: A provided 'experiment_name' in a sweep entry will name the corresponding experiment. Default
            #  experiment name is the name of the frame.
            {'experiment_name': 'condom_maternal_higher', 'Condom_Transmission_Blocking_Probability': 0.9, 'Maternal_Infection_Transmission_Probability': 0.4},
            {'experiment_name': 'condom_maternal_lower', 'Condom_Transmission_Blocking_Probability': 0.7, 'Maternal_Infection_Transmission_Probability': 0.2},
            {'experiment_name': 'condom_higher', 'Condom_Transmission_Blocking_Probability': 0.9},
            {}  # This is a do nothing different, baseline scenario
        ]
    }
}
```
