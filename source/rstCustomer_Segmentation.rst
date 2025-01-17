Bank Customer Segmentation
--------------------------

.. automodule:: scriptNotebook

.. raw:: html

   <h2 style="font-weight:bold; font-family:sans-serif">

Goal of creating this Notebook

.. raw:: html

   </h2>

1. Perform Clustering / Segmentation on the dataset and identify popular
   customer groups along with their definitions/rules
2. Perform Location-wise analysis to identify regional trends in India
3. Perform transaction-related analysis to identify interesting trends
   that can be used by a bank to improve / optimi their user experiences
4. Customer Recency, Frequency, Monetary analysis

**Table of contents of this notebook:**

**1.** Importing Necessary Libraries **2.** Data Collection **3.** Data
Cleaning **4.** Exploratory Data Analysis

.. raw:: html

   <h2  style="text-align: center; padding: 20px; font-weight:bold">

1. Importing Libraries

   .. raw:: html

      </h2>

.. code:: ipython3

    import re
    import numpy as np 
    import pandas as pd 
    import matplotlib.pyplot as plt
    %matplotlib inline
    plt.style.use("fivethirtyeight")
    import seaborn as sns

.. raw:: html

   <h2  style="text-align: center; padding: 20px; font-weight:bold">

2. Data Collection

   .. raw:: html

      </h2>

.. autofunction:: scriptNotebook.import_data

.. code:: ipython3

    def import_data():
        dfGet = pd.read_csv("../data/bank_transactions.csv")
        dfGet = dfGet.rename(columns={'TransactionAmount (INR)':'TransactionAmount'})
        return dfGet
    df = import_data()
    # Showing or Checking results
    df.head()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>TransactionID</th>
          <th>CustomerID</th>
          <th>CustomerDOB</th>
          <th>CustGender</th>
          <th>CustLocation</th>
          <th>CustAccountBalance</th>
          <th>TransactionDate</th>
          <th>TransactionTime</th>
          <th>TransactionAmount</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>T1</td>
          <td>C5841053</td>
          <td>10/1/94</td>
          <td>F</td>
          <td>JAMSHEDPUR</td>
          <td>17819.05</td>
          <td>2/8/16</td>
          <td>143207</td>
          <td>25.0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>T2</td>
          <td>C2142763</td>
          <td>4/4/57</td>
          <td>M</td>
          <td>JHAJJAR</td>
          <td>2270.69</td>
          <td>2/8/16</td>
          <td>141858</td>
          <td>27999.0</td>
        </tr>
        <tr>
          <th>2</th>
          <td>T3</td>
          <td>C4417068</td>
          <td>26/11/96</td>
          <td>F</td>
          <td>MUMBAI</td>
          <td>17874.44</td>
          <td>2/8/16</td>
          <td>142712</td>
          <td>459.0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>T4</td>
          <td>C5342380</td>
          <td>14/9/73</td>
          <td>F</td>
          <td>MUMBAI</td>
          <td>866503.21</td>
          <td>2/8/16</td>
          <td>142714</td>
          <td>2060.0</td>
        </tr>
        <tr>
          <th>4</th>
          <td>T5</td>
          <td>C9031234</td>
          <td>24/3/88</td>
          <td>F</td>
          <td>NAVI MUMBAI</td>
          <td>6714.43</td>
          <td>2/8/16</td>
          <td>181156</td>
          <td>1762.5</td>
        </tr>
      </tbody>
    </table>
    <br>
    </div>



.. autofunction:: scriptNotebook.dfInformation

.. code:: ipython3

    def dfInformation(dataframe):
        
        display(dataframe.info())
    
    dfInformation(df)
    # Getting the dataframe size for following amortized values
    initialRows = len(df)


.. parsed-literal::

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1048567 entries, 0 to 1048566
    Data columns (total 9 columns):
     #   Column              Non-Null Count    Dtype  
    ---  ------              --------------    -----  
     0   TransactionID       1048567 non-null  object 
     1   CustomerID          1048567 non-null  object 
     2   CustomerDOB         1045170 non-null  object 
     3   CustGender          1047467 non-null  object 
     4   CustLocation        1048416 non-null  object 
     5   CustAccountBalance  1046198 non-null  float64
     6   TransactionDate     1048567 non-null  object 
     7   TransactionTime     1048567 non-null  int64  
     8   TransactionAmount   1048567 non-null  float64
    dtypes: float64(2), int64(1), object(6)
    memory usage: 72.0+ MB
    


.. parsed-literal::

    None


.. raw:: html

   <h2  style="text-align: center; padding: 20px; font-weight:bold">

3. Data Cleaning

   .. raw:: html

      </h2>

.. autofunction:: scriptNotebook.check

.. code:: ipython3

    def check(dataframe):
        l=[]
        columns=dataframe.columns
        for col in columns:
            dtypes=dataframe[col].dtypes
            nunique=dataframe[col].nunique()
            sum_null=dataframe[col].isnull().sum()
            l.append([col,dtypes,nunique,sum_null])
        df_check=pd.DataFrame(l)
        df_check.columns=['Column','Types','Unique','Nulls']
        return df_check 
    check(df)




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Column</th>
          <th>Types</th>
          <th>Unique</th>
          <th>Nulls</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>TransactionID</td>
          <td>object</td>
          <td>1048567</td>
          <td>0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>CustomerID</td>
          <td>object</td>
          <td>884265</td>
          <td>0</td>
        </tr>
        <tr>
          <th>2</th>
          <td>CustomerDOB</td>
          <td>object</td>
          <td>17254</td>
          <td>3397</td>
        </tr>
        <tr>
          <th>3</th>
          <td>CustGender</td>
          <td>object</td>
          <td>3</td>
          <td>1100</td>
        </tr>
        <tr>
          <th>4</th>
          <td>CustLocation</td>
          <td>object</td>
          <td>9355</td>
          <td>151</td>
        </tr>
        <tr>
          <th>5</th>
          <td>CustAccountBalance</td>
          <td>float64</td>
          <td>161328</td>
          <td>2369</td>
        </tr>
        <tr>
          <th>6</th>
          <td>TransactionDate</td>
          <td>object</td>
          <td>55</td>
          <td>0</td>
        </tr>
        <tr>
          <th>7</th>
          <td>TransactionTime</td>
          <td>int64</td>
          <td>81918</td>
          <td>0</td>
        </tr>
        <tr>
          <th>8</th>
          <td>TransactionAmount</td>
          <td>float64</td>
          <td>93024</td>
          <td>0</td>
        </tr>
      </tbody>
    </table>
    </div>



Observation:

.. raw:: html

   <h6>

The amount of null values to eliminate is equal to 6953 data, we
eliminate these values because they do not represent more than 0.7% of
the total data. We check if there are repeated elements in our DataSet

.. raw:: html

   </h6>

.. autofunction:: scriptNotebook.removeNullValues

.. code:: ipython3

    def removeNullValues(dataframe):
        shapeInitial = dataframe.shape[0]
        dataframe.dropna(inplace=True)
        return shapeInitial-dataframe.shape[0]
    
    print("Amount to remove " + str(removeNullValues(df)))


.. parsed-literal::

    Amount to remove 6953
    
.. autofunction:: scriptNotebook.checkDuplicates

.. code:: ipython3

    def checkDuplicates(dataframe):
        return dataframe.duplicated().sum()
    checkDuplicates(df)




.. parsed-literal::

    0



Consideration

.. raw:: html

   <h6>

The CustomerDOB column is analyzed because it may contain atypical data.
We analyze the number of records for each client’s date of birth.

.. raw:: html

   </h6>

.. autofunction:: scriptNotebook.uniqueRows

.. code:: ipython3

    def uniqueRows(dataframe, column):
        return dataframe[column].value_counts()
    uniqueRows(df,'CustomerDOB')




.. parsed-literal::

    1/1/1800    56292
    1/1/89        809
    1/1/90        784
    6/8/91        698
    1/1/91        665
                ...  
    2/12/51         1
    20/3/52         1
    26/9/47         1
    4/10/41         1
    24/10/44        1
    Name: CustomerDOB, Length: 17233, dtype: int64



Take in mind

.. raw:: html

   <h6>

Dates 1/1/1800 are deleted because it is not possible to define whether
they are children, adults or persons without date of birth. This is an
important variable for the business, for this reason we cannot make
assumptions that bias the project, for this reason, it is better to
eliminate these outliers or erroneously measured data.

.. raw:: html

   </h6>

.. autofunction:: scriptNotebook.removeValues

.. code:: ipython3

    def removeValues(dataframe,column, value):
        return dataframe.loc[~(dataframe[column] == value)]
    
    df = removeValues(df,'CustomerDOB','1/1/1800')
    # Cheking distinct values from dataframe
    uniqueRows(df,'CustomerDOB')




.. parsed-literal::

    1/1/89      809
    1/1/90      784
    6/8/91      698
    1/1/91      665
    1/1/92      631
               ... 
    23/2/05       1
    28/11/42      1
    23/9/49       1
    14/3/40       1
    24/10/44      1
    Name: CustomerDOB, Length: 17232, dtype: int64


.. autofunction:: scriptNotebook.minAndMax

.. code:: ipython3

    def minAndMax(dataframe, column):
        print("min: " + str(dataframe[column].min()) + " max: " + str(dataframe[column].max()))
    minAndMax(df,'CustomerDOB')


.. parsed-literal::

    min: 1973-01-01 00:00:00 max: 2072-12-31 00:00:00
    

It can be seen that the person with the oldest birth date has a date of
January 1, 1900 and the youngest person has a date of September 9, 1997.
(It looks weird)

Convert type of columns TransactionDate, CustomerDOB from string to
datetime, this convertation will be in the format of dayfirst, so the
date will be DD/MM/YY

.. autofunction:: scriptNotebook.dateConvertion

.. code:: ipython3

    def dateConvertion(dataframe, column):
        return pd.to_datetime(dataframe[column], dayfirst=True)
    df['CustomerDOB'] = dateConvertion(df,'CustomerDOB')

Now we will check if the conversion was as expected and in the required
format.

.. code:: ipython3

    # Checking converting problem of to_datetime pandas function
    minAndMax(df,'CustomerDOB')


.. parsed-literal::

    min: 1973-01-01 00:00:00 max: 2072-12-31 00:00:00
    

We can see that the most “recent” date is December 31, 2072, but it is
illogical because this is a future date, so we subtract 100 from all
values greater than 1999 to get the real value. (This is a problem
Pandas has when converting a date).

.. autofunction:: scriptNotebook.refactorDates

.. code:: ipython3

    def refactorDates(dataframe):
        dataframe.loc[df['CustomerDOB'].dt.year > 1999, 'CustomerDOB'] -= pd.DateOffset(years=100)
        return dataframe
    df = refactorDates(df)
    minAndMax(df,'CustomerDOB')


.. parsed-literal::

    min: 1900-01-01 00:00:00 max: 1999-12-28 00:00:00
    


In the same way that we converted the CustomerDOB column, we convert the
TransactionDate column with the same expected format as the first day.

.. code:: ipython3

    # Using pandas convert to datetime tool for TransactionDate variable
    df['TransactionDate'] = dateConvertion(df,'TransactionDate')
    # Checking range of TransactionDate variable
    minAndMax(df,'TransactionDate')


.. parsed-literal::

    min: 2016-08-01 00:00:00 max: 2016-10-21 00:00:00
    

All the trnasactions took place in a roughly two month period from
August to October, this could account for the low transaction frequency

Once all the unnecessary data for the study has been eliminated, we can
see the following summary, which shows us how much data we lost and what
is the percentage of loss obtained

.. code:: ipython3

    print(" Number of initial rows: ", initialRows, "\n",
    "Number of rows after: ", df.shape[0], "\n",
    "Number of rows deleted: ", initialRows - df.shape[0], "\n",
    "Percentage of rows deleted: ", (initialRows - df.shape[0]) / initialRows * 100, "%")


.. parsed-literal::

     Number of initial rows:  1048567 
     Number of rows after:  985322 
     Number of rows deleted:  63245 
     Percentage of rows deleted:  6.03156498344884 %
    

We can see that we lost 6.03% of the data, although what is expected by
theory is to lose less than 5% in this case we must ignore this metric
because there are null values and measurement failure errors that force
us to eliminate them, because we cannot speculate about them.

.. raw:: html

   <h2  style="text-align: center;padding: 20px;font-weight:bold">

4. Exploratory Data Analysis

   .. raw:: html

      </h2>

