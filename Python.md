# Python CheetSeet

## Install Python packages 
```
code and notes Pip, easy_install for python packages
Brew install for mac apt_get

Pip for python 2 default
Pip install tensorflow
Pip install numpy
Pip install scipy
Pip install mat
pip install --upgrade matplotlib
pip install  keras==2.2
Pip install jupyter
pip install seaborn

Brew install python3, then you have python3 and pip3
Pip3 for python 3
Install python 3.6.5 since 3.7.0 has no tensorflow
https://stackoverflow.com/questions/51125013/how-can-i-install-a-previous-version-of-python-3-in-macos-using-homebrew

The Jupyter Notebook needs ipython kernel, this fix the dead kernel error
python2 -m pip install ipykernel python2 -m ipykernel install --user
python3 -m pip install ipykernel python3 -m ipykernel install --user

Python setup.py if not found in pip
Annaconda, conda install other packages
```
### pyspark create dataframe 
    >>> df = spark.createDataFrame([(0.5,)], ["values"])

    >>> data = [(0, Vectors.dense([-1.0, -1.0 ]),),
    ...         (1, Vectors.dense([-1.0, 1.0 ]),),
    ...         (2, Vectors.dense([1.0, -1.0 ]),),
    ...         (3, Vectors.dense([1.0, 1.0]),)]
    >>> df = spark.createDataFrame(data, ["id", "features"])
    
    >>> df = spark.createDataFrame(
    ...    [(0, ["a", "b", "c"]), (1, ["a", "b", "b", "c", "a"])],
    ...    ["label", "raw"])
    
    >>> df1 = spark.createDataFrame([(Vectors.dense([5.0, 8.0, 6.0]),)], ["vec"])
    
    >>> df = spark.createDataFrame([(["a", "b", "c"],)], ["words"])


### pandas dataframe for quick plot
https://jakevdp.github.io/PythonDataScienceHandbook/04.14-visualization-with-seaborn.html
https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html
```
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
%matplotlib inline
sns.set_style("whitegrid")
log_df = pd.read_csv("log.csv")
log_df.head()
log_df.groupby('host').agg('count')
from dateutil import parser
log_df['processed_date'] = log_df['logDate'].str.replace(':', ' ', 1).str.replace(r'\[|\]', '')
log_df.describe()
#add a column and set index for plot
one_ip_sample_df = log_df[log_df['host'] == '122.119.64.67']
one_ip_sample_df['dt_index'] = pd.to_datetime(one_ip_sample_df['processed_date'])
one_ip_sample_df = one_ip_sample_df.set_index('dt_index')
def resample_plot(df,freq,col='value'):
    resample_df = df[col].resample(freq).mean().fillna(value=0)
    resample_df.plot()
resample_plot(one_ip_sample_df,'30Min')

#plot original values and resampled values in the same figure
new_df_1['in_value_min'].plot()
resample_plot(new_df_1,'1D',col='in_value_min')
plt.legend(['sampling freq = 1 hour(original)','sampling freq = 1 day'])
plt.title("in_value_min");
```