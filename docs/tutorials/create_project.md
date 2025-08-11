# Create Project

- a.Explain to the user that they always need to be working within the top level of a project
- b.Have them create a project and view what is included in a base project
- c.Point to reference data on projects
- d.Explain that they need a frame in order to run anything because that is where the simulation is defined
- e.Explain that they also need a default country model and point to the reference
- f.Have them create a frame and inspect the files that have been created.
- g.Explain that the frame is one of the places they can make changes to the simulations configuration.
- h.Point to reference data on frames


## General command format

```
python -m emodpy_workflow.scripts.XXX <arguments>
```

## new_project
The 'new_project' command is use to create a [new project directory](../reference/reference_overview.md#Projects).
It is assumed that all of the other commands will be executed in the root directory of the project.

```
python -m emodpy_workflow.scripts.new_project my_first_project
```

When the command is finished, you should something similar to the following:

```
$ ls
my_first_project
$ ls my_first_project
bin/
frames/
calibration
idmtools.ini
manifest.py
```

## new_frame

## extend_frame

## available_parameters

## calibrate

## download

## resample

## run