Determine minority group of people aged over 100 years

.. autofunction:: scriptNotebook.filterDOB

.. code:: ipython3

    # We filter our dataframe specifically on the DOB column to make a decision regarding date ambiguity.
    def filterDOB(dataframe):
        return dataframe["CustomerDOB"].apply(lambda x: x if x.year < 1917 else 0)
    
    df_filtered = filterDOB(df)
    
    
    # Amortizing and removing values ​​greater than 1917 represented as 0
    counts = df_filtered.value_counts().drop(0)
    print(counts)
    # Amount of customer in this age range
    print(len(counts))
    # Plot the amortized
    counts.plot()
    plt.show()
    del df_filtered


.. parsed-literal::

    1900-04-11    18
    1902-03-31    18
    1903-02-19    18
    1901-09-22    18
    1907-01-02    16
                  ..
    1913-10-22     1
    1913-02-04     1
    1905-02-23     1
    1903-03-07     1
    1915-11-18     1
    Name: CustomerDOB, Length: 254, dtype: int64
    254
    


.. image:: Customer_Segmentation_files/Customer_Segmentation_32_1.png


.. raw:: html

   <h3>

Calculate customer age :

.. raw:: html

   </h3>

CustomerDOB: is the birth date of the customer TransactionDate: is the
date of transaction that customer is done The age calculation is done by
subtracting the TransactionDate from the CustomerDOB.

.. autofunction:: scriptNotebook.getCustomerAge

.. code:: ipython3

    # Getting the customer age at transaction moment and adding a new column in our dataframe
    def getCustomerAge(dataframe):
        df['CustomerAge'] = (df['TransactionDate'] - df['CustomerDOB'])/np.timedelta64(1, 'Y')
        df['CustomerAge'] = df['CustomerAge'].astype(int)
        # Checking range of CustomerAge variable
        print("min: " + str(df['CustomerAge'].min()) + " max: " + str(df['CustomerAge'].max()))
    
    getCustomerAge(df)
    


.. parsed-literal::

    min: 16 max: 116
    

Once this is obtained, we have that the minimum age is equal to 16 years
and the maximum age is equal to 116 years, it should be noted that the
ages over 100 are a minimum percentage.

We obtain the percentage between customers who are women and men.

.. code:: ipython3

    # Getting distinct values from CustGender variable
    df.CustGender.value_counts()




.. parsed-literal::

    M    712454
    F    272868
    Name: CustGender, dtype: int64



.. raw:: html

   <h5>

Visualize the distribution of the numeric data and detect posible
outliers. Boxplots show the median, quartiles, and extreme values ​​of the
data, and points that are above or below the extreme values ​​are
considered outliers.

.. raw:: html

   </h5>

.. autofunction:: scriptNotebook.outliers

.. code:: ipython3

    def outliers(dataframe):
        num_col = df.select_dtypes(include=np.number)
        cat_col = df.select_dtypes(exclude=np.number)
    
        for i in num_col.columns:
            sns.boxplot(x=num_col[i])
            plt.title("Boxplot " + i)
            plt.show()
    
    outliers(df)



.. image:: Customer_Segmentation_files/Customer_Segmentation_39_0.png



.. image:: Customer_Segmentation_files/Customer_Segmentation_39_1.png



.. image:: Customer_Segmentation_files/Customer_Segmentation_39_2.png



.. image:: Customer_Segmentation_files/Customer_Segmentation_39_3.png


.. raw:: html

   <h3 style="font-family:Sans-Serif; font-weight:bold">

RFM Metrics:

.. raw:: html

   </h3>

.. raw:: html

   <ul>

.. raw:: html

   <li>

Recency: The freshness of customer activity e.g. time since last
activity

.. raw:: html

   </li>

.. raw:: html

   <li>

Frequency: The requency of customer transactions e.g. the totla number
of recorded transactions

.. raw:: html

   </li>

.. raw:: html

   <li>

Monetary: The willingness to spend e.g. the thoal transaction value

.. raw:: html

   </li>

.. raw:: html

   </ul>

.. raw:: html

   <p>

Those two articles will help you to understand this topic:

.. raw:: html

   </p>

What Are RFM Scores and How To Calculate Them Introduction to Customer
Segmentation in Python

We prepare some columns to make the RFM table

.. code:: ipython3

    df['TransactionDate1']=df['TransactionDate'] # ==> to calculate the minimum (first transaction)
    df['TransactionDate2']=df['TransactionDate'] # ==> to calculate the maximum (last transaction)

This code is used to create a RFM (Recency, Frequency, Monetary) table.
The data is grouped by CustomerID using the groupby method and then the
agg function is used to calculate different metrics for each customer.

The metrics that are calculated are as follows:

.. raw:: html

   <ul>

.. raw:: html

   <li>

TransactionID: number of transactions performed by the customer.

.. raw:: html

   </li>

.. raw:: html

   <li>

CustGender: customer’s gender (taken from the first transaction recorded
for the customer).

.. raw:: html

   </li>

.. raw:: html

   <li>

CustLocation: location of the customer (also taken from the first
transaction recorded for the customer).

.. raw:: html

   </li>

.. raw:: html

   <li>

CustAccountBalance: average balance of the customer’s account.

.. raw:: html

   </li>

.. raw:: html

   <li>

TransactionTime: average time of transactions performed by the customer.

.. raw:: html

   </li>

.. raw:: html

   <li>

TransactionAmount: average amount of transactions made by the customer.

.. raw:: html

   </li>

.. raw:: html

   <li>

CustomerAge: median age of the client.

.. raw:: html

   </li>

.. raw:: html

   <li>

TransactionDate2: most recent date on which the customer made a
transaction.

.. raw:: html

   </li>

.. raw:: html

   <li>

TransactionDate1: oldest date on which the customer made a transaction.

.. raw:: html

   </li>

.. raw:: html

   <li>

TransactionDate: median date on which the customer made a transaction.

.. raw:: html

   </li>

.. raw:: html

   </ul>

.. autofunction:: scriptNotebook.MRFTable

.. code:: ipython3

    #Creating MRF Table Strategy
    
    def MRFTable(df):       
        RFM_df = df.groupby("CustomerID").agg({
                                                "TransactionID" : "count",
                                                "CustGender" : "first",
                                                "CustLocation":"first",
                                                "CustAccountBalance"  : "mean",
                                                "TransactionTime": "mean",
                                                "TransactionAmount" : "mean",
                                                "CustomerAge" : "median",
                                                "TransactionDate2":"max",
                                                "TransactionDate1":"min",
                                                "TransactionDate":"median"
                                })
    
        RFM_df.reset_index()
        return RFM_df
        
    
    RFM_df = MRFTable(df)
    RFM_df.head()
    




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>CustomerID</th>
          <th>TransactionID</th>
          <th>CustGender</th>
          <th>CustLocation</th>
          <th>CustAccountBalance</th>
          <th>TransactionTime</th>
          <th>TransactionAmount</th>
          <th>CustomerAge</th>
          <th>TransactionDate2</th>
          <th>TransactionDate1</th>
          <th>TransactionDate</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>C1010011</td>
          <td>2</td>
          <td>F</td>
          <td>NOIDA</td>
          <td>76340.635</td>
          <td>67521.0</td>
          <td>2553.0</td>
          <td>28.5</td>
          <td>2016-09-26</td>
          <td>2016-08-09</td>
          <td>2016-09-02</td>
        </tr>
        <tr>
          <th>1</th>
          <td>C1010012</td>
          <td>1</td>
          <td>M</td>
          <td>MUMBAI</td>
          <td>24204.490</td>
          <td>204409.0</td>
          <td>1499.0</td>
          <td>22.0</td>
          <td>2016-08-14</td>
          <td>2016-08-14</td>
          <td>2016-08-14</td>
        </tr>
        <tr>
          <th>2</th>
          <td>C1010014</td>
          <td>2</td>
          <td>F</td>
          <td>MUMBAI</td>
          <td>100112.950</td>
          <td>187378.0</td>
          <td>727.5</td>
          <td>27.5</td>
          <td>2016-08-07</td>
          <td>2016-08-01</td>
          <td>2016-08-04</td>
        </tr>
        <tr>
          <th>3</th>
          <td>C1010018</td>
          <td>1</td>
          <td>F</td>
          <td>CHAMPARAN</td>
          <td>496.180</td>
          <td>170254.0</td>
          <td>30.0</td>
          <td>26.0</td>
          <td>2016-09-15</td>
          <td>2016-09-15</td>
          <td>2016-09-15</td>
        </tr>
        <tr>
          <th>4</th>
          <td>C1010024</td>
          <td>1</td>
          <td>M</td>
          <td>KOLKATA</td>
          <td>87058.650</td>
          <td>141103.0</td>
          <td>5000.0</td>
          <td>51.0</td>
          <td>2016-08-18</td>
          <td>2016-08-18</td>
          <td>2016-08-18</td>
        </tr>
      </tbody>
    </table>
    </div>



Now we calculate the number of records we have left after grouping by
CustomerID

.. code:: ipython3

    # After Grouping by CustomerID
    RFM_df.shape




.. parsed-literal::

    (839081, 11)



The ID of the customer is irrelevant to solve our problem, so we decided
to remove it

.. code:: ipython3

    # The id of the customer is irrelevant
    RFM_df.drop(columns=["CustomerID"],inplace=True)

.. raw:: html

   <h4>

Frequency

.. raw:: html

   </h4>

.. raw:: html

   <p>

As we count the TransactionID column, we can replace the name of this
column by Frequency, because this is the number of times a customer has
made a transaction.

.. raw:: html

   </p>

.. code:: ipython3

    # Renaming specific column adapting to problem goal and replacing with inplace property of function
    RFM_df.rename(columns={"TransactionID":"Frequency"},inplace=True)

.. raw:: html

   <h4>

Recency

.. raw:: html

   </h4>

.. raw:: html

   <p>

The recency is the number of days since the last purchase or order so we
will create a new column of TransactionDate to subtract the last
transaction from the first transaction

.. raw:: html

   </p>

.. code:: ipython3

    # Getting Recency that is by definition: number of days since the last purchase or order
    RFM_df['Recency']=RFM_df['TransactionDate2']-RFM_df['TransactionDate1']
    # Conversion from timedelta64[ns] to string representtion in days of weeks of Recency variable
    RFM_df['Recency']=RFM_df['Recency'].astype(str)

We apply a lambda function to adjust the format of our output in the
Recency variable

.. autofunction:: scriptNotebook.formatOutputInRecency

.. code:: ipython3

    def formatOutputInRecency(RFM_df):
        # Using re library for apply an regular expresion in each value of Recency column for extract the number of days in this string representation. 
        RFM_df['Recency']=RFM_df['Recency'].apply(lambda x :re.search('\d+',x).group())
        # Conversion from string '18' to int representtion for folloeing operations
        RFM_df['Recency']=RFM_df['Recency'].astype(int)
    
    formatOutputInRecency(RFM_df)

.. raw:: html

   <p>

Appreciation: Days mean that a customer has done transaction recently
one time by logic so I will convert 0 to 1

.. raw:: html

   </p>

.. code:: ipython3

    # Transformation of 0 days base on business meaning
    RFM_df['Recency'] = RFM_df['Recency'].apply(lambda x: 1 if x == 0 else x)

The TransactionDate1 and TransactionDate2 columns have already fulfilled
their objectives, which is to calculate the Recency, we can eliminate
these columns.

.. code:: ipython3

    # Columns that were only needed for the calculation we eliminated
    RFM_df.drop(columns=["TransactionDate1","TransactionDate2"],inplace=True)

Now, let’s see if our DataSet once cleaned contains atypical data

.. autofunction:: scriptNotebook.outliersWhenCleaned

