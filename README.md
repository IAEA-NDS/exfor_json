
                           Nuclear Data Section
                     International Atomic Energy Agency

                                2005-2022


## EXFOR converted to JSON
This repository contains all EXFOR entries converted to JSON. The latest EXFOR master files are in [this repository](https://github.com/IAEA-NDS/exfor_master/). Note that this is still a working version, and it may change without any notifications.


### Download
You can download the repository from a terminal using:
```
git clone https://github.com/shinokumura/exfor_json.git
```

### EXFOR_to_JSON format
The JSON format (Schema) is not the final version and is still under discussions. The current schema looks as follows:
```
{
  bib             : {}
      title             : "string" 
      authors           : []  
      institutes        : [] 
      references        : [] 
      facility          : [] 
      history           : [] 
  subent-001      : {}
      extra_information : {}
            analysis	      : [ pointer: {} ]
            correction        : [ pointer: {} ]
            decay-data        : [ pointer: {} ]
            detector          : [ pointer: {} ]
            err-analys        : [ pointer: {} ]
            facility          : [ pointer: {} ]
            monitor           : [ pointer: {} ]
            sample            : [ pointer: {} ]
            status            : [ pointer: {} ]
  subent-002      : {}
      reaction          : [ pointer: {} ]
      extra_information : {}
            inc-source        : [ pointer: {} ]
            method            : [ pointer: {} ]
      common            : {}
            heads             : []
            units             : []
      data              : {}
            heads             : []
            units             : []
            data              : []
}
```



### Use index
Index of reactions are stored in json (index.json) and pickle (index.pickle) files.
If the PRODUCT (SF4) in REACTION filed is either of MASS, ELEM, or ELEM/MASS, you cannot know real products until reading DATA block. In such cases, following index stores the columns by replacing MASS, ELEM, or ELEM/MASS to the real products defined in the DATA block.
The format is as follows:
```
        id  entry subentry pointer  year       author  min_inc_en  max_inc_en points     target     process            sf4       residual   sf5      sf6   sf7    sf8   sf9
C0290-009-0  C0290      009      XX  1981    R.A.Cecil   3.370e-04   3.370e-04      1   13-AL-27  10-NE-20,X         0-NN-1         0-NN-1  None    DA/DE  None   None  None
E1773-008-0  E1773      008      XX  2002     T.Wakasa   3.450e+02   3.450e+02      1   20-CA-40         P,X         0-NN-1         0-NN-1  None    DA/DE  None   None  None
41128-002-2  41128      002       2  1993 V.A.Anufriev         NaN         NaN      0  98-CF-250       N,TOT           None           None  None      WID  None   None  None
```
Note: The minimum and maximum energy values are still subject to discussion due to an ambiguous use of EN-MIN, EN-MAX, EN..etc in EXFOR.


### Load reaction index from pickle file
Load Python pickle after unzip [``index.pickle.zip``](https://github.com/shinokumura/exfor_json/blob/main/index.pickle.zip). You can load and manipulate data using Pandas DataFrame for instance:

```
import pandas as pd
df = pd.read_pickle("index.pickle")

target = "79-AU-197"
process = "N,G"
quantity = "SIG"

df = df[
    (df_reaction.target == target.upper())
    & (df_reaction.process == process.upper())
    & (df_reaction.sf5.isnull())
    & (df_reaction.sf6 == quantity.upper())
    ]
print(df)
```

### Load reaction index from json file
Load JSON file after unzip [``index.json.zip``](https://github.com/shinokumura/exfor_json/blob/main/index.json.zip). You can load and manipulate data using Pandas DataFrame for instance:

```
import pandas as pd
df = pd.read_json("index.json")
print(df)
```

