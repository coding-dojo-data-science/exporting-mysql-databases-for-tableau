# Export MySQL Database for Tableau

## Instructions:

- Update the variables in the "🎛️ Update These Variables" section below, and then click on the `Kernel` menu, and select `Restart and Run All.`
- Pay attention to the print 

## 🎛️ Update These Variables

- You must update the following variables:
    - `DB_NAME`: Then name of the database. (Most likely "movies").
    - `MYSQL_LOGIN`: the filepath to the json file with your mysql username and password. 
        - Note: if you have not saved your mysql credentials to a json file in your .secret folder yet, we strongly suggest you do so now. 
        - Change `USER_KEY` to be the correct key from your json file with your user name.
        - Change `PASSWORD_KEY` to be the the correct key from your json file with your password.
        
        
- (Optional) You can change where the csv file will be saved by changing the `folder` variable.


```python
## UPDATE THESE VARIABLES TO MATCH YOUR OWN PC/DATABASE
# MySQL Database to export 
DB_NAME = "movies"

# Json file with mysql login credentials
MYSQL_LOGIN = "/Users/codingdojo/.secret/mysql.json"
USER_KEY = "user"
PASSWORD_KEY = "password"

## (Optional) - Change folder
folder = "Data-for-Tableau/"
```


```python
######## CODE TO TEST LOGIN CREDENTIALS
import os, json
os.makedirs(folder, exist_ok=True)

with open(MYSQL_LOGIN) as f:
	login = json.load(f)

if (USER_KEY not in login):
    raise Exception(f"[!] The json file did not have a {USER_KEY} key.")
    
if (PASSWORD_KEY not in login):
    raise Exception(f"[!] The json file did not have a {PASSWORD_KEY} key.")
    
```

## Run All Below!


```python

```


```python
# !pip install pymysql
```


```python
import pandas as pd
import os
import numpy as np

from sqlalchemy import create_engine
from sqlalchemy_utils import create_database, database_exists

import pymysql
pymysql.install_as_MySQLdb()
```


```python
connection = f"mysql+pymysql://{login[USER_KEY]}:{login[PASSWORD_KEY]}@localhost/{DB_NAME}"
engine = create_engine(connection)

if database_exists(engine.url):
    print(f"[i] Database {DB_NAME} found.")
else:
    raise Exception(f'[!] Database {DB_NAME} does not exist.')
```


```python
q  = """SHOW TABLES;"""
tables = pd.read_sql(q, engine)
tables
```


```python
table_names = tables[f'Tables_in_{DB_NAME}'].to_list()
table_names
```


```python
# Empty containers for new filenames and error messages
errors = {}
new_files = []

dashes = '---'*25
print(dashes,f"    EXPORTING DATABASE ({DB_NAME}) to '{folder}'", 
      dashes, sep='\n')


# Loop through all tables to export
for table in table_names:
    
    try:
        ## Get all data for table and save to csv
        temp = pd.read_sql(f"SELECT * FROM {table}", engine )
        fname = folder+f"{table}.csv"
        temp.to_csv(fname,index=False)
        
        # Save filename and print message
        new_files.append(fname)
        print(f"  - Exported {table} to '{fname}'")

    except Exception as e:
        # Save error message
        errors[table] = e
        print(f"  - [!] Error with '{table}' table")
        
```

### Errors


```python
## if errors, print out details
if len(errors) > 0:
    print('\n\n[!] ERRORS FOUND DURING EXPORT:')
    for k, v in errors.keys():
        print(f"  - Error for table {k}:   {e}")
        
else:
    print('[i]  No errors. :-)')
```

### Final Preview


```python
## Print preview of exported files.
for file in new_files:
    temp_df = pd.read_csv(file)
    print(dashes, f"[i] Preview of {file}:", dashes, sep='\n')

    display(temp_df.head(), temp_df.tail())
```