.. code:: ipython3

    # To calculate the otliers for each feature
    def outliersWhenCleaned():
        lower_list=[]
        upper_list=[]
        num_list=[]
        perc_list=[]
        cols=['Frequency', 'CustAccountBalance','TransactionAmount', 'CustomerAge', 'Recency']
        for i in cols:
            Q1 = RFM_df[i].quantile(0.25)
            Q3 = RFM_df[i].quantile(0.75)
            IQR = Q3 - Q1
            lower = Q1 - 1.5 * IQR
            upper = Q3 + 1.5 * IQR
            # Calculate number of outliers
            num=RFM_df[(RFM_df[i] < lower) | (RFM_df[i] > upper)].shape[0]
            # Calculate percentage of outliers
            perc = (num / RFM_df.shape[0]) * 100
            lower_list.append(lower)
            upper_list.append(upper)
            num_list.append(num)
            perc_list.append(round(perc,2))
    
            
        dic={'lower': lower_list, 'upper': upper_list, 'outliers': num_list, 'Perc%':perc_list }
        outliers_df=pd.DataFrame(dic,index=['Frequency', 'CustAccountBalance','TransactionAmount', 'CustomerAge', 'Recency'])
        outliers_df
    
    outliersWhenCleaned()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>lower</th>
          <th>upper</th>
          <th>outliers</th>
          <th>Perc%</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>Frequency</th>
          <td>1.000</td>
          <td>1.000</td>
          <td>128896</td>
          <td>15.36</td>
        </tr>
        <tr>
          <th>CustAccountBalance</th>
          <td>-72439.305</td>
          <td>135042.015</td>
          <td>110026</td>
          <td>13.11</td>
        </tr>
        <tr>
          <th>TransactionAmount</th>
          <td>-1313.595</td>
          <td>2669.325</td>
          <td>87229</td>
          <td>10.40</td>
        </tr>
        <tr>
          <th>CustomerAge</th>
          <td>11.500</td>
          <td>47.500</td>
          <td>40804</td>
          <td>4.86</td>
        </tr>
        <tr>
          <th>Recency</th>
          <td>1.000</td>
          <td>1.000</td>
          <td>122150</td>
          <td>14.56</td>
        </tr>
      </tbody>
    </table>
    </div>



.. raw:: html

   <h3 style="font-family:Sans-Serif; font-weight:bold">

Observations:

.. raw:: html

   </h3>

.. raw:: html

   <p>

We will not remove outliers for the following two reasons: First, in
boxplots those values ​​can be outliers because they represent points that
are above or below extreme values. However, these were not measurement
errors and are both true and significant, given that while customers
100+ do not represent a key demographic for most banks. Secoind it is
important that banks are aware of the specific needs and challenges that
these clients may face, and that they adapt their strategies
accordingly.

.. raw:: html

   </p>

Now, let’s go to see our RFM Table

.. code:: ipython3

    RFM_df.head()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Frequency</th>
          <th>CustGender</th>
          <th>CustLocation</th>
          <th>CustAccountBalance</th>
          <th>TransactionTime</th>
          <th>TransactionAmount</th>
          <th>CustomerAge</th>
          <th>TransactionDate</th>
          <th>Recency</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2</td>
          <td>F</td>
          <td>NOIDA</td>
          <td>76340.635</td>
          <td>67521.0</td>
          <td>2553.0</td>
          <td>28.5</td>
          <td>2016-09-02</td>
          <td>48</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1</td>
          <td>M</td>
          <td>MUMBAI</td>
          <td>24204.490</td>
          <td>204409.0</td>
          <td>1499.0</td>
          <td>22.0</td>
          <td>2016-08-14</td>
          <td>1</td>
        </tr>
        <tr>
          <th>2</th>
          <td>2</td>
          <td>F</td>
          <td>MUMBAI</td>
          <td>100112.950</td>
          <td>187378.0</td>
          <td>727.5</td>
          <td>27.5</td>
          <td>2016-08-04</td>
          <td>6</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1</td>
          <td>F</td>
          <td>CHAMPARAN</td>
          <td>496.180</td>
          <td>170254.0</td>
          <td>30.0</td>
          <td>26.0</td>
          <td>2016-09-15</td>
          <td>1</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1</td>
          <td>M</td>
          <td>KOLKATA</td>
          <td>87058.650</td>
          <td>141103.0</td>
          <td>5000.0</td>
          <td>51.0</td>
          <td>2016-08-18</td>
          <td>1</td>
        </tr>
      </tbody>
    </table>
    </div>



We describe each of the columns with different factors

.. code:: ipython3

    RFM_df.describe()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Frequency</th>
          <th>CustAccountBalance</th>
          <th>TransactionTime</th>
          <th>TransactionAmount</th>
          <th>CustomerAge</th>
          <th>Recency</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>count</th>
          <td>839081.000000</td>
          <td>8.390810e+05</td>
          <td>839081.000000</td>
          <td>8.390810e+05</td>
          <td>839081.000000</td>
          <td>839081.000000</td>
        </tr>
        <tr>
          <th>mean</th>
          <td>1.174287</td>
          <td>1.058545e+05</td>
          <td>157446.381829</td>
          <td>1.453624e+03</td>
          <td>30.755438</td>
          <td>3.666118</td>
        </tr>
        <tr>
          <th>std</th>
          <td>0.435129</td>
          <td>7.862524e+05</td>
          <td>49194.229430</td>
          <td>6.045200e+03</td>
          <td>8.706026</td>
          <td>8.061373</td>
        </tr>
        <tr>
          <th>min</th>
          <td>1.000000</td>
          <td>0.000000e+00</td>
          <td>0.000000</td>
          <td>0.000000e+00</td>
          <td>16.000000</td>
          <td>1.000000</td>
        </tr>
        <tr>
          <th>25%</th>
          <td>1.000000</td>
          <td>5.366190e+03</td>
          <td>125604.000000</td>
          <td>1.800000e+02</td>
          <td>25.000000</td>
          <td>1.000000</td>
        </tr>
        <tr>
          <th>50%</th>
          <td>1.000000</td>
          <td>1.768220e+04</td>
          <td>163936.000000</td>
          <td>4.750000e+02</td>
          <td>29.000000</td>
          <td>1.000000</td>
        </tr>
        <tr>
          <th>75%</th>
          <td>1.000000</td>
          <td>5.723652e+04</td>
          <td>194953.000000</td>
          <td>1.175730e+03</td>
          <td>34.000000</td>
          <td>1.000000</td>
        </tr>
        <tr>
          <th>max</th>
          <td>6.000000</td>
          <td>1.150355e+08</td>
          <td>235959.000000</td>
          <td>1.560035e+06</td>
          <td>116.000000</td>
          <td>81.000000</td>
        </tr>
      </tbody>
    </table>
    </div>



It creates a correlation matrix between the different features in the
RFM table (RFM_df), and then plots this matrix as a heat map using it.
The correlation matrix is a square matrix that shows how the different
features are related to each other.

.. autofunction:: scriptNotebook.correlation

.. code:: ipython3

    # correlation between features
    def correlation():   
        plt.figure(figsize=(7,5))
        correlation=RFM_df.corr(numeric_only=True)
        sns.heatmap(correlation,vmin=None,
            vmax=0.8,
            cmap='rocket_r',
            annot=True,
            fmt='.1f',
            linecolor='white',
            cbar=True);
    
    correlation()



.. image:: Customer_Segmentation_files/Customer_Segmentation_68_0.png


We obtain the frequency bar chart, this chart shows the distribution of
the variable Frequency, it is worth noting that the frequency is the
number of times a customer has made transactions in the period from
August to October.

.. autofunction:: scriptNotebook.distributionFrequency

.. code:: ipython3

    def distributionFrequency():
        plt.style.use("fivethirtyeight")
        chart=sns.countplot(x='Frequency',data=RFM_df,palette='rocket', order = RFM_df['Frequency'].value_counts().index)
        plt.title("Frequency",
                fontsize='20',
                backgroundcolor='AliceBlue',
                color='magenta');
    
    distributionFrequency()



.. image:: Customer_Segmentation_files/Customer_Segmentation_70_0.png


We obtain the age distribution of the clients and also the percentage of
women and men in the records we have.

.. autofunction:: scriptNotebook.graphAgeAndGender

.. code:: ipython3

    def graphAgeAndGender():
        plt.style.use("fivethirtyeight")
        fig,ax=plt.subplots(ncols=2,nrows=1,figsize=(15,5))
        palette_color = sns.color_palette('rocket')
        ax[0].hist(x=RFM_df['CustomerAge'],color='purple')
        ax[0].set_title("Distribution of Customer Age")
        ax[1].pie(RFM_df['CustGender'].value_counts(),autopct='%1.f%%',colors=palette_color,labels=['Male','Female'])
        ax[1].set_title("Customer Gender")
        plt.tight_layout();
    
    graphAgeAndGender()



.. image:: Customer_Segmentation_files/Customer_Segmentation_72_0.png


In this graph we obtain the number of times a transaction was made in
different areas of the country, only the top 20 locations with the most
transactions made will be shown.

.. autofunction:: scriptNotebook.graphLocation

.. code:: ipython3

    def graphLocation():
        plt.style.use("fivethirtyeight")
        plt.figure(figsize=(15,7))
        chart=sns.countplot(y='CustLocation',data=RFM_df,palette='rocket', order = RFM_df['CustLocation'].value_counts()[:20].index)
        plt.title("Most 20 Location of Customer ",
                fontsize='15',
                backgroundcolor='AliceBlue',
                color='magenta');
    
    graphLocation()



.. image:: Customer_Segmentation_files/Customer_Segmentation_74_0.png


We generate the scatter plot of the data referring to the variable
Frequency.

.. autofunction:: scriptNotebook.scatterOFData

.. code:: ipython3

    def scatterOFData():
        plt.style.use("fivethirtyeight")
        sns.pairplot(RFM_df,hue='Frequency')
    
    scatterOFData()




.. parsed-literal::

    <seaborn.axisgrid.PairGrid at 0x2238b194bb0>




.. image:: Customer_Segmentation_files/Customer_Segmentation_76_1.png


This code generates a scatter plot. The data used comes from the RFM_df
dataframe and is represented on the X axis (horizontal) the transaction
amounts (TransactionAmount) and on the Y axis (vertical) the customer’s
account balance (CustAccountBalance). We add a third dimension to the
graph which is Frequency and a fourth one with Recency.

.. autofunction:: scriptNotebook.graphTransactionAmountAndCustAccountBalance

.. code:: ipython3

    def graphTransactionAmountAndCustAccountBalance():
        plt.style.use("fivethirtyeight")
        sns.scatterplot(x='TransactionAmount',y='CustAccountBalance',data=RFM_df,palette='rocket',hue='Frequency',size='Recency' )
        plt.legend(loc = "upper right")
        plt.title("TransactionAmount (INR) and CustAccountBalance",
                fontsize='15',
                backgroundcolor='AliceBlue',
                color='magenta');
    
    graphTransactionAmountAndCustAccountBalance()



.. image:: Customer_Segmentation_files/Customer_Segmentation_78_0.png


We calculate the farthest distance between two completed transactions

.. code:: ipython3

    # difference between maximum and minimum date
    RFM_df['TransactionDate'].max()-RFM_df['TransactionDate'].min()




.. parsed-literal::

    Timedelta('81 days 00:00:00')



We group the transactions according to the month in which they were made
and obtain the average for each table.

.. autofunction:: scriptNotebook.groupTransaccionsByMonth

.. code:: ipython3

    def groupTransaccionsByMonth():
        RFM_df=RFM_df.sort_values(by='TransactionDate')
        groupbby_month = RFM_df.groupby([pd.Grouper(key='TransactionDate', freq='M')])[['Frequency', 'TransactionAmount', 'CustAccountBalance', 'TransactionTime', 'CustomerAge', 'Recency']].mean()
        print(groupbby_month.shape)
        return groupbby_month
    
    groupbby_month = groupTransaccionsByMonth()


.. parsed-literal::

    (3, 6)
    



.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Frequency</th>
          <th>TransactionAmount</th>
          <th>CustAccountBalance</th>
          <th>TransactionTime</th>
          <th>CustomerAge</th>
          <th>Recency</th>
        </tr>
        <tr>
          <th>TransactionDate</th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>2016-08-31</th>
          <td>1.196796</td>
          <td>1455.968945</td>
          <td>106177.136987</td>
          <td>156767.237318</td>
          <td>30.780328</td>
          <td>4.099551</td>
        </tr>
        <tr>
          <th>2016-09-30</th>
          <td>1.135252</td>
          <td>1445.607858</td>
          <td>105375.580700</td>
          <td>158413.057073</td>
          <td>30.710065</td>
          <td>2.902352</td>
        </tr>
        <tr>
          <th>2016-10-31</th>
          <td>1.056486</td>
          <td>1904.367370</td>
          <td>94726.805639</td>
          <td>185756.771718</td>
          <td>30.884106</td>
          <td>2.816907</td>
        </tr>
      </tbody>
    </table>
    </div>



We made line graphs of the information we obtained previously.

.. autofunction:: scriptNotebook.grphLineBalanceAndTransactionAmount

