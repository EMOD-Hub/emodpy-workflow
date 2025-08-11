# Install

## Software Prerequisites

- This guide assumes Python 3.9.X (3.9.13 or higher) (64-bit) is installed 
(https://www.python.org/downloads/release/python-3913/) in Windows or Linux
and assumes it is custom-installed into C:\Python39 in Windows or /c/Python39
in Linux. Downloads:
    - [exe](https://www.python.org/ftp/python/3.9.13/python-3.9.13-amd64.exe)
    - [tgz](https://www.python.org/ftp/python/3.9.13/Python-3.9.13.tgz)

- This guide further assumes a Linux-like command terminal is being used for Windows (e.g. git bash in Windows), not the
built-in Windows cmd.

---


If you have Python installed to a different directory, please update the Python interpreter path below to match your 
installation of Python.

## 1. Create the virtual environment
```
/c/Python39/python -m venv ~/environments/emodpy-workflow
```

## 2. Activate the virtual environment:

_Bash in Windows:_
```
source ~/environments/emodpy-workflow/Scripts/activate
```

_In Linux:_
```
source ~/environments/emodpy-workflow/bin/activate
```

## 3. Ensure pip is up-to-date:

```
python -m pip install pip --upgrade
```

## 4. Install emodpy-workflow:
```
python -m pip install emodpy-workflow --extra-index-url=https://packages.idmod.org/api/pypi/pypi-production/simple
```
