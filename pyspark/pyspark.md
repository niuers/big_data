## Construct a DataFrame
* Use a list `Row`s

```
spark = SparkSession.builder.appName('test').getOrCreate()
df = spark.createDataFrame([Row(ID=10001, day='2019-05-01', value=70),
                            Row(ID=10002, day='2019-06-01', value=20),
                            Row(ID=10003, day='2019-07-01', value=35)
                           ]).show()

```

* Use a list of tuples and a list of column names

```
values = [
('05/25/2018', 'a', 37,  12),
('04/15/2018', 'a', 29,  13),
('12/18/2017', 'b', 17,  14),
('07/28/2017', 'a', 10,  23),
('01/01/2018', 'b', 12,  33)
]
columns = ['Date', 'Name','Count', 'ID']
df=spark.createDataFrame(values, columns).show()
```

* use a list of tuples without column names
```
df = spark.createDataFrame([(1, 5, 3), (1,2,6),(2, 3,1), (2,1,-1)] )
```


## Groupby and Apply on DataFrame

```
from pyspark.sql import functions as F
df.groupBy(["ID", "Amount"]).agg(F.countDistinct(df['d']).alias('count_d')).show()
df.groupBy(["ID", "Amount"]).agg({"d": "count", "Amount":"max"}).show()

```

## [Apply a user defined function (udf) to create a new column](https://stackoverflow.com/questions/57095416/what-is-wrong-with-this-function-on-pyspark/57097490#57097490)
### General UDF
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

### UDF on a column of pyspark.ml.linalg.DenseVector

* As of Spark 2.4.0, in the udf for DenseVector etc., the return values have to be python type, not numpy type. 
For example, in the code below, we have to use `float` to convert the result of `v.dot(vm)` to python type `float`, whereas the `DenseVector.dot` method's default return type is `numpy.float64`. This return type can't be converted by `udf`, and will [throw error](https://stackoverflow.com/questions/44150375/converting-row-into-list-rdd-in-pyspark) due to [spark bug](https://issues.apache.org/jira/browse/SPARK-12157).
> net.razorvine.pickle.PickleException: expected zero arguments for construction of ClassDict (for numpy.dtype)

```
v = [('a', DenseVector([1,2,3])),
    ('b', DenseVector([4,5,6])),
    ('c', DenseVector([9,8,7]))]
dfv1 = spark.createDataFrame(v, ['id', 'values'])
dfv1.show()

vm = Vectors.dense([1,2,3])
dot_prod_udf = F.udf(lambda v: float(v.dot(vm)), FloatType())
dfv1 = dfv1.withColumn('dot_prod', dot_prod_udf('values'))
dfv1.show()
```


## [Don't put two spark DataFrames in one line!](https://stackoverflow.com/questions/57093177/pyspark-isin-with-column-in-argument-doesnt-exclude-rows)

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

## [Calling `pandas shift` within `pandas_udf` causes error if the target column is initialized with F.lit()](https://stackoverflow.com/questions/57152199/value-at-index-is-null-error-when-using-pd-shift-inside-pandas-udf)

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

## [Don't do filter using aggregrate column](https://stackoverflow.com/questions/57144409/filtering-a-dataframe-after-groupby-and-user-define-aggregate-function-in-pyspar)

The current spark optimizer tries to optimize the execution plan, and place the filter right after the DataFrame is created. But the code there seems do not check for evaluation error, and will fail the whole program. To avoid such optimization, call `cache()` after you have generated the aggregrate column.

This is the code that generates error
```
df = spark.createDataFrame([(1, 1.0), (1, 2.0), (2, 3.0), (2, 5.0), (2, 10.0)],("id", "v"))
df.show()
@pandas_udf("double", PandasUDFType.GROUPED_AGG)
def mean_udf(v):
    return v.mean()
df = df.groupby("id").agg(mean_udf(df['v']).alias("mean"))
# df.cache()  #without this line, there'll be errors
df.filter(F.col("mean") > 5).show()
```

Errors caused by the `filter` operation
```
java.lang.UnsupportedOperationException: Cannot evaluate expression: mean_udf(input[1, double, true])
```
[A related issue with PySpark.](https://issues.apache.org/jira/browse/SPARK-17100)

## [Don't use for-loop](https://stackoverflow.com/questions/57154430/how-to-apply-multiple-filters-in-a-for-loop-for-pyspark)

Use for-loop to iterate transformations on a DataFrame or RDD is a bad idea. `Code1` and `Code2` below give two different results.
This might be related to [closures](https://spark.apache.org/docs/2.2.1/rdd-programming-guide.html#understanding-closures-).
```
# Code 1
test_input = [('0', '00'), ('1', '1'), ('', '22'), ('', '3')]
rdd = sc.parallelize(test_input, 1)

# Index 0 needs to be longer than length 0
# Index 1 needs to be longer than length 1
for i in [0,1]:
    rdd = rdd.filter(lambda arr: len(arr[i]) > i)   

print('final: ', rdd.collect())
```
This outputs
```
final:  [('0', '00'), ('', '22')]
```

```
# Code2
test_input = [('0', '00'), ('1', '1'), ('', '22'), ('', '3')]
rdd = sc.parallelize(test_input, 1)
rdd = rdd.filter(lambda arr: len(arr[0]) > 0)
rdd = rdd.filter(lambda arr: len(arr[1]) > 1)
print('final: ', rdd.collect())
```
This outputs:
```
final: [('0', '00')]
```

## [Use Select to Re-Order DataFrame Columns](https://stackoverflow.com/questions/42912156/python-pyspark-data-frame-rearrange-columns)

```
df.select([col1, col2, col3]).show()
```