.. code:: ipython3

    def grphLineBalanceAndTransactionAmount():
        plt.figure(figsize=(13.4,7))
        plt.title("Average of account balance per month")
        plt.plot(groupbby_month.index,groupbby_month['CustAccountBalance'],color='purple',marker='o',label='Customer Account Balance', linestyle='dashed', linewidth=2, markersize=10)
        plt.legend();
    
        plt.figure(figsize=(13.8,7))
        plt.title("Average of transaction amount per month")
        plt.plot(groupbby_month.index,groupbby_month['TransactionAmount'],color='green',marker='o',label='Transaction Amount', linestyle='dashed', linewidth=2, markersize=10)
    
    grphLineBalanceAndTransactionAmount()




.. parsed-literal::

    [<matplotlib.lines.Line2D at 0x2238b3089d0>]




.. image:: Customer_Segmentation_files/Customer_Segmentation_84_1.png


.. image:: Customer_Segmentation_files/Customer_Segmentation_84_2.png


5. Feature Engineering
----------------------

.. code:: ipython3

    RFM_df=RFM_df.reset_index(drop=True)

.. autofunction:: scriptNotebook.replaceGenderforInt

.. code:: ipython3

    def replaceGenderforInt():
        
        """
        Replace de gender data to -1 if is women and 1 if is men
    
        Args:
            none
    
        Returns:
            none
    
        Raises:
            TypeError: If the dataframe is not a DataFrame.
        """
        RFM_df.CustGender.replace(['F','M'],[-1,1],inplace=True)
        RFM_df.CustGender = RFM_df.CustGender.astype(np.int64)
    
    replaceGenderforInt()

We decided to drop the next features, as they are not important to the
problem.

.. code:: ipython3

    RFM_df.drop(['TransactionDate'], axis=1, inplace=True) # Porque creamos 3 variable basadas en la fecha, Dia, Mes, Nombre dia
    RFM_df.drop(['Recency'], axis=1, inplace=True) # Correlación con variable frecuuencia

.. autofunction:: scriptNotebook.dataToEncoder

.. code:: ipython3

    def dataToEncoder(RFM_df):
          
        """
          Apply the label encoder to the data, transform variable type objects or string to int
    
          Args:
            The Dataset
    
          Returns:
            none
    
          Raises:
            TypeError: If the dataframe is not a DataFrame.
        """
        encoder = LabelEncoder()
        RFM_df.CustLocation = encoder.fit_transform(RFM_df.CustLocation)
        RFM_df.TransactionMonthName = encoder.fit_transform(RFM_df.TransactionMonthName)
        RFM_df.TransactionDayName = encoder.fit_transform(RFM_df.TransactionDayName)
        # Custom Cast
        RFM_df.CustLocation = RFM_df.CustLocation.astype(np.int64)
        RFM_df.TransactionMonthName = RFM_df.TransactionMonthName.astype(np.int64)
        RFM_df.TransactionDayName = RFM_df.TransactionDayName.astype(np.int64)
        return RFM_df
    
    RFM_df = dataToEncoder(RFM_df)

Tecnicas de reducción de dimencionalidad: Analisis de Componentes
principales

El gráfico muestra la suma acumulativa de la varianza explicada por cada
componente principal (PC) en un Análisis de Componentes Principales
(PCA). La varianza explicada indica la cantidad de información que se
conserva después de reducir la dimensionalidad de los datos. En el eje x
del gráfico, se muestra el número de componentes principales y en el eje
y se muestra la varianza explicada acumulativa. La línea azul muestra
cómo se acumula la varianza explicada a medida que se agregan más
componentes principales. Por lo tanto, el gráfico permite evaluar
cuántos componentes principales son necesarios para conservar la mayor
cantidad de información posible en los datos. En general, se busca
encontrar un equilibrio entre la cantidad de componentes principales
utilizados y la cantidad de varianza explicada.

.. autofunction:: scriptNotebook.pca_analysis

.. code:: ipython3

    from sklearn.preprocessing import FunctionTransformer
    
    def pca_analysis(df):
    
        """
        We are going to do the method of PCA to find the best features
    
        Args:
            The Dataset
    
        Returns:
            none
    
        Raises:
            TypeError: If the dataframe is not a DataFrame.
        """
    
        pca = PCA()
        pca.fit(df)
        plt.plot(range(1,len(pca.explained_variance_)+1), pca.explained_variance_ratio_.cumsum(), marker='o', linestyle='--')
    
        components = pca.components_
        variance_ratio = pca.explained_variance_ratio_
        features_importance = pd.DataFrame({"feature": df.columns, "importance": variance_ratio})
        features_importance = features_importance.sort_values("importance", ascending=False)
        return features_importance
    
    pipeline = Pipeline([
        ('pca', FunctionTransformer(pca_analysis))
    ])
    
    features_importance = pipeline.fit_transform(RFM_df)
    print(features_importance)


.. parsed-literal::

                     feature    importance
    0              Frequency  9.960355e-01
    1             CustGender  3.899204e-03
    2           CustLocation  5.869700e-05
    3     CustAccountBalance  6.552220e-06
    4        TransactionTime  1.216498e-10
    5      TransactionAmount  1.175417e-10
    6            CustomerAge  6.117558e-12
    7       TransactionMonth  1.612191e-12
    8   TransactionMonthName  1.287304e-12
    9         TransactionDay  3.009819e-13
    10    TransactionDayName  8.716306e-15
    

.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_95_1.png


We only selected the best features from the dataset, so we discarded
those of lesser importance.

.. code:: ipython3

    RFM_df.drop(['TransactionMonth', 'TransactionMonthName', 'TransactionDay', 'TransactionDayName'], axis=1, inplace=True)

Now, we are going to apply a scaling method, the goal is to get a mean
equal to 0 and a standard deviation equal to 1.

.. autofunction:: scriptNotebook.scale_data

.. code:: ipython3

    def scale_data(df):
        """
        Scale the data using StandardScaler.
    
        Args:
            df (pd.DataFrame): The input data to be scaled.
    
        Returns:
            pd.DataFrame: The scaled data.
    
        """
        column_names = df.columns
        scaler = StandardScaler()
        scaler.fit(df)
        df = pd.DataFrame(scaler.transform(df), columns=column_names)
        return df
    
    pipeline = Pipeline([
        ('scaler', FunctionTransformer(scale_data)),
    ])
    
    RFM_df = pipeline.fit_transform(RFM_df)
    RFM_df.head()




.. raw:: html

    
      <div id="df-4f03d62c-db6f-4120-ae56-24f95a7987a2">
        <div class="colab-df-container">
          <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Frequency</th>
          <th>CustGender</th>
          <th>CustLocation</th>
          <th>CustAccountBalance</th>
          <th>TransactionTime</th>
          <th>TransactionAmount</th>
          <th>CustomerAge</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1.897629</td>
          <td>-1.615225</td>
          <td>0.746619</td>
          <td>-0.037537</td>
          <td>-1.827967</td>
          <td>0.181860</td>
          <td>-0.259066</td>
        </tr>
        <tr>
          <th>1</th>
          <td>-0.400541</td>
          <td>0.619109</td>
          <td>0.493720</td>
          <td>-0.103847</td>
          <td>0.954637</td>
          <td>0.007506</td>
          <td>-1.005676</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1.897629</td>
          <td>-1.615225</td>
          <td>0.493720</td>
          <td>-0.007302</td>
          <td>0.608438</td>
          <td>-0.120116</td>
          <td>-0.373929</td>
        </tr>
        <tr>
          <th>3</th>
          <td>-0.400541</td>
          <td>-1.615225</td>
          <td>-1.087143</td>
          <td>-0.134001</td>
          <td>0.260348</td>
          <td>-0.235497</td>
          <td>-0.546224</td>
        </tr>
        <tr>
          <th>4</th>
          <td>-0.400541</td>
          <td>0.619109</td>
          <td>0.071231</td>
          <td>-0.023906</td>
          <td>-0.332222</td>
          <td>0.586644</td>
          <td>2.325352</td>
        </tr>
      </tbody>
    </table>
    </div>
          <button class="colab-df-convert" onclick="convertToInteractive('df-4f03d62c-db6f-4120-ae56-24f95a7987a2')"
                  title="Convert this dataframe to an interactive table."
                  style="display:none;">
    
      <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
           width="24px">
        <path d="M0 0h24v24H0V0z" fill="none"/>
        <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
      </svg>
          </button>
    
      <style>
        .colab-df-container {
          display:flex;
          flex-wrap:wrap;
          gap: 12px;
        }
    
        .colab-df-convert {
          background-color: #E8F0FE;
          border: none;
          border-radius: 50%;
          cursor: pointer;
          display: none;
          fill: #1967D2;
          height: 32px;
          padding: 0 0 0 0;
          width: 32px;
        }
    
        .colab-df-convert:hover {
          background-color: #E2EBFA;
          box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
          fill: #174EA6;
        }
    
        [theme=dark] .colab-df-convert {
          background-color: #3B4455;
          fill: #D2E3FC;
        }
    
        [theme=dark] .colab-df-convert:hover {
          background-color: #434B5C;
          box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
          filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
          fill: #FFFFFF;
        }
      </style>
    
          <script>
            const buttonEl =
              document.querySelector('#df-4f03d62c-db6f-4120-ae56-24f95a7987a2 button.colab-df-convert');
            buttonEl.style.display =
              google.colab.kernel.accessAllowed ? 'block' : 'none';
    
            async function convertToInteractive(key) {
              const element = document.querySelector('#df-4f03d62c-db6f-4120-ae56-24f95a7987a2');
              const dataTable =
                await google.colab.kernel.invokeFunction('convertToInteractive',
                                                         [key], {});
              if (!dataTable) return;
    
              const docLinkHtml = 'Like what you see? Visit the ' +
                '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
                + ' to learn more about interactive tables.';
              element.innerHTML = '';
              dataTable['output_type'] = 'display_data';
              await google.colab.output.renderOutput(dataTable, element);
              const docLink = document.createElement('div');
              docLink.innerHTML = docLinkHtml;
              element.appendChild(docLink);
            }
          </script>
        </div>
      </div>
    



.. code:: ipython3

    RFM_df.CustAccountBalance.max()




.. parsed-literal::

    146.174071799302

.. autofunction:: scriptNotebook.plot_boxplot

.. code:: ipython3

    def plot_boxplot(df, columns):
        """
        Crea un gráfico de boxplot para las columnas especificadas de un DataFrame normalizado.
    
        Args:
            df (pandas.DataFrame): El DataFrame a graficar.
            columns (list): Lista de nombres de columnas para graficar.
    
        Returns:
            None
        """
        df_norm = pd.DataFrame(df, columns=columns)
        fig, ax = plt.subplots(figsize=(10,5))
        sns.boxplot(data=df_norm, ax=ax, palette="Paired")
        plt.title("Boxplot de las features normalizadas")
        plt.show()
    
    plot_boxplot(RFM_df, columns=['TransactionAmount', 'CustAccountBalance'])



.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_101_0.png

.. autofunction:: scriptNotebook.encode_units

.. code:: ipython3

    def encode_units(x):
        """
        Encode units to binary values.
    
        Args:
          x: input unit.
    
        Returns:
          int: 1 if x >= 1, 0 otherwise.
        """
        if x <= 0:
          return 0
        if x >= 1:
          return 1

.. autofunction:: scriptNotebook.get_frequent_itemsets

.. code:: ipython3

    def get_frequent_itemsets(df):
        """
        Apply apriori algorithm on input DataFrame to obtain frequent itemsets.
    
        Args:
            df (pd.DataFrame): Input DataFrame.
    
        Returns:
            pd.DataFrame: Dataframe with frequent itemsets and their support.
    
        """
        RFM_df_boolean = df.applymap(encode_units)
        frequent_itemsets = apriori(RFM_df_boolean.fillna(0), min_support=0.05, use_colnames=True)
    
        return frequent_itemsets
    
    get_frequent_itemsets(RFM_df)




