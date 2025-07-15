# Installation

<a id="software-prerequisites"></a>
## Software Prerequisites

- This guide assumes Python 3.9.X (3.9.13 or higher) (64-bit) is installed 
(https://www.python.org/downloads/release/python-3913/) in Windows or Linux and assumes it is custom-installed into 
C:\Python39 in Windows or /c/Python39 in Linux.  (Downloads:  <a href="https://www.python.org/ftp/python/3.9.13/python-3.9.13-amd64.exe">exe</a>,  <a href="https://www.python.org/ftp/python/3.9.13/Python-3.9.13.tgz">tgz</a>)

- This guide further assumes a Linux-like command terminal is being used for Windows (e.g. git bash in Windows), not the
built-in Windows cmd.

---

<a id="create-and-activate-virtual-environment"></a>
## 1. Create and activate a clean virtual environment

If you have Python installed to a different directory, please update the Python interpreter path below to match your 
installation of Python.

### > Create the virtual environment
```
/c/Python39/python -m venv ~/environments/emodpy-workflow
```

### > Activate the virtual environment:


#### _Bash in Windows:_
```
source ~/environments/emodpy-workflow/Scripts/activate
```


#### _In Linux:_
```
source ~/environments/emodpy-workflow/bin/activate
```

### > Ensure pip is up-to-date:
```
python -m pip install pip --upgrade
```

<a id="obtain-and-install-emodpy-workflow"></a>
## 2. Obtain and install emodpy-workflow:
```
python -m pip install emodpy-workflow --extra-index-url=https://packages.idmod.org/api/pypi/pypi-production/simple
```
