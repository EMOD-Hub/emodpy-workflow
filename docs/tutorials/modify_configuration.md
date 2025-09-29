# Modify Configuration Parameters

## What are Configuration Parameters?

Configuration parameters are simulation-wide parameters that affect how EMOD runs at a fundamental level. These include
parameters related, but not limited to:

* model duration, timestep length, and when the model starts
* how a disease progresses and transmits
* whether agents can be born, age, and/or die
* output report specifications

## Modifying a Configuration Parameter

Configuration parameters are assigned values either in a country model or in a project frame, in one of two functions
(in both locations): `initialize_config` and `get_config_parameterized_calls`.

### Example

In this example we update the value of configuration parameter named **Base_Infectivity**, which alters how infectious
a disease is at a fundamental level (before any other modifiers apply).

#### Create a new project and baseline frame

```bash 
python -m emodpy_workflow.scripts.new_project -d configuration_tutorial
cd configuration_tutorial
python -m emodpy_workflow.scripts.new_frame --country Zambia --dest baseline
```

#### Extend the baseline frame for alteration

```bash 
python -m emodpy_workflow.scripts.extend_frame --source baseline --dest more_infectious
```

#### Edit the frame config.py

Here we edit the `initialize_config` function, opting to not make a hyperparameter for setting `Base_Infectivity`.

Edit file `frames/more_infectious/config.py` to have 10 times greater infectivity. It should contain the following:

```python  
from typing import List

from emod_api.schema_to_class import ReadOnlyDict
from emodpy_hiv.parameterized_call import ParameterizedCall
from emodpy_hiv.reporters.reporters import Reporters

from .. import baseline as source_frame


def initialize_config(manifest):
    config = source_frame.model.config_initializer(manifest=manifest)
    config.parameters.x_Base_Population = config.parameters.x_Base_Population
    config.parameters.Base_Infectivity = config.parameters.Base_Infectivity * 10  # Increasing infectivity here.
    return config


def get_config_parameterized_calls(config: ReadOnlyDict) -> List[ParameterizedCall]:
    parameterized_calls = source_frame.model.config_parameterizer(config=config)
    return parameterized_calls


def build_reports(reporters: Reporters):
    reporters = source_frame.model.build_reports(reporters)
    return reporters
```

#### Run EMOD

We will run the **baseline** frame and **more_infectious** frame:

```bash
python -m emodpy_workflow.scripts.run -f baseline,more_infectious -N ConfigUpdate -o config_output -p ContainerPlatform -d output/InsetChart.json
```

#### Compare InsetChart.json

Run:

```bash
python -m emodpy_hiv.plotting.plot_inset_chart_mean_compare config_output/ConfigUpdate--0/InsetChart/InsetChart_sample00000_run00001.json config_output/ConfigUpdate--1/InsetChart/InsetChart_sample00000_run00001.json
```

Blue below represents the **more_infectious** frame and green represents the frame **baseline**. As can be seen, highly
infectious HIV results in higher infection rates and mortality.

![image](../images/InsetChart_Compare--more_infectious.png)