.. raw:: html

    
      <div id="df-8e4ace5f-a88e-40e6-8f6f-37d808d6f928">
        <div class="colab-df-container">
          <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>support</th>
          <th>itemsets</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>0.153616</td>
          <td>(Frequency)</td>
        </tr>
        <tr>
          <th>1</th>
          <td>0.158243</td>
          <td>(CustLocation)</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.144920</td>
          <td>(TransactionTime)</td>
        </tr>
        <tr>
          <th>3</th>
          <td>0.123739</td>
          <td>(CustomerAge)</td>
        </tr>
      </tbody>
    </table>
    </div>
          <button class="colab-df-convert" onclick="convertToInteractive('df-8e4ace5f-a88e-40e6-8f6f-37d808d6f928')"
                  title="Convert this dataframe to an interactive table."
                  style="display:none;">
    
      <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
           width="24px">
        <path d="M0 0h24v24H0V0z" fill="none"/>
        <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
      </svg>
          </button>
    
      <style>
        .colab-df-container {
          display:flex;
          flex-wrap:wrap;
          gap: 12px;
        }
    
        .colab-df-convert {
          background-color: #E8F0FE;
          border: none;
          border-radius: 50%;
          cursor: pointer;
          display: none;
          fill: #1967D2;
          height: 32px;
          padding: 0 0 0 0;
          width: 32px;
        }
    
        .colab-df-convert:hover {
          background-color: #E2EBFA;
          box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
          fill: #174EA6;
        }
    
        [theme=dark] .colab-df-convert {
          background-color: #3B4455;
          fill: #D2E3FC;
        }
    
        [theme=dark] .colab-df-convert:hover {
          background-color: #434B5C;
          box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
          filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
          fill: #FFFFFF;
        }
      </style>
    
          <script>
            const buttonEl =
              document.querySelector('#df-8e4ace5f-a88e-40e6-8f6f-37d808d6f928 button.colab-df-convert');
            buttonEl.style.display =
              google.colab.kernel.accessAllowed ? 'block' : 'none';
    
            async function convertToInteractive(key) {
              const element = document.querySelector('#df-8e4ace5f-a88e-40e6-8f6f-37d808d6f928');
              const dataTable =
                await google.colab.kernel.invokeFunction('convertToInteractive',
                                                         [key], {});
              if (!dataTable) return;
    
              const docLinkHtml = 'Like what you see? Visit the ' +
                '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
                + ' to learn more about interactive tables.';
              element.innerHTML = '';
              dataTable['output_type'] = 'display_data';
              await google.colab.output.renderOutput(dataTable, element);
              const docLink = document.createElement('div');
              docLink.innerHTML = docLinkHtml;
              element.appendChild(docLink);
            }
          </script>
        </div>
      </div>
    



Now we selected the best features to start to build and fit our models

.. code:: ipython3

    RFM_df_importance = RFM_df[["Frequency","CustLocation", "CustGender", "CustAccountBalance", "TransactionAmount", "CustomerAge"]]

6. Unsupervised learning modeling
---------------------------------

K-mean
~~~~~~

The k-means clustering is a method of vector quantization, originally
from signal processing, that

1. The Elbow method Is a graphical representation of finding the optimal
   ‘K’ in a K-means cluster

2. The Silhouette Coefficient Is a metric used to calculate the goodness
   of a clustering technique. Its value ranges from -1 to 1. 1: Means
   clusters are well apart from each other and clearly distinguished

3. The Dendrogram Is a diagram that shows the hierarchical relationship
   between objects. It is most commonly created as an output from
   hierarchical clustering

.. autofunction:: scriptNotebook.calculate_sse

.. code:: ipython3

    def calculate_sse(data):
        """
        Calculates the sum of squared errors (SSE) for a range of k values (number of clusters)
        using the KMeans algorithm and plots the resulting SSE values.
    
        Args:
          data: A pandas DataFrame with the input data for KMeans clustering.
    
        Returns:
          A dictionary with the SSE values for each k value.
        """
        sse = {}
        for k in range(1, 12, 2):
            kmeans = KMeans(n_clusters=k, random_state=42, n_init='auto')
            kmeans.fit(data)
            sse[k] = kmeans.inertia_
        plt.title("The Elbow Method")
        plt.xlabel('k')
        plt.ylabel('SSE')
        sns.pointplot(x=list(sse.keys()), y=list(sse.values()))
        plt.show()
    
    calculate_sse(RFM_df_importance)



.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_108_0.png


.. raw:: html

   <h4>

Busqueda de hiperparametros K

.. raw:: html

   </h4>

El objetivo es encontrar un punto donde la adición de un cluster
adicional ya no contribuya significativamente a la variación explicada

.. autofunction:: scriptNotebook.elbowFindValue

.. code:: ipython3

    def elbowFindValue(RFM_df_importance):
        """
        This function fits a KMeans model and generates an elbow plot to help identify the optimal number of clusters.
        The function returns the optimal number of clusters identified by the elbow plot.  
      
        Args:
          df (pandas.DataFrame): DataFrame with the data to fit the model to.
        
        Returns:
          none
        """
        model = KMeans(init = 'random',n_init='auto', random_state = 42)
        visualizer = KElbowVisualizer(model, k=(1,12), timings=False)
        visualizer2 = KElbowVisualizer(model, k=(1,12), timings=False)
        visualizer.fit(RFM_df_importance)
        visualizer.show()
    
    elbowFindValue(RFM_df_importance)



.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_110_0.png




.. parsed-literal::

    <Axes: title={'center': 'Distortion Score Elbow for KMeans Clustering'}, xlabel='k', ylabel='distortion score'>



.. code:: ipython3

    pipeline = Pipeline([
        ('kmeans', KMeans(init='random', n_init='auto', random_state=42)),
        ('grid_search', GridSearchCV(KMeans(random_state=42, n_init='auto'), param_grid={'n_clusters': range(3,6,2)}, cv=5)),
    ])
    
    pipeline.fit(RFM_df_importance)
    
    print("Mejor número de clusters: ", pipeline['grid_search'].best_params_['n_clusters'])
    print("Mejor score: ", pipeline['grid_search'].best_score_)


.. parsed-literal::

    Mejor número de clusters:  5
    Mejor score:  -10684588.976592684
    

Primer modelo con K = 3
^^^^^^^^^^^^^^^^^^^^^^^

Like, we got two possible values to K, we are going to try with the
first value, in this case the value will be 3

.. code:: ipython3

    model1 = KMeans(n_clusters=3, random_state=42, n_init='auto')
    model1.fit(RFM_df_importance)




.. raw:: html

    <style>#sk-container-id-1 {color: black;background-color: white;}#sk-container-id-1 pre{padding: 0;}#sk-container-id-1 div.sk-toggleable {background-color: white;}#sk-container-id-1 label.sk-toggleable__label {cursor: pointer;display: block;width: 100%;margin-bottom: 0;padding: 0.3em;box-sizing: border-box;text-align: center;}#sk-container-id-1 label.sk-toggleable__label-arrow:before {content: "▸";float: left;margin-right: 0.25em;color: #696969;}#sk-container-id-1 label.sk-toggleable__label-arrow:hover:before {color: black;}#sk-container-id-1 div.sk-estimator:hover label.sk-toggleable__label-arrow:before {color: black;}#sk-container-id-1 div.sk-toggleable__content {max-height: 0;max-width: 0;overflow: hidden;text-align: left;background-color: #f0f8ff;}#sk-container-id-1 div.sk-toggleable__content pre {margin: 0.2em;color: black;border-radius: 0.25em;background-color: #f0f8ff;}#sk-container-id-1 input.sk-toggleable__control:checked~div.sk-toggleable__content {max-height: 200px;max-width: 100%;overflow: auto;}#sk-container-id-1 input.sk-toggleable__control:checked~label.sk-toggleable__label-arrow:before {content: "▾";}#sk-container-id-1 div.sk-estimator input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-1 div.sk-label input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-1 input.sk-hidden--visually {border: 0;clip: rect(1px 1px 1px 1px);clip: rect(1px, 1px, 1px, 1px);height: 1px;margin: -1px;overflow: hidden;padding: 0;position: absolute;width: 1px;}#sk-container-id-1 div.sk-estimator {font-family: monospace;background-color: #f0f8ff;border: 1px dotted black;border-radius: 0.25em;box-sizing: border-box;margin-bottom: 0.5em;}#sk-container-id-1 div.sk-estimator:hover {background-color: #d4ebff;}#sk-container-id-1 div.sk-parallel-item::after {content: "";width: 100%;border-bottom: 1px solid gray;flex-grow: 1;}#sk-container-id-1 div.sk-label:hover label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-1 div.sk-serial::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: 0;}#sk-container-id-1 div.sk-serial {display: flex;flex-direction: column;align-items: center;background-color: white;padding-right: 0.2em;padding-left: 0.2em;position: relative;}#sk-container-id-1 div.sk-item {position: relative;z-index: 1;}#sk-container-id-1 div.sk-parallel {display: flex;align-items: stretch;justify-content: center;background-color: white;position: relative;}#sk-container-id-1 div.sk-item::before, #sk-container-id-1 div.sk-parallel-item::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: -1;}#sk-container-id-1 div.sk-parallel-item {display: flex;flex-direction: column;z-index: 1;position: relative;background-color: white;}#sk-container-id-1 div.sk-parallel-item:first-child::after {align-self: flex-end;width: 50%;}#sk-container-id-1 div.sk-parallel-item:last-child::after {align-self: flex-start;width: 50%;}#sk-container-id-1 div.sk-parallel-item:only-child::after {width: 0;}#sk-container-id-1 div.sk-dashed-wrapped {border: 1px dashed gray;margin: 0 0.4em 0.5em 0.4em;box-sizing: border-box;padding-bottom: 0.4em;background-color: white;}#sk-container-id-1 div.sk-label label {font-family: monospace;font-weight: bold;display: inline-block;line-height: 1.2em;}#sk-container-id-1 div.sk-label-container {text-align: center;}#sk-container-id-1 div.sk-container {/* jupyter's `normalize.less` sets `[hidden] { display: none; }` but bootstrap.min.css set `[hidden] { display: none !important; }` so we also need the `!important` here to be able to override the default hidden behavior on the sphinx rendered scikit-learn.org. See: https://github.com/scikit-learn/scikit-learn/issues/21755 */display: inline-block !important;position: relative;}#sk-container-id-1 div.sk-text-repr-fallback {display: none;}</style><div id="sk-container-id-1" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>KMeans(n_clusters=3, n_init=&#x27;auto&#x27;, random_state=42)</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-1" type="checkbox" checked><label for="sk-estimator-id-1" class="sk-toggleable__label sk-toggleable__label-arrow">KMeans</label><div class="sk-toggleable__content"><pre>KMeans(n_clusters=3, n_init=&#x27;auto&#x27;, random_state=42)</pre></div></div></div></div></div>


.. autofunction:: scriptNotebook.groupByCluster3

.. code:: ipython3

    def groupByCluster3(RFM_df_importance, model1):
        """
        This function generates a summary of the clusters generated by a KMeans model.
        
        Args:
          df (pandas.DataFrame): DataFrame with the data used to generate the clusters.
          model1: model of the kmeans fit
        Returns:
          RFM_df_kmeans3: the new Dataset
        """
        RFM_df_kmeans3 = RFM_df_importance
        RFM_df_kmeans3["Cluster"] = model1.labels_
        RFM_df_kmeans3.groupby("Cluster").agg({
          "Frequency": "mean",
          "CustGender": "mean",
          "CustAccountBalance": "mean",
          "TransactionAmount": "mean",
          "CustomerAge" : "mean"
        }).round(2)
    
        return RFM_df_kmeans3
      
    RFM_df_kmeans3 = groupByCluster3(RFM_df_importance, model1)


.. parsed-literal::

    <ipython-input-64-e30aa4e1c3d6>:2: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      RFM_df_kmeans3["Cluster"] = model1.labels_
    



