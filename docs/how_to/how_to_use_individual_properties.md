# Individual Properties (IndividualProperty)

Individual properties can be defined in a country model or a project frame. The examples here assume they are in a 
frame. More details regarding Individual properties can be found in the 
[modify demographics tutorial](../tutorials/modify_demographics.md) and in the 
[IndividualProperties reference documentation](https://docs.idmod.org/projects/emodpy-hiv/en/latest/emod/model-properties.html).

## How to create a new IndividualProperty

IndividualProperties are part of EMOD demographics, so they are added to `demographics.py` in a frame.

For example, to add a property:

* Named **ReceivedVaccine** 
* That can have values of **Yes** or **No**
* With a starting population distribution of 100% **No**
* That applies to all simulation nodes

... add the following to function **initialize_demographics** in `demographics.py` in your chosen frame:

```python 
property = 'ReceivedVaccine'
values = ['Yes', 'No']
distribution = [0.0, 1.0]
demographics._add_or_update_individual_property_distribution(property_name=property, values=values,
                                                             distribution=distribution)

return demographics  # This is already in demographics.py . Added to example for location reference.
```

## How to modify an IndividualProperty via an event

Altering an IndividualProperty value for a model agent requires editing the campaign configuration by adding a
PropertyValueChanger intervention. A typical setup involves a triggered event distribution.

The location of the appropriate code can either be in a country model or in a project frame ``campaign.py`` file, 
usually in function ``get_campaign_parameterized_calls``.

Assume the IndividualProperty named **ReceivedVaccine** should be updated to a value of **Yes** when an agent is 
vaccinated, that the following exists in a frame ``campaign.py``:

```python 
def add_hiv_vaccine(campaign: api_campaign,
                    vaccine_efficacy: float = 1.0,
                    node_ids: List[int] = None):
    from emodpy.campaign.distributor import add_intervention_triggered
    from emodpy.campaign.individual_intervention import ControlledVaccine
    from emodpy.campaign.waning_config import Constant

    hiv_vaccine = ControlledVaccine(campaign=campaign,
                                    waning_config=Constant(constant_effect=vaccine_efficacy))
    add_intervention_triggered(campaign=campaign,
                               intervention_list=[hiv_vaccine],
                               triggers_list=["STIDebut"],
                               start_year=2026,
                               node_ids=node_ids)
    return campaign

def get_campaign_parameterized_calls(campaign: api_campaign) -> List[ParameterizedCall]:
    parameterized_calls = country_model.get_campaign_parameterized_calls(campaign=campaign)
    pc = ParameterizedCall(func=add_hiv_vaccine)
    parameterized_calls.append(pc)    
    return parameterized_calls
```

We add a new distribution trigger to the **ControlledVaccine** such that when it is applied, a new signal named
**Vaccinated** is sent. We add a single listening event that will update the IndividualProperty accordingly.

```python 
def add_hiv_vaccine(campaign: api_campaign,
                    vaccine_efficacy: float = 1.0,
                    node_ids: List[int] = None):
    vaccination_trigger = "Vaccinated"
    hiv_vaccine = ControlledVaccine(campaign=campaign,
                                    waning_config=Constant(constant_effect=vaccine_efficacy),
                                    distributed_event_trigger=vaccination_trigger)
    add_intervention_triggered(campaign=campaign,
                               intervention_list=[hiv_vaccine],
                               triggers_list=["STIDebut"],
                               start_year=2026,
                               node_ids=node_ids)

    # Event listening for "Vaccinated" trigger. Updates ReceivedVaccine IP for individual to Yes.                               
    ip_update = PropertyValueChanger(target_property_key="ReceivedVaccine", target_property_value="Yes")
    add_intervention_triggered(campaign=campaign,
                               intervention_list=[ip_update],
                               triggers_list=[vaccination_trigger],
                               start_year=2026,
                               node_ids=node_ids)

    return campaign

def get_campaign_parameterized_calls(campaign: api_campaign) -> List[ParameterizedCall]:
    parameterized_calls = country_model.get_campaign_parameterized_calls(campaign=campaign)
    pc = ParameterizedCall(func=add_hiv_vaccine)
    parameterized_calls.append(pc)    
    return parameterized_calls
```
