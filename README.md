# Methodology to exploit the SAAQ database

## Merging with the EPA database
**What kind of data are we dealing with ?**

The **SAAQ database** gathers data on all the vehicles registered in Québec. Our work focuses on the following columns/data, as they are the ones matching the expected level of detail : 
* MARQ_VEH : the make of the vehicle
* MODEL_VEH : the model
* ANNEE_MOD : the year the vehicle was manufactured
* MASSE_NETTE : the net mass of the vehicle (kg)
* TYP_CARBU : energy source of the vehicle

The names of the makes and models are underspecified in the SAAQ database as we get the first five letters of the make/model (or the first word if it is shorter than five letters).

The **EPA database** is a catalog of vehicles on the USA market. Our work focuses on the following data/columns, as they are the ones matching the expected level of detail :
* Model Year : the year the vehicle was manufactured
* Represented Test Veh make : the make of the vehicle
* Represented Test Veh Model : the model
* Equivalent Test Weight (ETW) (in lbs.) : net mass of the vehicle + mass of all liquids filled + 300 (estimated mass of passenger + cargo)

The ETW data is the center of our methodology. Therefore it needs to be understood right and some preliminary work concerns it.

**What result are we expecting ?**
The expected result of the following works is to obtain a new database with the following information for each vehicle :
* YEAR
* MAKE/ MODEL
* VEHICLE CLASS
* MASS

### Preliminary work
To be able to exploit the ETW data, some pre-treatments are needed :

The assigned ETW of a vehicle is the EPA database is the closest step to its measured weight. The steps are the following :
* 125 lbs steps from 2000 to 4000 lbs [2000; 2125;...;3875;4000]
* 250 lbs step from 4000 to 5500 lbs [4000;4250;...,5250;5500]
* 500 lbs step above 5500 lbs

Each model can be assigned with multiple ETW in the EPA database as the specifications of a same model can make the weight vary (motorization, trasnmission,...).
Therefore, we are able to obtain a confidence interval for the measured mass of each model :

$[ETW_{min} - \frac{lower STEP}{2} ; ETW_{max} + \frac{upper STEP}{2}]$

Thus, we get a confidence interval for the net mass of each model :

$[min ; max] = [\frac{ETW_{min} - \frac{lower STEP}{2} - 300}{2,20462} - estim liquids mass ; \frac{ETW_{max} - \frac{upper STEP}{2} - 300}{2,20462}  - estim liquids mass]$

* lower STEP : difference between the ETW_{min} and the ETW step below
* upper STEP : difference between the ETW_{max} and the ETW step above
* estim liquid mass : estimated mass of the fuel, oil, coolant... when all of those are filled
* 2,20462 is the conversion factor from lbs to kg

The estimated liquid mass is defined as following :
* 50 kg for models with ETW_{max} < 4000 lbs
* 60 kg for models with 4000 lbs <= ETW_{max} < 5500 lbs
* 70 kg for models with 5500 lbs <= ETW_{max}