.. raw:: html

    
      <div id="df-8d1bceea-5e72-4ed4-90f5-6a58a8c5f67f">
        <div class="colab-df-container">
          <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Frequency</th>
          <th>CustGender</th>
          <th>CustAccountBalance</th>
          <th>TransactionAmount</th>
          <th>CustomerAge</th>
        </tr>
        <tr>
          <th>Cluster</th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>0.03</td>
          <td>-0.28</td>
          <td>1.23</td>
          <td>1.08</td>
          <td>0.29</td>
        </tr>
        <tr>
          <th>1</th>
          <td>0.00</td>
          <td>-0.11</td>
          <td>727.21</td>
          <td>5.56</td>
          <td>2.37</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.03</td>
          <td>-0.21</td>
          <td>114.77</td>
          <td>3.25</td>
          <td>1.65</td>
        </tr>
      </tbody>
    </table>
    </div>
          <button class="colab-df-convert" onclick="convertToInteractive('df-8d1bceea-5e72-4ed4-90f5-6a58a8c5f67f')"
                  title="Convert this dataframe to an interactive table."
                  style="display:none;">
    
      <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
           width="24px">
        <path d="M0 0h24v24H0V0z" fill="none"/>
        <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
      </svg>
          </button>
    
      <style>
        .colab-df-container {
          display:flex;
          flex-wrap:wrap;
          gap: 12px;
        }
    
        .colab-df-convert {
          background-color: #E8F0FE;
          border: none;
          border-radius: 50%;
          cursor: pointer;
          display: none;
          fill: #1967D2;
          height: 32px;
          padding: 0 0 0 0;
          width: 32px;
        }
    
        .colab-df-convert:hover {
          background-color: #E2EBFA;
          box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
          fill: #174EA6;
        }
    
        [theme=dark] .colab-df-convert {
          background-color: #3B4455;
          fill: #D2E3FC;
        }
    
        [theme=dark] .colab-df-convert:hover {
          background-color: #434B5C;
          box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
          filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
          fill: #FFFFFF;
        }
      </style>
    
          <script>
            const buttonEl =
              document.querySelector('#df-8d1bceea-5e72-4ed4-90f5-6a58a8c5f67f button.colab-df-convert');
            buttonEl.style.display =
              google.colab.kernel.accessAllowed ? 'block' : 'none';
    
            async function convertToInteractive(key) {
              const element = document.querySelector('#df-8d1bceea-5e72-4ed4-90f5-6a58a8c5f67f');
              const dataTable =
                await google.colab.kernel.invokeFunction('convertToInteractive',
                                                         [key], {});
              if (!dataTable) return;
    
              const docLinkHtml = 'Like what you see? Visit the ' +
                '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
                + ' to learn more about interactive tables.';
              element.innerHTML = '';
              dataTable['output_type'] = 'display_data';
              await google.colab.output.renderOutput(dataTable, element);
              const docLink = document.createElement('div');
              docLink.innerHTML = docLinkHtml;
              element.appendChild(docLink);
            }
          </script>
        </div>
      </div>
    



We are going to do the scatter plot to view the relation between the
CustAccountBalance and TransactionAmount

.. code:: ipython3

    sns.scatterplot(data=RFM_df_importance ,x='CustAccountBalance',y='TransactionAmount',hue=model1.labels_,s=300,alpha=0.6,palette='summer')




.. parsed-literal::

    <Axes: xlabel='CustAccountBalance', ylabel='TransactionAmount'>




.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_117_1.png

.. autofunction:: scriptNotebook.viewTheClustersKmeans3

.. code:: ipython3

    def viewTheClustersKmeans3(RFM_df_kmeans3):
        """
        This function generate the plot with the clusters
        
        Args:
          df (pandas.DataFrame): DataFrame with the data used to generate the clusters.
        
        Returns:
          none
        """
    
        df_n = pd.DataFrame(RFM_df_kmeans3, columns=["Frequency", "CustGender", "CustLocation", "CustAccountBalance", "TransactionAmount", "CustomerAge"])
    
        df_n["ID"] = RFM_df_kmeans3.index
        df_n["Cluster"] = model1.labels_
    
        df_nor_melt = pd.melt(df_n.reset_index(),
                          id_vars=['ID', 'Cluster'],
                          value_vars=["CustAccountBalance", "TransactionAmount"],
                          var_name='Attribute',
                          value_name='Value')
    
        df_nor_melt2 = pd.melt(df_n.reset_index(),
                          id_vars=['ID', 'Cluster'],
                          value_vars=[ "CustGender", "CustomerAge"],
                          var_name='Attribute',
                          value_name='Value')
    
        fig, (ax1, ax2) = plt.subplots(ncols=2, figsize=(12, 6))
    
        sns.lineplot(x='Attribute', y='Value', hue='Cluster', data=df_nor_melt, ax=ax1)
        ax1.set_xticklabels(ax1.get_xticklabels(), rotation=45)
        ax1.set_title('Balance y Transaciones')
    
        sns.lineplot(x='Attribute', y='Value', hue='Cluster', data=df_nor_melt2, ax=ax2)
        ax2.set_xticklabels(ax2.get_xticklabels(), rotation=45)
        ax2.set_title('Genero y edad')
    
        plt.show()
    
    viewTheClustersKmeans3(RFM_df_kmeans3)


.. parsed-literal::

    <ipython-input-66-c2633c6af93d>:21: UserWarning: FixedFormatter should only be used together with FixedLocator
      ax1.set_xticklabels(ax1.get_xticklabels(), rotation=45)
    <ipython-input-66-c2633c6af93d>:25: UserWarning: FixedFormatter should only be used together with FixedLocator
      ax2.set_xticklabels(ax2.get_xticklabels(), rotation=45)
    


.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_118_1.png


El algoritmo de agrupación espectral ha dividido a los clientes en tres
grupos distintos. El **primer** grupo está formado por los clientes
dinámicos 👥 que tienen un saldo de cuenta más bajo y suelen gastar
menos efectivo en las transacciones. El **segundo** son mujeres de entre
20 y 30 años que realizan transacciones de gran valor 👩🏻🛍️. El
**tercer** grupo de hombres de entre 30 y 40 años 👨🏻💼 que trabajan y
guardan dinero en su cuenta para una posible inversión.

.. code:: ipython3

    dataframeToArray = RFM_df_importance.to_numpy()

Now, once we have the cluster graph, it is time to test how good was the
chosen model, for this we will rely on the Silhouette Score, this metric
is based on intra-cluster similarity (how similar are the points within
a cluster) and inter-cluster dissimilarity (how different are the points
between clusters).

The Silhouette Score measures how well separated the clusters are, and
ranges from -1 to 1. A high score (near 1) indicates that points within
a cluster are very similar to each other and very different from points
in other clusters, while a low score (near -1) indicates the opposite,
i.e., that clusters are very mixed and points within a cluster are
similar to points in other clusters.

.. code:: ipython3

    silhouette = silhouette_score(dataframeToArray, model1.labels_)
    print("Silhouette Score:",str(np.round(silhouette*100,2)) + '%')


.. parsed-literal::

    Silhouette Score: 95.53%
    

Segundo modelo con K = 5
^^^^^^^^^^^^^^^^^^^^^^^^

Now, just as we tested with a K=3, it is now the turn to test with a
K=5.

.. code:: ipython3

    model2 = KMeans(n_clusters=5, random_state=42, n_init='auto')
    model2.fit(RFM_df_importance)




.. raw:: html

    <style>#sk-container-id-2 {color: black;background-color: white;}#sk-container-id-2 pre{padding: 0;}#sk-container-id-2 div.sk-toggleable {background-color: white;}#sk-container-id-2 label.sk-toggleable__label {cursor: pointer;display: block;width: 100%;margin-bottom: 0;padding: 0.3em;box-sizing: border-box;text-align: center;}#sk-container-id-2 label.sk-toggleable__label-arrow:before {content: "▸";float: left;margin-right: 0.25em;color: #696969;}#sk-container-id-2 label.sk-toggleable__label-arrow:hover:before {color: black;}#sk-container-id-2 div.sk-estimator:hover label.sk-toggleable__label-arrow:before {color: black;}#sk-container-id-2 div.sk-toggleable__content {max-height: 0;max-width: 0;overflow: hidden;text-align: left;background-color: #f0f8ff;}#sk-container-id-2 div.sk-toggleable__content pre {margin: 0.2em;color: black;border-radius: 0.25em;background-color: #f0f8ff;}#sk-container-id-2 input.sk-toggleable__control:checked~div.sk-toggleable__content {max-height: 200px;max-width: 100%;overflow: auto;}#sk-container-id-2 input.sk-toggleable__control:checked~label.sk-toggleable__label-arrow:before {content: "▾";}#sk-container-id-2 div.sk-estimator input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-2 div.sk-label input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-2 input.sk-hidden--visually {border: 0;clip: rect(1px 1px 1px 1px);clip: rect(1px, 1px, 1px, 1px);height: 1px;margin: -1px;overflow: hidden;padding: 0;position: absolute;width: 1px;}#sk-container-id-2 div.sk-estimator {font-family: monospace;background-color: #f0f8ff;border: 1px dotted black;border-radius: 0.25em;box-sizing: border-box;margin-bottom: 0.5em;}#sk-container-id-2 div.sk-estimator:hover {background-color: #d4ebff;}#sk-container-id-2 div.sk-parallel-item::after {content: "";width: 100%;border-bottom: 1px solid gray;flex-grow: 1;}#sk-container-id-2 div.sk-label:hover label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-2 div.sk-serial::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: 0;}#sk-container-id-2 div.sk-serial {display: flex;flex-direction: column;align-items: center;background-color: white;padding-right: 0.2em;padding-left: 0.2em;position: relative;}#sk-container-id-2 div.sk-item {position: relative;z-index: 1;}#sk-container-id-2 div.sk-parallel {display: flex;align-items: stretch;justify-content: center;background-color: white;position: relative;}#sk-container-id-2 div.sk-item::before, #sk-container-id-2 div.sk-parallel-item::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: -1;}#sk-container-id-2 div.sk-parallel-item {display: flex;flex-direction: column;z-index: 1;position: relative;background-color: white;}#sk-container-id-2 div.sk-parallel-item:first-child::after {align-self: flex-end;width: 50%;}#sk-container-id-2 div.sk-parallel-item:last-child::after {align-self: flex-start;width: 50%;}#sk-container-id-2 div.sk-parallel-item:only-child::after {width: 0;}#sk-container-id-2 div.sk-dashed-wrapped {border: 1px dashed gray;margin: 0 0.4em 0.5em 0.4em;box-sizing: border-box;padding-bottom: 0.4em;background-color: white;}#sk-container-id-2 div.sk-label label {font-family: monospace;font-weight: bold;display: inline-block;line-height: 1.2em;}#sk-container-id-2 div.sk-label-container {text-align: center;}#sk-container-id-2 div.sk-container {/* jupyter's `normalize.less` sets `[hidden] { display: none; }` but bootstrap.min.css set `[hidden] { display: none !important; }` so we also need the `!important` here to be able to override the default hidden behavior on the sphinx rendered scikit-learn.org. See: https://github.com/scikit-learn/scikit-learn/issues/21755 */display: inline-block !important;position: relative;}#sk-container-id-2 div.sk-text-repr-fallback {display: none;}</style><div id="sk-container-id-2" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>KMeans(n_clusters=5, n_init=&#x27;auto&#x27;, random_state=42)</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-2" type="checkbox" checked><label for="sk-estimator-id-2" class="sk-toggleable__label sk-toggleable__label-arrow">KMeans</label><div class="sk-toggleable__content"><pre>KMeans(n_clusters=5, n_init=&#x27;auto&#x27;, random_state=42)</pre></div></div></div></div></div>

.. autofunction:: scriptNotebook.groupByCluster5

.. code:: ipython3

    def groupByCluster5(RFM_df_importance, model2):
        """
        This function generates a summary of the clusters generated by a KMeans model.
        
        Args:
          df (pandas.DataFrame): DataFrame with the data used to generate the clusters.
          model2: model of the kmeans fit
        
        Returns:
          RFM_df_kmeans5: the new Dataset
        """
        RFM_df_kmeans5 = RFM_df_importance
        RFM_df_kmeans5["Cluster"] = model2.labels_
        RFM_df_kmeans5.groupby("Cluster").agg({
          "Frequency": "mean",
          "CustGender": "mean",
          "CustAccountBalance": "mean",
          "TransactionAmount": "mean",
          "CustomerAge" : "mean"
        }).round(2)
    
        return RFM_df_kmeans5
      
    RFM_df_kmeans5 = groupByCluster5(RFM_df_importance, model2)


.. parsed-literal::

    <ipython-input-70-8441070a4c68>:2: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      RFM_df_kmeans5["Cluster"] = model2.labels_
    



