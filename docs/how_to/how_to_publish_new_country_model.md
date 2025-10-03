## Public a new country model

To publish a new country model, you can follow the steps below. This example uses the `Eswatini` country model in 
[Modify Campaign 3: New Country Model](../tutorials/modify_campaign_3_new_country_model.md) tutorial.

1. Move `eswatini.py` to `/emodpy-hiv/emodpy_hiv/countries/eswatini/eswatini.py`

2. Add a `__init__.py` file under `emodpy_hiv/countries/eswatini` folder with:
```python
from .eswatini import Eswatini # noqa: F401
```

3. Add this line to `__init__.py` file under `emodpy_hiv/countries` folder:
```python
from emodpy_hiv.countries.eswatini import Eswatini # noqa: F401
```

4. Go to `emodpy_workflow`, reinstall `emodpy-hiv` with the new change, verify you can use the new country `Eswatini` model 
by running:
```bash
python -m emodpy_workflow.scripts.new_frame --country Eswatini --dest Eswatini
python -m emodpy_workflow.scripts.available_parameters -F Eswatini
python -m emodpy_workflow.scripts.run -N Eswatini -F Eswatini -o results/Eswatini -p ContainerPlatform
```
You should not need to edit the `campaign.py`, `config.py` or `demographics.py` file in the frame anymore and no more 
warning message about country model not found when creating a new frame with `Eswatini` country model.
