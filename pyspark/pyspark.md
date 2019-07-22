* [Apply a user defined function to create a new column](https://stackoverflow.com/questions/57095416/what-is-wrong-with-this-function-on-pyspark/57097490#57097490)
```
from pyspark.sql import functions as F
from pyspark.sql.functions import udf

my_udf = udf(lambda my_texts: "text_passed" if my_texts.startswith('_text1') == True else my_texts, StringType())
# The column 'my_texts' can be an existing column name
df = spark_df.withColumn('my_texts', my_udf(spark_df['my_texts']))

```

This specific problem can also be done without using UDF
```
df = spark_df.withColumn("my_texts", F.when(F.instr(spark_df["my_texts"], '_text1')>0, 'text_passed').otherwise("my_texts"))
```

- [Don't put two spark DataFrames in one line!](https://stackoverflow.com/questions/57093177/pyspark-isin-with-column-in-argument-doesnt-exclude-rows)

It seems that when you mix the columns from two spark DataFrames in one line, the code won't work as expected. 

(1) If both `df` and `df_t` have a column `name`, following code won't produce expected results. It will always return empty. 
Since when the spark creates execution plan for this statement, it seems to remove the DataFrame reference, so in both places the `name` belongs to `df`.

```
df_f = df.filter(df.name.isin(df_t.name)== False)
df_f.explain(True)
> Filter (name#1836 IN (name#1836) = false)
```

(2) If `df` has a column `name`, but `df_t` has a column `name2`, the following code will error out. The execution plan failed to find `name2` in `df` DataFrame.
```
df_f = df.filter(df.name.isin(df_t.name2)== False)
```

- [Calling `pandas shift` within `pandas_udf` causes error if the target column is initialized with F.lit()](https://stackoverflow.com/questions/57152199/value-at-index-is-null-error-when-using-pd-shift-inside-pandas-udf)
Following code will cause errors:
```
spark = SparkSession.builder.appName('test').getOrCreate()
df = spark.createDataFrame([Row(id=1, name='a', c='3'),
Row(id=2, name='b', c='6'),
Row(id=3, name='a', c='2'),
Row(id=4, name='b', c='9'),
Row(id=5, name='c', c='7')])

# This will cause the error
df = df.withColumn('f', F.lit(1))
## This is ok.
## df = df.withColumn('f', df['c'])
df.show()

@pandas_udf(df.schema, PandasUDFType.GROUPED_MAP)
def shift_test(pdf):
    pdf['f'] = pdf['c'].shift(1)
return pdf

df = df.groupby(['name']).apply(shift_test)
df.show()

```
The error:
```
Caused by: java.lang.IllegalStateException: Value at index is null
```

  