.. raw:: html

    
      <div id="df-ebd210d0-57b5-46f8-be42-7aa8958890b2">
        <div class="colab-df-container">
          <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>Frequency</th>
          <th>CustGender</th>
          <th>CustAccountBalance</th>
          <th>TransactionAmount</th>
          <th>CustomerAge</th>
        </tr>
        <tr>
          <th>Cluster</th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>0.03</td>
          <td>-0.28</td>
          <td>1.00</td>
          <td>1.02</td>
          <td>0.28</td>
        </tr>
        <tr>
          <th>1</th>
          <td>0.00</td>
          <td>-0.16</td>
          <td>764.74</td>
          <td>7.01</td>
          <td>2.68</td>
        </tr>
        <tr>
          <th>2</th>
          <td>0.02</td>
          <td>-0.25</td>
          <td>53.88</td>
          <td>7.83</td>
          <td>1.44</td>
        </tr>
        <tr>
          <th>3</th>
          <td>0.00</td>
          <td>0.00</td>
          <td>2419.98</td>
          <td>6.75</td>
          <td>2.33</td>
        </tr>
        <tr>
          <th>4</th>
          <td>0.03</td>
          <td>-0.25</td>
          <td>263.25</td>
          <td>4.26</td>
          <td>1.78</td>
        </tr>
      </tbody>
    </table>
    </div>
          <button class="colab-df-convert" onclick="convertToInteractive('df-ebd210d0-57b5-46f8-be42-7aa8958890b2')"
                  title="Convert this dataframe to an interactive table."
                  style="display:none;">
    
      <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
           width="24px">
        <path d="M0 0h24v24H0V0z" fill="none"/>
        <path d="M18.56 5.44l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94zm-11 1L8.5 8.5l.94-2.06 2.06-.94-2.06-.94L8.5 2.5l-.94 2.06-2.06.94zm10 10l.94 2.06.94-2.06 2.06-.94-2.06-.94-.94-2.06-.94 2.06-2.06.94z"/><path d="M17.41 7.96l-1.37-1.37c-.4-.4-.92-.59-1.43-.59-.52 0-1.04.2-1.43.59L10.3 9.45l-7.72 7.72c-.78.78-.78 2.05 0 2.83L4 21.41c.39.39.9.59 1.41.59.51 0 1.02-.2 1.41-.59l7.78-7.78 2.81-2.81c.8-.78.8-2.07 0-2.86zM5.41 20L4 18.59l7.72-7.72 1.47 1.35L5.41 20z"/>
      </svg>
          </button>
    
      <style>
        .colab-df-container {
          display:flex;
          flex-wrap:wrap;
          gap: 12px;
        }
    
        .colab-df-convert {
          background-color: #E8F0FE;
          border: none;
          border-radius: 50%;
          cursor: pointer;
          display: none;
          fill: #1967D2;
          height: 32px;
          padding: 0 0 0 0;
          width: 32px;
        }
    
        .colab-df-convert:hover {
          background-color: #E2EBFA;
          box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
          fill: #174EA6;
        }
    
        [theme=dark] .colab-df-convert {
          background-color: #3B4455;
          fill: #D2E3FC;
        }
    
        [theme=dark] .colab-df-convert:hover {
          background-color: #434B5C;
          box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
          filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
          fill: #FFFFFF;
        }
      </style>
    
          <script>
            const buttonEl =
              document.querySelector('#df-ebd210d0-57b5-46f8-be42-7aa8958890b2 button.colab-df-convert');
            buttonEl.style.display =
              google.colab.kernel.accessAllowed ? 'block' : 'none';
    
            async function convertToInteractive(key) {
              const element = document.querySelector('#df-ebd210d0-57b5-46f8-be42-7aa8958890b2');
              const dataTable =
                await google.colab.kernel.invokeFunction('convertToInteractive',
                                                         [key], {});
              if (!dataTable) return;
    
              const docLinkHtml = 'Like what you see? Visit the ' +
                '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
                + ' to learn more about interactive tables.';
              element.innerHTML = '';
              dataTable['output_type'] = 'display_data';
              await google.colab.output.renderOutput(dataTable, element);
              const docLink = document.createElement('div');
              docLink.innerHTML = docLinkHtml;
              element.appendChild(docLink);
            }
          </script>
        </div>
      </div>
    



Generate our scatter plot

.. code:: ipython3

    sns.scatterplot(data=RFM_df_importance,x='CustAccountBalance',y='TransactionAmount',hue=model2.labels_,s=300,alpha=0.6,palette='summer')




.. parsed-literal::

    <Axes: xlabel='CustAccountBalance', ylabel='TransactionAmount'>




.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_128_1.png

.. autofunction:: scriptNotebook.viewTheClustersKmeans5

.. code:: ipython3

    def viewTheClustersKmeans5(RFM_df_kmeans5):
        """
        This function generate the plot with the clusters
        
        Args:
          df (pandas.DataFrame): DataFrame with the data used to generate the clusters.
      
        Returns:
          none
        """
    
        df_n = pd.DataFrame(RFM_df_kmeans5, columns=["Frequency", "CustGender", "CustLocation", "CustAccountBalance", "TransactionAmount", "CustomerAge"])
    
        df_n["ID"] = RFM_df_kmeans5.index
        df_n["Cluster"] = model2.labels_
    
        df_nor_melt = pd.melt(df_n.reset_index(),
                          id_vars=['ID', 'Cluster'],
                          value_vars=["CustAccountBalance", "TransactionAmount"],
                          var_name='Attribute',
                          value_name='Value')
    
        df_nor_melt2 = pd.melt(df_n.reset_index(),
                          id_vars=['ID', 'Cluster'],
                          value_vars=[ "CustGender", "CustomerAge"],
                          var_name='Attribute',
                          value_name='Value')
    
        fig, (ax1, ax2) = plt.subplots(ncols=2, figsize=(12, 6))
    
        sns.lineplot(x='Attribute', y='Value', hue='Cluster', data=df_nor_melt, ax=ax1)
        ax1.set_xticklabels(ax1.get_xticklabels(), rotation=45)
        ax1.set_title('Balance y Transaciones')
    
        sns.lineplot(x='Attribute', y='Value', hue='Cluster', data=df_nor_melt2, ax=ax2)
        ax2.set_xticklabels(ax2.get_xticklabels(), rotation=45)
        ax2.set_title('Genero y edad')
    
        plt.show()
    
    viewTheClustersKmeans5(RFM_df_kmeans5)


.. parsed-literal::

    <ipython-input-253-13b1c280d5a4>:21: UserWarning: FixedFormatter should only be used together with FixedLocator
      ax1.set_xticklabels(ax1.get_xticklabels(), rotation=45)
    <ipython-input-253-13b1c280d5a4>:25: UserWarning: FixedFormatter should only be used together with FixedLocator
      ax2.set_xticklabels(ax2.get_xticklabels(), rotation=45)
    


.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_129_1.png


We see our Silhouette Score:

.. code:: ipython3

    silhouette = silhouette_score(dataframeToArray,model2.labels_)
    print("Silhouette Score:",str(np.round(silhouette*100,2)) + '%')


.. parsed-literal::

    Silhouette Score: 91.95%
    

The Spectral clustering algorithm has segregated the customers into five
distinct groups. The **first** group comprises the dynamic customers 👥
who have lower account balance and mostly expend less cash on
transactions. The **second** women between 20 and 30 who perform
large-value transactions 👩🏻🛍️. The **third** man between 30 and 40 👨🏻💼
who work and keep money in their account for a possible investment.The
**Four** and **Five** group includes the more conservative and money
saving-minded people 🧓🏻👴🏻 who inspite of having really high account
balances, spend the least amount of money in transactions, thereby
judiciously maintaining their savings accounts.

Find number of sample due to compute limitations
------------------------------------------------

The method used to calculate the required sample size is called the
“simple random sample formula”. This formula is widely used to determine
the sample size needed for a given population, and is an important tool
in the design of research studies and surveys.

.. autofunction:: scriptNotebook.calculateSample

.. code:: ipython3

    import math
    
    def calculateSample(RFM_df_importance):
        """
        This function calculates the sample size needed based on the number of observations in the `RFM_df_importance` dataset.
    
        Args:
          none
    
        Returns:
           none
        """
        N = len(RFM_df_importance)  # n
        e = 0.01  # margen de error deseado
        n = N / (1 + N * e**2)
        n = math.ceil(n)  # round up
    
        print(n)  # tamaño de la muestra necesario
    
    calculateSample(RFM_df_importance)


.. parsed-literal::

    9883
    

Hierarchical clustering
~~~~~~~~~~~~~~~~~~~~~~~

Hierarchical clustering is where you build a dendrogram to represent
data, where each group links to two or more successor groups. The groups
are nested and organized as a tree, which ideally ends up as a
meaningful classification scheme.

Each node in the cluster tree contains a group of similar data; Nodes
group on the graph next to other, similar nodes. Clusters at one level
join with clusters in the next level up, using a degree of similarity.
The process carries on until all nodes are in the tree, which gives a
visual snapshot of the data contained in the whole set. The total number
of clusters is not predetermined before you start the tree creation.

Hierarchical clustering based on customer’s age
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. autofunction:: scriptNotebook.hierarchicalClusteringAge

.. code:: ipython3

    import scipy.cluster.hierarchy as sch
    
    customerAgeSample = None
    
    def hierarchicalClusteringAge(RFM_df_importance):
        """
          Perform hierarchical clustering on the age and account balance variables in the input DataFrame using Ward's method.
        
        Args:
          RFM_df_importance : pandas DataFrame
            A DataFrame containing customer information, including the 'CustomerAge' and 'CustAccountBalance' columns.
    
        Returns:
          dendogram_age : dict
            A dictionary containing information about the dendrogram visualization of the hierarchical clustering. The keys are 'icoord', 'dcoord', 'leaves', and 'color_list', which correspond to the output of scipy's dendrogram function.
        """
        global customerAgeSample
        customerAgeSample = pd.DataFrame(RFM_df_importance[['CustomerAge', 'CustAccountBalance']].values).sample(10000)
        plt.figure(figsize=(20, 10))
        dendogram_age = sch.dendrogram(sch.linkage(customerAgeSample, method='ward'))
        plt.title("Dendogram")
        plt.xlabel("Customers")
        plt.ylabel("Euclidean distances")
        plt.show()
        return dendogram_age
    
    dendogram_age = hierarchicalClusteringAge(RFM_df_importance)



.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_138_0.png

.. autofunction:: scriptNotebook.getBestClustersAge

.. code:: ipython3

    def getBestClustersAge(dendogram_age):
        """
        Determines the optimal number of clusters based on a dendrogram obtained through hierarchical clustering.
    
        Args:
          dendogram_age (dict): A dictionary containing the dendrogram information.
    
        Returns:
          num_clusters_age (int): The optimal number of clusters.
        """
        unique_colors_age = set(dendogram_age['color_list'])
        num_clusters_age = len(unique_colors_age)-1
        print("Optimal number of clusters is : ", num_clusters_age)
        return num_clusters_age
    
    num_clusters_age = getBestClustersAge(dendogram_age)


.. parsed-literal::

    Optimal number of clusters is :  2
    

Calculate the Estimated number of noise points

.. code:: ipython3

    cluster_age = AgglomerativeClustering(n_clusters=num_clusters_age, affinity='euclidean', linkage='ward')
    cluster_age.fit_predict(customerAgeSample)
    print("Estimated number of noise points:", list(cluster_age.labels_).count(-1))


