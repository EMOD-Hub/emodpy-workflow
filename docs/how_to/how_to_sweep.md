# Sweep parameters

## Create a scenario or sweep

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

## Create a set of sweeps

First, please see [Create a scenario or sweep](#create-a-scenario-or-sweep). Then use the following for sweeps.py instead, which 
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

## Use a sweep to explore hyperparameter sensitivity of a frame

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

## Use a sweep to explore internal variability of a frame

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

## Run a sweep with a calibrated frame

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

