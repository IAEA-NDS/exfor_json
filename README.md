
                           Nuclear Data Section
                     International Atomic Energy Agency

                                2005-2022


## EXFOR converted to JSON
This repository contains all EXFOR entries converted to JSON. The latest EXFOR master files are in [exfor_master repository](https://github.com/IAEA-NDS/exfor_master/). Note that this is still a working version, and it may change without any notifications. Current data is created using v20230315.0 in [exfor_master](https://github.com/IAEA-NDS/exfor_master/).


### Download
You can download the repository from a terminal using:
```
git clone https://github.com/IAEA-NDS/exfor_json.git
```

### EXFOR_to_JSON format
The JSON format (Schema) is not the final version and still under discussions. The current schema looks as follows:
```
{
  entry           : "string" 
  last_updated    : "string" 
  number_of_rev   : "string"
  histories       : []
  bib_record            : {}
      title             : "string" 
      authors           : []  
      institutes        : [] 
      references        : [] 
      facility          : [] 
      history           : []
  experimental_conditions : {}
    001           : {}
      analysis	        : [ pointer: {} ]
      correction        : [ pointer: {} ]
      decay-data        : [ pointer: {} ]
      detector          : [ pointer: {} ]
      err-analys        : [ pointer: {} ]
      facility          : [ pointer: {} ]
      monitor           : [ pointer: {} ]
      sample            : [ pointer: {} ]
      status            : [ pointer: {} ]
    002           : {}
  data_tables      : {}
    001          : {}
      common            : {}
          heads             : []
          units             : []
    002          : {}
      reaction          : [ pointer: {} ]
      common            : {}
          heads             : []
          units             : []
          data              : []
      data              : {}
          heads             : []
          units             : []
          data              : []
}
```


### Use EXFOR_to_JSON format
A symple example of plotting the data in Entry #12898, Subentry #002 is as follows:
```
import json
import pandas as pd
import matplotlib.pyplot as plt

f = open("json/128/12898.json")
data = json.load(f)

entry_data = data["12898-002"]["data"]

## Header
print("data headers:   ", entry_data["heads"])
## Unit
print("data units:     ", entry_data["units"])

## Load to Python dictionary
dict = {entry_data["heads"][i]: entry_data["data"][i]  for i in range(len(entry_data["heads"]))}

## Load dictionary to pd.DataFrame
df = pd.DataFrame(dict, index=None)
df["dy"] = df["DATA      2"] * df["ERR-T     2"]/100
print(df)

df.plot(x ="EN", y="DATA      2", yerr='dy', kind="scatter")
plt.show()
```


