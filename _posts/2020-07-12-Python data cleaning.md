---
layout:     post
title:      python data cleaning
subtitle:   
date:       2020-07-12
author:     neverset
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - python
    - pandas
---

## data wrangling
### missing value

    # a list with all missing value formats
    missing_value_formats = ["n.a.","?","NA","n/a", "na", "--"]
    df = pd.read_csv("employees.csv", na_values = missing_value_formats)   

### invalid data type

    def make_int(i):
        try:
            return int(i)
        except:
            return pd.np.nan

    # apply make_int function to the entire series using map
    df['Salary'] = df['Salary'].map(make_int)

### marking&removing missing values

    # check if there are missing values in dataframe
    print(df.isnull().values.any())
    # statistically list missing value
    print(df.isnull().sum())
    # notnull will return False for all NaN values
    null_filter = df['Gender'].notnull()
    # drop all rows with NaN values
    df.dropna(axis=0,inplace=True)
    # drop all rows with atleast one NaN
    new_df = df.dropna(axis = 0, how ='any')  
    # drop all rows with all NaN
    new_df = df.dropna(axis = 0, how ='all')

### filling missing values

    # replace na with constants
    df['Salary'].fillna(0, inplace=True)
    # or with replace function 
    df['Salary'].replace(to_replace = np.nan, value = 0,inplace=True)
    # replace na with value in previous row
    df['Salary'].fillna(method='pad', inplace=True)
    # replace na with value in next row
    df['Salary'].fillna(method='bfill', inplace=True)
    # interpolation
    df['Salary'].interpolate(method='linear', direction = 'forward', inplace=True)

## open source python libraries
### pandas profiling
pandas profiling provides EDA information about datas to be analysed

    import pandas as pd
    import numpy as np
    # create data 
    df = pd.DataFrame(np.random.randint(0,200,size=(15, 6)), columns=list('ABCDEF'))
    # run your report!
    df.profile_report()

### Dabl
data analysis library designed for data exploration and preprocessing of a machine learning project

    #pip install dabl
    data_clean = dabl.clean(data)
    dabl.plot(data, 'class')

### Dora
includes useful utilities for cleaning, transforming and visually exploring a data set, besides it has data versioning capabilities

    pip install Dora
    from Dora import Dora
    # Read the dataframe into the Dora format, specifying the target column in output
    dora = Dora(output = 'class', data = data)
    # Impute missing values
    dora.impute_missing_values()
    # Scale values
    dora.scale_input_values()
    # Save a specific version of the data
    dora.snapshot('cleaned_data')
    # Track changes to the data
    dora.logs

### Pretty pandas
create “pretty” summary tables for your data

    #pip install prettypandas
    from prettypandas import PrettyPandas
    # Convert the class to a numeric data type
    data['class'] = data['class'].astype('float64')
    # Summarise the data set using the Pandas groupby function
    df_grp = data.groupby('class').mean()
    # Add an overall average to the table
    PrettyPandas(df_grp).average()


## pandas data cleaning tips
### filter with query()

    #filter can be done like this
    df.loc[(df['tip']>6) & (df['total_bill']>=30)]
    # filter is more elegant with query
    df.query("tip>6 & total_bill>=30")
    # it support in and not in
    df.query("release_year in [2018, 2019]")

    # reference global variable name with @
    median_tip = df['tip'].median()
    display(df.query("tip>@median_tip").head())

    #parse python func
    def country_count(s):
        return s.split(',').__len__()
    df.query("release_year.isin([2018, 2019]) and country.apply(@country_count) > 5")

    # support Index and MultiIndex
    df.set_index('title').query("index.str.contains('king', case=False)")
    temp = df.set_index(['title', 'type'])
    temp.query("title.str.contains('king', case=False) and type == 'Movie'")


### sorting multiple columns

    df.sort_values(by=[‘total_bill’, ‘tip’], ascending=[True, False]).head()

### Use nsmallest() or nlargest()
check out data extract for records that have the smallest or largest values in a particular column

    df.nsmallest(5, 'total_bill')

### Customise describe()
summary stats for selected columns

    display(df.describe(include=['category'])) # categorical types
    display(df.describe(include=['number'])) # numerical types

### preprocess data to change type
after read data, the data type of all data is string. Before processing (such as logical comparison) it makes sense to assign type to them

    df = df.astype({"Open":'float',
                "High":'float',
                "Low":'float',
                "Close*":'float',
                "Adj Close**":'float',
                "Volume":'float'})

### Logical Comparisons wrapper
eq (equivalent to ==) — equals to
ne (equivalent to !=) — not equals to
le (equivalent to <=) — less than or equals to
lt (equivalent to <) — less than
ge (equivalent to >=) — greater than or equals to
gt (equivalent to >) — greater than

    df['Bool Price Increase'] = df['Close*'].gt(df['Open'])
    df['Bool Over Time Increase'] = df['Close*'].gt(df['Close*'].shift(-1))

### persisting data without csv
csv does nto persist data type of pandas
alternative:
* Pickle and to_pickle()
    #Pandas's to_pickle method
    df.to_pickle(path)  
* Parquet and to_parquet()
    #Pandas's to_parquet method
    df.to_parquet(path, engine, compression, index, partition_cols)
* Excel and to_excel()
    #exporting a dataframe to excel
    df.to_excel(excel_writer, sheet_name, many_other_parameters)
* HDF5 and to_hdf()
If the data are stored as table (PyTable) you can directly query the hdf store using store.select(key,where="A>0 or B<5")
    #exporting a dataframe to hdf
    df.to_hdf(path_or_buf, key, mode, complevel, complib, append ...)
* SQL and to_sql()
    #Set up sqlalchemy engine
    engine = create_engine(
        'mssql+pyodbc://user:pass@localhost/DB?driver=ODBC+Driver+13+for+SQL+server',
        isolation_level="REPEATABLE READ"
    )
    #connect to the DB
    connection = engine.connect()
    #exporting dataframe to SQL
    df.to_sql(name="test", con=connection)

### binning data
here are some methods to speed up the binning process
#### iterrows()

    %%timeit
    cat_list = []
    for index, row in mpg.iterrows():
        wt = row['weight']    
        cat = apply_weights(wt)
        cat_list.append(cat)
    #print(len(cat_list))
    mpg['Wt_Categories_iter'] = cat_list

#### apply()

    %%timeit
    mpg['wt_cat_apply'] = mpg.apply(lambda row: apply_weights(row['weight']), axis=1)

#### cut()

    %%timeit
    mpg['wt_cat_cut'] = pd.cut(np.array(mpg['weight']), 4, labels=["Light", "Medium", "Heavy", "Very heavy"])

### adding data to dataframe

#### concat
it takes much less time than append

    %%timeit
    df_concat = pd.concat([df_a, df_b, df_c], axis = 0)