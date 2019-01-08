
Dos:

- keep in mind default Snowflake timezone (especially using to_timestamp);

- convert pandas dataframe timestamp column to your timezone;

- use timeZoneDallas.localize(datetime) to create timezone aware datetime variable;

 

Don'ts:

- do not compare timestamp and date (apply to_date function if you need to)

 

Read below if you need more details.

Load all required libraries and create a connection
```python
import pandas as pd
import datetime
import pytz
#pip install --upgrade snowflake-sqlalchemy 
from snowflake.sqlalchemy import URL
from sqlalchemy import create_engine

engine = create_engine(URL(
    account = '***.us-east-1',
    user = '***',
    password = '***',
    database = 'testdb',
    schema = 'testschema',
    warehouse = 'TINY_WAREHOUSE',
    role='SYSADMIN',
    numpy=True,
))

connection = engine.connect()
```

<font color=red>**notes **</font>**PART 1**

<font color=red>**notes **</font>First of all, let's check what we have as default timezone and keep it mind:

```python
times_parameters = pd.read_sql_query(" SHOW PARAMETERS LIKE 'TIME%' ", engine)
times_parameters.reset_index(drop = True, inplace = True)
times_parameters.set_index('key', inplace = True)
print('Snowflake timezone is', times_parameters.loc['TIMEZONE', 'value'], 'set on', times_parameters.loc['TIMEZONE', 'level'], 'level.')
```
```
Snowflake timezone is America/New_York set on ACCOUNT level.
```

<font color=red>**notes **</font>Then let create our test table and populate with sample data. Although snowflake timezone is set as `America/New_York`, we do work in `America/Chicago` timezone, so we need to set timezone explicitly (`-0600`):

