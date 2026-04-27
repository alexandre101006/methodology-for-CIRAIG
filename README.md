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

The **EPA database** is a catalog of vehicles on the USA market (post 2010, for now). Our work focuses on the following data/columns, as they are the ones matching the expected level of detail :
* Model Year : the year the vehicle was manufactured
* Represented Test Veh make : the make of the vehicle
* Represented Test Veh Model : the model
* Equivalent Test Weight (ETW) (in lbs.) : net mass of the vehicle + mass of all liquids filled + 300 (estimated mass of passenger + cargo)

The ETW data is the center of our methodology. Therefore it needs to be understood right and some preliminary work concerns it.

The **nrcan** database is a catalog of the vehicles on the Canadian market (post 1995). Our work focuses on the following data/columns, as they are the ones matching the expected level of detail : 
* Model year : the year the vehicle was manufactured
* Make : the make of the vehicle
* Model : the model
* Vehicle class

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

* lower STEP : difference between the $ETW_{min}$ and the ETW step below
* upper STEP : difference between the $ETW_{max}$ and the ETW step above
* estim liquid mass : estimated mass of the fuel, oil, coolant... when all of those are filled
* 2,20462 is the conversion factor from lbs to kg

The estimated liquid mass is defined as following :
For MIN
* if $ETW_{min} < 4000 lbs$ : 50kg
* if 4000 lbs <= $ETW_{min} < 5500 lbs$ : 60kg
* if 5500 lbs <= $ETW_{min} : 70kg

For MAX
* if $ETW_{min} <= 4000 lbs$ : 50kg
* if 4000 lbs < $ETW_{min} <= 5500 lbs$ : 60kg
* if 5500 lbs < $ETW_{min} : 70kg

### Name matching EPA x nrcan database
Using a machine learning code (from sentence_transformers import SentenceTransformer, util) we are able to match EPA models with nrcan models.

That allow us to get a data base with :
YEAR - MAKE - MODEL - MIN - MAX - VEHICLE CLASS

Here are a few statistics regarding the matching :
* Out of the initial 12562 EPA vehicles, 10293 got a match
* 7.1% have a matching score below 0.5
* 12.9% have a matching score between 0.5 and 0.7
* 16.3% have a matching score between 0.7 and 0.8
* 15.4% have a matching score between 0.8 and 0.9
* 48.2% have a matching score above 0.9

### Matching EPA x SAAQ
As evoked previously, the make/model name in the SAAQ database are underspecified.
Therefore, all names in the EPA database are "stripped" and put under the same format (keeping the first five letters/word).

While merging on those underspecified/shorten names, for each YEAR x MAKE x MODEL combination in the SAAQ database we keep a count of the number of matching lines in the EPA database.

Finally, that count allows to define weights. If a SAAQ vehicle matched with multiple EPA vehicles, equal portions of that SAAQ vehicle will be assigned to all the possible EPA vehicles.
ex. for a 2010 TOYOT COROL having a weight matching Toyota Corolla and Corolla Matrix, we will assign half of that 2010 TOYOT COROL to the classic model and half to the Matrix.

This weight allocation is very important for the calculation of the estimated mean mass and number of each model.

$MEAN MASS = \frac{\sum_{YEAR x MAKE x MODEL}MASSE NETTE * weight}{\sum_{}weight}$

$NB = \sum_{YEAR x MAKE x MODEL}weights$