.. parsed-literal::

    /usr/local/lib/python3.10/dist-packages/sklearn/cluster/_agglomerative.py:983: FutureWarning: Attribute `affinity` was deprecated in version 1.2 and will be removed in 1.4. Use `metric` instead
      warnings.warn(
    

.. parsed-literal::

    Estimated number of noise points: 0
    

We make the scatter plot

.. code:: ipython3

    plt.figure(num=None, figsize=(20, 10), facecolor='w', edgecolor='k')
    plt.scatter(customerAgeSample[0], customerAgeSample[1], c=cluster_age.labels_, cmap='viridis');
    plt.ylim(-0.5,6)




.. parsed-literal::

    (-0.5, 6.0)




.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_143_1.png


Now, we are going to see the Silhouette Score

.. code:: ipython3

    silhouette = silhouette_score(customerAgeSample, cluster_age.labels_)
    print("Silhouette Score:", silhouette)


.. parsed-literal::

    Silhouette Score: 0.5690559012363822
    

Hierarchical clustering based on customer’s location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. autofunction:: scriptNotebook.hierarchicalClusteringLocation

.. code:: ipython3

    customerLocationSample = None
    
    def hierarchicalClusteringLocation(RFM_df_importance):
        """
        Perform hierarchical clustering on the age and account balance variables in the input DataFrame using Ward's method.
        
        Args:
          RFM_df_importance : pandas DataFrame
            A DataFrame containing customer information, including the 'CustomerAge' and 'CustAccountBalance' columns.
    
        Returns:
          dendogram_location : dict
            A dictionary containing information about the dendrogram visualization of the hierarchical clustering. The keys are 'icoord', 'dcoord', 'leaves', and 'color_list', which correspond to the output of scipy's dendrogram function.
        """
        global customerLocationSample
        customerLocationSample = pd.DataFrame(RFM_df_importance[['CustLocation', 'CustAccountBalance']].values).sample(10000)
        plt.figure(figsize=(20, 10))
        dendogram_location = sch.dendrogram(sch.linkage(customerLocationSample, method='ward'))
        plt.title("Dendogram")
        plt.xlabel("Customers")
        plt.ylabel("Euclidean distances")
        plt.show()
        return dendogram_location
    
    dendogram_location = hierarchicalClusteringLocation(RFM_df_importance)



.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_147_0.png

.. autofunction:: scriptNotebook.getBestClustersLocation

.. code:: ipython3

    def getBestClustersLocation(dendogram_location):
        """
        Determines the optimal number of clusters based on a dendrogram obtained through hierarchical clustering.
    
        Args:
          dendogram_location (dict): A dictionary containing the dendrogram information.
    
        Returns:
          num_clusters_location (int): The optimal number of clusters.
        """
        unique_colors_location = set(dendogram_location['color_list'])
        num_clusters_location = len(unique_colors_location)-1
        print("Optimal number of clusters is : ", num_clusters_location)
        return num_clusters_location
    
    num_clusters_location = getBestClustersLocation(dendogram_location)


.. parsed-literal::

    Optimal number of clusters is :  3
    

.. code:: ipython3

    cluster_location = AgglomerativeClustering(n_clusters=num_clusters_location, affinity='euclidean', linkage='ward')
    cluster_location.fit_predict(customerLocationSample)
    print("Estimated number of noise points:", list(cluster_location.labels_).count(-1))


.. parsed-literal::

    /usr/local/lib/python3.10/dist-packages/sklearn/cluster/_agglomerative.py:983: FutureWarning: Attribute `affinity` was deprecated in version 1.2 and will be removed in 1.4. Use `metric` instead
      warnings.warn(
    

.. parsed-literal::

    Estimated number of noise points: 0
    

We make the scatter plot

.. code:: ipython3

    plt.figure(num=None, figsize=(20, 10), facecolor='w', edgecolor='k')
    plt.scatter(customerLocationSample[0], customerLocationSample[1], c=cluster_location.labels_, cmap='viridis');
    plt.ylim(-0.5,6)




.. parsed-literal::

    (-0.5, 6.0)




.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_151_1.png


We get our Silhouette Score, this score is better than the score of the
age clusters

.. code:: ipython3

    silhouette = silhouette_score(customerLocationSample, cluster_location.labels_)
    print("Silhouette Score:", silhouette)


.. parsed-literal::

    Silhouette Score: 0.5947098347211797
    

DBSCAN
~~~~~~

Key Characteristics of DBSCAN Algorithm
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  It does not require the number of clusters as input.
-  It is can detect outliers while finding clusters.
-  DBSCAN algorithm can detect clusters that are complex or randomly
   shaped and sized.

In case you do not have the Kneed and Clusteval libraries, this code
installs them.

.. code:: ipython3

    !pip install kneed
    !pip install clusteval


.. autofunction:: scriptNotebook.dbscanInitial

.. code:: ipython3

    temp= RFM_df_importance.sample(10000)
    
    def dbscanInitial(temp):
        """
        Performs DBSCAN clustering on a sample of data using the default settings.
    
        Args:
          temp: pandas DataFrame
            The DataFrame containing the data to be clustered.
    
        Returns:
          None
        """
        base_dbscan = DBSCAN()
        base_dbscan.fit(temp) # Review
        print("Estimated number of clusters:", set(base_dbscan.labels_))
        print("Estimated number of noise points:", list(base_dbscan.labels_).count(-1))
    
    dbscanInitial(temp)


.. parsed-literal::

    Estimated number of clusters: {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, -1}
    Estimated number of noise points: 527
    

We do the dbscan initially without having the hyperparameters well
defined, this is to give us an overview of how good the model is, in the
Silhouette score we can see that it is very low, so we proceed to look
for the best hyperparameters.

.. code:: ipython3

    print("Silhouette Score:", str(np.round(silhouette_score(temp, base_dbscan.labels_) * 100, 2)) + '%')


.. parsed-literal::

    Silhouette Score: 26.83%
    
.. autofunction:: scriptNotebook.get_kdist_plot

.. code:: ipython3

    from kneed import KneeLocator
    
    def get_kdist_plot(X=None, k=None, radius_nbrs=1.0):
        """
        Plots the k-distance graph for a given dataset and returns the distance value at the knee point.
    
        Args:
            X (numpy array or pandas DataFrame): The input dataset.
            k (int): The number of neighbors to consider.
            radius_nbrs (float): The radius of neighbors to consider.
    
        Returns:
            float: The distance value at the knee point.
    
        Raises:
            ValueError: If the input dataset is empty or the number of neighbors is negative.
        """
        nbrs = NearestNeighbors(n_neighbors=k, radius=radius_nbrs).fit(X)
        distances, indices = nbrs.kneighbors(X)                          
        distances = np.sort(distances[:,k-1], axis=0)
        i = np.arange(len(distances))
        knee = KneeLocator(i, distances, S=1, curve='convex', direction='increasing', interp_method='polynomial')
        knee.plot_knee()
        plt.xlabel("Points")
        plt.ylabel("Distance")
        plt.show()
        return distances[knee.knee]
      
    k = 2 * temp.shape[-1] 
    x = get_kdist_plot(temp,k)
    print("Knee Point:",x)



.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_161_0.png


.. parsed-literal::

    Knee Point: 0.5333854057269539
    
.. autofunction:: scriptNotebook.calculateMinimumSamples

.. code:: ipython3

    def calculateMinimumSamples(x):
        """
        Calculates the minimum number of samples required for DBSCAN clustering by evaluating the silhouette score
        for different values of the minimum number of samples parameter.
        
        Args:
          x: float
            The distance threshold value for DBSCAN clustering.
    
        Returns:
          None
        """
        ms = np.arange(3,2*temp.shape[1],3)
        silhouette_scores = []
    
        for i in ms:
          dbscan = DBSCAN(eps=x,min_samples=i)
          dbscan.fit(temp)
          silhouette_scores.append(silhouette_score(temp,dbscan.labels_))
          print("{} Minimum Samples Tested!".format(i))
    
        plt.figure(figsize=(8,4),dpi=150)
        sns.lineplot(x=ms,y=silhouette_scores)
        plt.xlabel('Minimum Number of Samples')
        plt.ylabel('Silhouette Score')
        
    calculateMinimumSamples(x)


.. parsed-literal::

    3 Minimum Samples Tested!
    6 Minimum Samples Tested!
    9 Minimum Samples Tested!
    

Now, let’s see the best Silhouette score for each sample, we can
conclude that the best min_samples is 6





.. parsed-literal::

    Text(0, 0.5, 'Silhouette Score')




.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_164_1.png

.. autofunction:: scriptNotebook.optimizedDbscan

.. code:: ipython3

    def optimizedDbscan(x):
        """
        Apply DBSCAN algorithm to cluster data using the input value for epsilon (eps) and a minimum of 6 samples for each cluster.
    
        Args:
            x (float): The value of epsilon (eps) to use for DBSCAN clustering.
    
        Returns:
            optimized_dbscan (DBSCAN object): An instance of the DBSCAN algorithm with the specified parameters fit to the input data.
        """
        optimized_dbscan = DBSCAN(eps=x,min_samples=6)
        optimized_dbscan.fit(temp)
        print("Estimated number of clusters:",set(optimized_dbscan.labels_))
        print("Estimated number of noise points:",list(optimized_dbscan.labels_).count(-1))
        print("Silhouette Score:",str(np.round(silhouette_score(temp,optimized_dbscan.labels_)*100,2)) + '%')
    
        return optimized_dbscan
    
    optimized_dbscan = optimizedDbscan(x)


.. parsed-literal::

    Estimated number of clusters: {0, 1, 2, 3, 4, 5, 6, -1}
    Estimated number of noise points: 517
    Silhouette Score: 37.8%
    

.. code:: ipython3

    sns.scatterplot(data=temp, x='CustAccountBalance',y='TransactionAmount',hue=optimized_dbscan.labels_,s=300,alpha=0.6,palette='coolwarm')




.. parsed-literal::

    <Axes: xlabel='CustAccountBalance', ylabel='TransactionAmount'>




.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_166_1.png


Once we found the best number of samples, in this case 6, we proceeded
to run the model again and found that the Silhouette Score increased
from 25% to 37%, however we have not yet finished optimizing the model.

Next we are going to find the best value of epsilon

.. autofunction:: scriptNotebook.optDbscanClusteval

.. code:: ipython3

    from clusteval import clusteval
    temp= temp.sample(500)
    
    def optDbscanClusteval(temp):
        """
        Evaluates optimal number of clusters for DBSCAN clustering algorithm using clusteval library on input data.
        
        Args:
            temp : numpy array or pandas DataFrame
                A dataset of input features to be clustered using DBSCAN algorithm.
    
        Returns:
            ce : clusteval object
                An object of the clusteval library containing clustering evaluation results.
        """
        ce = clusteval(cluster='dbscan', verbose=3)
        results = ce.fit(np.reshape(np.ravel(temp),(-1,1)))
        cluster_labels = results['labx']
        print("Distinct Cluster Labels Detected:", np.unique(cluster_labels))
        return ce
    
    ce = optDbscanClusteval(temp)


.. parsed-literal::

    [clusteval] >INFO> Saving data in memory.
    [clusteval] >INFO> Fit with method=[dbscan], metric=[euclidean], linkage=[ward]
    [clusteval] >INFO> Gridsearch across Epsilon.
    [clusteval] >INFO> Evaluate using silhouette..
    [clusteval] >INFO: 100%|██████████| 245/245 [01:35<00:00,  2.57it/s]
    [clusteval] >INFO> Compute dendrogram threshold.
    [clusteval] >INFO> Optimal number clusters detected: [2].
    [clusteval] >INFO> Fin.
    

.. parsed-literal::

    Distinct Cluster Labels Detected: [-1  0]
    

.. code:: 1python3

    ce.plot()



.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_170_0.png




.. parsed-literal::

    (<Figure size 1500x800 with 2 Axes>,
     (<Axes: xlabel='Epsilon', ylabel='Score'>, <Axes: ylabel='Nr. Clusters'>))



We can now see that the best epsilon is equal to 3, we proceed to train
our new model with the optimized values

.. autofunction:: scriptNotebook.enhancedDbscan

.. code:: ipython3

    def enhancedDbscan(temp):
        """
        Perform density-based clustering using DBSCAN with an enhanced hyperparameter setting.
    
        Args:
            temp (pandas.DataFrame): A DataFrame containing the data to be clustered.
    
        Returns:
            enenhanced_dbscan (sklearn.cluster.DBSCAN): A DBSCAN object that has been fit to the data.
        """
        enhanced_dbscan = DBSCAN(eps=3,min_samples=2*temp.shape[1])
        enhanced_dbscan.fit(temp)
        print("Estimated number of clusters:",len(set(enhanced_dbscan.labels_)),set(enhanced_dbscan.labels_))
        return enhanced_dbscan
    
    enhanced_dbscan = enhancedDbscan(temp)


.. parsed-literal::

    Estimated number of clusters: 2 {0, -1}
    

.. code:: ipython3

    sns.scatterplot(data=temp,x='CustAccountBalance',y='TransactionAmount',hue=enhanced_dbscan.labels_,palette='prism',s=300,alpha=0.5)




.. parsed-literal::

    <Axes: xlabel='CustAccountBalance', ylabel='TransactionAmount'>




.. image:: Customer_Segmentation_files/CS_Bank_Transaction_v2_173_1.png


.. code:: ipython3

    print("Silhouette Score:",str(np.round(silhouette_score(temp,enhanced_dbscan.labels_)*100,2)) + '%')


.. parsed-literal::

    Silhouette Score: 56.73%
    

The optimized DBSCAN model has classified the entire population of
customers primarily into two major groups, one of them consists of all
those customers who have modest account balance and make low-value
transactions whereas the miscellaneous group includes either the
customers who have high account balances and spend very less money
through transactions or those who have minimal account balance but
expend a large amount of cash through high-value transactions.