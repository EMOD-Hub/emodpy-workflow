# Learn how to create scenarios and do sweeps

TODO: the following is a good idea

- Add distribution of vaccine on STIDebut to vaccine/campaign.py 
- What about doing it on their 15th birthday instead?  Simulate an annual checkup

TODO: ensure the run command and sample count discussion at the end lines up with the prior tutorials

TODO: anything else at the end?

## **What is a scenario or sweep?**

A scenario (sweep) is a set of hyperparameters and values that will be applied to a frame. One can think of them as 
"what if" scenarios/simulations.

Hyperparameter values in a scenario are _arbitrary_. Every scenario has its own unique set of hyperparameters and values
that may or may not have any relation to other scenarios, depending on how a user wants to use them.

The hyperparameter values in a scenario are always applied _last_ to ensure they have final say in configuration.

In emodpy-workflow, scenarios are defined in a Python file with a specific format. This gives the user maximum 
flexibility in defining their scenarios while ensuring they can be understood by the **run** command. These files are
often called "sweep files".

## **Making scenarios**

The following is an exact requirement for the starting point of a sweep file to be used with a frame of choice with one 
blank scenario (with comments noting details of usage):

```python
frame_name = FILL_IN_NAME_OF_FRAME_TO_USE
# A sweep python file must contain a 'parameter_sets' attribute, which is a dict with keys being names of
# frames and values being dicts of param_name:value entries OR a generator of such dicts
parameter_sets = {
    # This key indicates the contained information is for building off the 'hiv_vaccine' frame
    frame_name: {
        # Each dict in 'sweeps' list is a set of param: value overrides to be applied. A scenario.
        # Note that the parameter lists are arbitrary. Each scenario can include as many or few parameters
        # as you wish.
        # One experiment will be created per entry in 'sweeps'
        'sweeps': [
            # Optional: A provided 'experiment_name' in a sweep entry will name the corresponding experiment. Default
            #  experiment name is the name of the frame.
            {}  # This is an entirely blank scenario entry
        ]
    }
}
```

To modify the above for practical use requires:

- filling in the name of the frame to use
- filling in one or more sets of hyperparameters to modify

The following is a basic functional example intended to be used with the sequence of tutorials in emodpy-workflow. It
creates five scenarios.

**Copy/paste** it into a file named **sweeps_vaccine.py** in your project directory.

```python
frame_name = 'hiv_vaccine'
parameter_sets = {
    frame_name: {
        'sweeps': [
            {'experiment_name': 'no_efficacy',          'vaccine_efficacy': 0.0},
            {'experiment_name': 'low_efficacy',         'vaccine_efficacy': 0.25},
            {'experiment_name': 'medium_efficacy',      'vaccine_efficacy': 0.5},
            {'experiment_name': 'medium_high_efficacy', 'vaccine_efficacy': 0.75},
            {'experiment_name': 'high_efficacy',        'vaccine_efficacy': 1.0},
        ]
    }
}
```

## **Using scenarios**

Sweep files are inputs to the **run** command. They are optional. Not specifying one is equivalent to running a frame
as-is (no changes).

When specified as an input to **run**, one experiment is created per 'sweeps' entry, each containing one simulation per
(calibrated) parameterization in the provided sample file.

The following command combines the above sweeps file that varies vaccine_efficacy with the previously generated
calibration samples.

Since there are five sweep entries and three samples, the command will generate:
one suite of five experiments of three simulations, or 1 * 5 * 3 = 15 simulations in total.

```bash
python -m emodpy_workflow.scripts.run -F hiv_vaccine -N vaccine_scenarios -o output -p ContainerPlatform -s samples.csv -S sweeps_vaccine.py
```

