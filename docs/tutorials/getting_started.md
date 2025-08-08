# Getting Started with EMOD-HIV and emodpy-workflow

## **Comparison with older HIV dtk-tools workflow**

If you have already been using EMOD-HIV via DtkTools, this overview will explain
what is the same and what is different.


Even though emodpy-workflow is designed to be a general workflow tool, its development was driven by the need to 
port the pre-existing EMOD-HIV workflow tools using dtk-tools to idmtools and emodpy. As such, every feature of 
the prior workflow has been replicated. There is nothing that could be done in dtk-tools that cannot be done with 
emodpy-workflow.

For those already familiar with the dtk-tools EMOD-HIV workfow, this section highlights the correspondence of its 
components and features with those of emodpy-workflow. **Bolded** items have changed the most.


| dtk-tools | emodpy-workflow |
| --- | --- |
| EMOD JSON input files | **country models** (from emodpy-hiv) |
| directories of templated JSON input files | **frames** |
| JSON KP tag parameters | **ParameterizedCalls** (from emodpy-hiv) |
| optim_script.py (calibration and configuration) | built-in calibrate command |
| ingest form (reference data and calibration configuration) | ingest form (still used, little changed) |
| calibration resampling via run_scenarios.py | built-in resample command |
| resampled_parameter_sets.csv | resampled_parameter_sets.csv (same format) |
| run_scenarios.py (scenario running tool) | built-in run and download commands |
| scenarios.csv | **sweeps.py file format** |

### Country Models

Country models are Python classes that define the default "baseline" behavior of an EMOD-HIV configuration. They often 
correspond to countries, but can be localized regions of interest. They contain all information (Python functions) 
needed to build a default set of EMOD-HIV inputs (config, demographics, campaign).

Country models are standard **Python code** and live in the emodpy-hiv repository. Usage of country models in a project
is the domain of **frames**.

### Frames

Frames are a construct in emodpy-workflow that replace directories of KP-tagged (templated) JSON files in a project. A 
frame is the central point where all Python code for building EMOD-HIV config, demographics, and campaign inputs is
found by emodpy-workflow commands. Frames can **import** and **extend** other frames to eliminate inconsistencies. With
JSON, one had to copy and modify files, creating duplicate information that could desynchronize.

For example, one could have a **baseline** frame and several **scenario** frames that import the baseline and then add 
campaign interventions. **Updating any frame automatically updates all dependent frames**.

### Parameterized Calls

ParameterizedCalls are Python objects that directly replace the functionality of JSON KP tags. ParameterizedCall 
objects let a user define a "hyperparameter" (an arbitrary string name) that connects to an arbitrary point in the 
EMOD-HIV input building code to allow modification during calibration and scenarios. Users can add contextual labels to
ParameterizedCalls to distinguish very specific values as could be done with KP tags.

e.g., instead of this KP-tagged parameter name:

```
Society__KP_Central.TRANSITORY.Relationship_Parameters.Condom_Usage_Probability.Max 
```

... one could create a ParameterizedCall hyperparameter named the following:

```
condom_usage_max--COMMERCIAL-Central
```

... which uses a contextual label of "COMMERCIAL-Central"

One can also tie multiple values together by reusing (in Python code) a hyperparameter name with identical context.

e.g., one could define and reuse a ParameterizedCall hyperparameter that connects to each model node individually

```
condom_usage_max--COMMERCIAL-ALL-NODES
```

... which would then apply any value changes to all nodes for COMMERCIAL relationships.

Each country model defines a starting set of ParameterizedCalls and hyperparameters for use. **Additional ones can be 
added to project frames**.

### Ingest forms

Ingest forms are still used for specifying observational reference data and configuring analyzers used in the
calibration process. There is virtually no difference in its usage.

The **only** sheet in an older ingest form that would need updating for use is the  **Model Parameters** sheet:

- **map_to** column can be present but now **ignored**
- **name** column values will need updating to new hyperparameter names from old KP-tag parameter names.

### Sweep Files

Scenario csv files (e.g. scenarios.csv) has been replaced with a new sweep Python file format. Instead of rows of
parameter overrides with parameter-named columns, one defines lists of hyperparameter/value dictionary overrides.
Because the file is Python instead of csv, one can utilize code to simplify the generation of complex or repetitive
sets of overrides.

e.g. instead of:
```
Scenario,Campaign,Base_Infectivity
baseline,campaign.json,
high_infectivity,campaign.json,0.1
low_infectivity,campaign.json,0.001
```

... one would define:
```
parameter_sets = {
    #  This indicates experiments start with the baseline frame
    'baseline': {
        'sweeps': [
            {},  # This runs with no modifications over the baseline and is auto-named "baseline"
            {'experiment_name': 'higher_infectivity', 'Base_Infectivity': 0.1},
            {'experiment_name': 'lower_infectivity', 'Base_Infectivity': 0.001},
        ]
    }
}
```

TODO: Do we need the format of this file (as a list of links to tutorials) given the base template of the docs having
a header with links to everything?

## Installation

Please see the [installation guide](../installation.md) for creating a virtual environment and installing emodpy-workflow in it.


## 1. Learn about the project related commands

[This tutorial](project_related_commands.md) will introduce you to the set of commands that you can do with emodpy-workflow.
The other tutorials will do more with these commands and show you more on when and why to use them.

## 2. Learn about model input building
[This tutorial](input_building_process.md) will teach you how EMOD inputs are built in emodpy-workflow.

## 2. Learn about frames and model input building
[This tutorial](using_frames.md) will teach you what a frame is, the basics of making them, and how emodpy-workflow uses
them to define and build model inputs.

## 3. Learn to make hyperparameters with ParameterizedCall

[This tutorial](using_parameterized_calls.md) will teach you how hyperparameters are defined by ParameterizedCall 
objects and how to make your own to enable dynamic model changes.

## 4. Learn how to run EMOD

[This tutorial](running_emod.md) will get you running your first EMOD simulations.  It will combine the above learnings
to show how easy it is to change parameters, make changes to the campaign, and plot data.

## 5. Brief introduction to Calibration

In [this tutorial](calibration_intro.md), you will do your first simple calibration with emodpy-workflow.
You will learn how to specify reference data and the hyperparameters to calibrate. You will also learn how to resample
a calibration process (select calibrated parameterizations). However, it is not intended to teach the art/science of 
calibration.

# 5.5 Using a Calibration (Parameterizations)

[This tutorial](using_a_calibration.md) will show you how to use a calibration as input to configure and run EMOD. 

## 6. Learn how to create scenarios and do sweeps

[This tutorial](scenarios_and_sweeps.md) will teach you how to define scenarios and hyperparameter sweeps in a Python
sweep file and use them with the run command.

## 7. Learn to configure EMOD reports
[This tutorial](using_reporters.md) will teach you how to alter what and how EMOD generates output.

## 8. Learn how to change the Cascade of Care

[This tutorial](changing_the_cascade_of_care.md) ... TODO
