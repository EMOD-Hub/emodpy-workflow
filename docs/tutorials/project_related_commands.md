# Learn about the project related commands

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

