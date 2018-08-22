.. pyreadstat documentation master file, created by
   sphinx-quickstart on Wed May 30 17:03:20 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to pyreadstat's documentation!
======================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:
   
pyreadstat
============

A python module to read sas (sas7bdat, xport), spps (sav, zsav, por) and stata (dta) data files into pandas dataframes.
<br> 

This module is a wrapper around the excellent [Readstat](https://github.com/WizardMac/ReadStat) C library by 
[Evan Miller](https://www.evanmiller.org/). Readstat is the library used in the back of the R library 
[Haven](https://github.com/tidyverse/haven), 
meaning pyreadstat is a python equivalent to R Haven (but writing files is currently not supported.)

Detailed documentation on all available methods is in the 
[Module documentation](https://ofajardo.github.io/pyreadstat_documentation/_build/html/index.html)


**DISCLAIMER** 

**Pyreadstat is not a validated package. The results may have inaccuracies deriving from the fact most of the data formats
are not open. Do not use it for critical tasks such as reporting to the authorities. Pyreadstat is not meant to replace
the original applications in this regard and for that reason writing is not supported.**  


Motivation
==========

The original motivation came from reading sas7bdat files in python. That is already possible using either the (pure
python) package [sas7bdat](https://pypi.org/project/sas7bdat/) or the (cythonized) method 
[read_sas](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.read_sas.html)
from pandas. However, those methods are slow (important if you want to read several large files), do not give you
the possibility to recover variable labels (only variable names), do not give the possibility to recover value labels (stored in 
the file itself in the case of spss or stata, or in catalog files in sas), convert both dates and datetime variables to datetime,
you have to specify the encoding otherwise in python 3 instead of strings you get bytes and do not offer the possibility to read
only the headers.
This package corrects those problems. 

**1. Good Performance:** Here a comparison of reading a 190 Mb sas7dat file with different methods. As you can see
pyreadstat is the fastest for python and matches the speeds of R Haven.

| Method | time  | 
| :----- | :-----------------: | 
| Python 3 - sas7bdat | 6 min | 
| Python 3- pandas | 42 s | 
| Python 3- pyreadstat | 7 s  | 
| R - Haven | 7 s | 

**2. Reading Variable Labels** sas7bdat and pandas.read_sas gives you the numerical data with column names and do not
 extract the variable labels. Pyreadstat also gives you the numerical data as a pandas data frame, but also gives you
 a metadata object that contains the variable labels for reference. As pandas dataframe cannot handle both variable names and labels, 
 you as user can take the decision of replacing the column names by labels if you want.
 
**3. Reading Value Labels** Neither sas7bdat and pandas.read_sas gives the possibility to read sas7bcat catalog files.
Pyreadstat can do that and also extract value labels from SPSS and STATA files. As pandas dataframes cannot handle value
labels, you as user will have to take the decision wether to use those values or not.

**4. Reading dates and datetimes** sas7bdat and pandas.read_sas convert both date and datetime variables into datetime.
That means if you have a date such a '01-01-2018' it will be transformed to '01-01-2018 00:00:00' (it always inserts a 
time), making it impossible
to know looking only at the data if the variable was originally a datetime (if it had a time) or not. 
Pyreadstat transforms dates to dates and datetimes to datetimes, so that you have a better correspondence with the original
data. However, it is possible to keep the original pandas behavior and get always datetimes.

**5. Encoding** On python 3, pandas.read_sas reads all strings as bytes. If you want strings you have to specify the encoding manually.
pyreadstat read strings as str. Thas is possible because readstat guesses correctly the original encoding and translates 
to utf-8, so that you don't have to care about that anymore. However it is still possible to manually set the encoding.

**6. Read only headers** sas7bdat and pandas.read_sas read all the data, no possibility to read only the headers. The
same with R Haven. Sometimes you want to take a quick look to many (sas) files looking for the datasets that contain
some specific columns, and you want to do it quick. This package offers the possibility to read only the metadata making 
it possible a very fast metadata scraping.

Dependencies
=============

The module depends on pandas, which you normally have installed if you got Anaconda (highly recommended.) Otherwise you
will have to install it manually before using pyreadstat.

In order to compile from source, you will need a C compiler (see installation). If you want to do changes to the
cython source code, you will need cython.

Installation
============

### From a pre-compiled python wheel

In this repository, look in the folder dist. If there is a wheel (.whl file) compatible for your python version and 
operating system, download it and do:

```
pip install pyreadstat-0.1.0-cp36-cp36m-linux_x86_64.whl
```

the example file pyreadstat-0.1.0-cp36-cp36m-linux_x86_64.whl can be a different one depending on your python version and system.

cp36-cp36m-linux_x86_64 means (C)Python 3.6 linux (Red Hat 7.5) 32/64 bits and is good for BEE. We also provide wheels
for python 3.7 and 3.5 for this operating system.

cp36-cp36m-win_amd64.whl means (C) Python 3.6 windows 64 bits and it has been tested both on win 7 and win 10 standard
roche computers, with Anaconda Python installed (it has not been tested with plain python). We also provide wheels for
python 3.5 and 3.7 on windows.

If there is not a suitable wheel for your system, you have to compile the package from source (distribution).

### From source distribution

From this repository, in the folder dist, download the file pyreadstat-x.x.x.tar.gz where x.x.x is the version and do:

```python
pip install pyreadstat-x.x.x.tar.gz
```

If you don't have admin privileges on the machine (for example on BEE) do:

```python
pip install pyreadstat-x.x.x.tar.gz --user
```

You need a working C compiler.

### From the latest sources

Download or clone the repo, open a command window and type:

```
python3 setup.py install
```

If you don't have admin privileges to the machine (for example on Bee) do:

```
python3 setup.py install --user
```

You need a working C compiler.

Compiling on linux is very easy, but on windows is a bit more challenging. 
Some instructions are found [here](https://github.roche.com/MDH-Tools/pyreadstat/blob/master/windows_compilation.md)

Basic Usage
===========

Pass the path to a file to any of the functions provided by pyreadstat. It will return a pandas data frame and a metadata
object. <br>
The dataframe uses the column names. The metadata object contains the column names, column labels, number_rows, 
number_columns, file label
(if any), file encoding (if applicable), notes and objects about value labels (if present). Be aware that file_label and
file_encoding may be None, not all columns may have labels, notes may not be present and there may be no value labels.

For example, in order to read a sas7bdat file:

```python
import pyreadstat

df, meta = pyreadstat.read_sas7bdat('/path/to/a/file.sas7bdat')

# done! let's see what we got
print(df.head())
print(meta.colum_names)
print(meta.column_labels)
print(meta.number_rows)
print(meta.number_columns)
print(meta.file_label)
print(meta.file_encoding)
```

You can replace the column names by column labels very easily (but check first that all columns have distinct labels!):

```python
# replace column names with column labels
df.columns = meta.column_labels
# to go back to column names
df.columns = meta.column_names
```

Here there is a relation of all functions available. 
You can also check the [Module documentation](https://ofajardo.github.io/pyreadstat_documentation/_build/html/index.html).

| Function in this package | Purpose |
| ------------------- | ----------- |
| read_sas7dat        | read SAS sas7bdat files |
| read_xport          | read SAS Xport (XPT) files |
| read_sas7bcat       | read SAS catalog files |
| read_dta            | read STATA dta files |
| read_sav            | read SPSS sav and zsav files  |
| read_por            | read SPSS por files  |
| set_catalog_to_sas  | enrich sas dataframe with catalog formats |
| set_value_labels    | replace values by their labels |

## Reading only the headers

All functions accept a keyword argument "metadataonly" which by default is False. If True, then no data will be read, 
but still both the metadata and the dataframe will be returned. The metadata will contain all fields as usual, but
the dataframe will be emtpy, although with the correct columns names. Sometimes number_rows may be None if it was not
possible to determine the number of rows without reading the data.

```python
import pyreadstat

df, meta = pyreadstat.read_sas7bdat('/path/to/a/file.sas7bdat', metadataonly=True)
```

## Reading value labels

For sas7bdat files, value labels are stored in separated sas7bcat files. You can use them in combination with the sas7bdat
or read them separately.

If you want to read them in combination with the sas7bdat files, pass the path to the sas7bcat files to the read_sas7bdat
function. The original values will be replaced by the values in the catalog.

```python
import pyreadstat

# formats_as_category is by default True, and it means the replaced values will be transformed to a pandas category column.
df, meta = pyreadstat.read_sas7bdat('/path/to/a/file.sas7bdat', catalog_file='/path/to/a/file.sas7bcat', formats_as_category=True)
```

If you prefer to read the sas7bcat file separately, you can apply the formats later with the function set_catalog_to_sas.
In this way you can have two copies of the dataframe, one with catalog and one without.

```python
import pyreadstat

# this df will have the original values
df, meta = pyreadstat.read_sas7bdat('/path/to/a/file.sas7bdat')
# read_sas7bdat returns an emtpy data frame and the catalog
df_empty, catalog = pyreadstat.read_sas7bdat('/path/to/a/file.sas7bcat')
# enrich the dataframe with the catalog
# formats_as_category is by default True, and it means the replaced values will be transformed to a pandas category column.
df_enriched, meta_enriched = pyreadstat.set_catalog_to_sas(df, meta, catalog, formats_as_category=True)
```

For SPSS and STATA files, the value labels are included in the files. You can choose to replace the values by the labels
when reading the file using the option apply_value_formats, ...

```python
import pyreadstat

# apply_value_formats is by default False, so you have to set it to True manually if you want the labels
# formats_as_category is by default True, and it means the replaced values will be transformed to a pandas category column.
df, meta = pyreadstat.read_sav("/path/to/sav/file.sav", apply_value_formats=True, formats_as_category=True)
```

... or to do it later with the function set_value_labels:

```python
import pyreadstat

# This time no value labels.
df, meta = pyreadstat.read_sav("/path/to/sav/file.sav", apply_value_formats=False)
# now let's add them to a second copy
df_enriched = pyreadstat.set_value_labels(df, meta, formats_as_category=True)
```


Other options
=============

You can set the encoding of the original file manually. The encoding must be a [iconv-compatible encoding](https://gist.github.com/hakre/4188459) 

```python
import pyreadstat

df, meta = pyreadstat.read_sas7bdat('/path/to/a/file.sas7bdat', encoding="LATIN1")
```

You can preserve the original pandas behavior regarding dates (meaning dates are converted to pandas datetime) with the
dates_as_pandas_datetime option

```python
import pyreadstat

df, meta = pyreadstat.read_sas7bdat('/path/to/a/file.sas7bdat', dates_as_pandas_datetime=True)
```

For more information, please check the [Module documentation](https://ofajardo.github.io/pyreadstat_documentation/_build/html/index.html).


Metadata Object Description
===========================

Each parsing function returns a metadata object in addition to a pandas dataframe. That
object contains the following fields:

  * notes: notes or documents (text annotations) attached to the file if any (spss and stata).
  * column_names : a list with the names of the columns.
  * column_labels : a list with the column labels, if any.
  * file_encoding : a string with the file encoding, may be empty
  * number_columns : an int with the number of columns
  * number_rows : an int with the number of rows. If metadataonly option was used, it may
    be None if the number of rows could not be determined. If you need the number of rows in
    this case you need to parse the whole file.
  * variable_value_labels : a dict with keys being variable names, and values being a dict with values as keys and labels
    as values. It may be empty if the dataset did not contain such labels. For sas7bdat files it will be empty unless
    a sas7bcat was given. It is a combination of value_labels and variable_to_label.
  * value_labels : a dict with label name as key and a dict as value, with values as keys and labels
    as values. In the case of parsing a sas7bcat file this is where the formats are.
  * variable_to_label : A dict with variable name as key and label name as value. Label names are those described in
    value_labels. Sas7bdat files may have this member populated and its information can be used to match the information
    in the value_labels coming from the sas7bcat file.
  * original_variable_types : a dict of variable name to variable format in the original file. For debugging purposes.
  * table_name : table name (string)

There are two functions to deal with value labels: set_value_labels and set_catalog_to_sas. You can read about them
in the next section.

Functions Documentation
=======================

.. automodule:: pyreadstat.pyreadstat
    :members:
    :undoc-members:
    


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
