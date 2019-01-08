
Dos:

- keep in mind default Snowflake timezone (especially using to_timestamp);

- convert pandas dataframe timestamp column to your timezone;

- use timeZoneDallas.localize(datetime) to create timezone aware datetime variable;

Don'ts:

- do not compare timestamp and date (apply to_date function if you need to)

Read below if you need more details.


First of all, load all required libraries and create a connection:
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

Now we are ready ro check what we have as default timezone and keep it mind:
```python
times_parameters = pd.read_sql_query(" SHOW PARAMETERS LIKE 'TIME%' ", engine)
times_parameters.reset_index(drop = True, inplace = True)
times_parameters.set_index('key', inplace = True)
print('Snowflake timezone is', times_parameters.loc['TIMEZONE', 'value'], 'set on', times_parameters.loc['TIMEZONE', 'level'], 'level.')
```
```
Snowflake timezone is America/New_York set on ACCOUNT level.
```

Then let create our test table and populate with sample data. Although snowflake timezone is set as `America/New_York`, we do work in `America/Chicago` timezone, so we need to set timezone explicitly (`-0600`):
```python
query = ''' 
create or replace table times as 
(
    select dateadd(minutes, 15*0, to_timestamp_tz({timest})) as cret_dt
    union select dateadd(minutes, 15*1, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*2, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*3, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*4, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*5, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*6, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*7, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*8, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*9, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*10, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*11, to_timestamp_tz({timest}))
    union select dateadd(minutes, 15*12, to_timestamp_tz({timest}))
)
'''
query = query.format(timest = "'2018-12-06 22:12:17-0600'")
query_result = pd.read_sql(query.replace('\n', ' '), engine)
```

And finally query our table and keep the result in dataframe:
```python
query = ''' 
select cret_dt as cret_dt, to_date(cret_dt) as cret_dt_date_for_control 
from times
order by cret_dt;
'''
times = pd.read_sql(query.replace('\n', ' '), engine)
```
