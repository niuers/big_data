* [Apply a user defined function to create a new column](https://stackoverflow.com/questions/57095416/what-is-wrong-with-this-function-on-pyspark/57097490#57097490)
```
from pyspark.sql import SparkSession, SQLContext, Column
from pyspark.sql import functions as F
from pyspark.sql.types import StringType
from pyspark.sql.functions import udf
import pandas as pd

spark = SparkSession.builder.appName('test').getOrCreate()
df2 = pd.DataFrame([['_text1']],columns = ['my_texts'])
spark_df = spark.createDataFrame(df2)
spark_df.show()

text_func = udf(lambda my_texts: "text_passed" if my_texts.startswith('_text1') == True else my_texts, StringType())
df = spark_df.withColumn('my_texts', text_func(spark_df['my_texts']))
df.show()

```

This specific problem can also be done without using UDF
```
df = spark_df.withColumn("my_texts", F.when(F.instr(spark_df["my_texts"], '_text1')>0, 'text_passed').otherwise("my_texts"))
```
