# Deep Learning Pipelines for Apache Spark

## Overview
This project involves developing a robust pipeline for large-scale image datasets, implementing transfer learning, and evaluating performance using Apache Spark. It demonstrates an efficient workflow for deep learning in image processing.

## Table of Contents
1. Set up the Environment
2. Download Programs and Related Documentation
3. Process of Program Execution
4. Screenshot of Execution Results

## 1. Set up the Environment
To set up the environment for this project, follow these steps:

1. **Create an Account in Databricks**
   - Go to the Databricks website and create an account.
   - Create a workspace and a new notebook within Databricks.

2. **Configuring the Cluster Environment**
   - **Install Libraries**: Utilize the Deep Learning Pipelines available as a Spark Package. Create a new library in your cluster with the Source option set to "Maven Coordinate". Search for "spark-deep-learning" in "Search Spark Packages and Maven Central" and attach the library to the cluster.
   - **Install Dependencies**: Add the Maven coordinate for spark-deep-learning to your Spark configuration. Install the necessary Python libraries: TensorFlow, Keras, h5py, and spark-deep-learning.

```python
%sh 
curl -O http://download.tensorflow.org/example_images/flower_photos.tgz
tar xzf flower_photos.tgz

display(dbutils.fs.ls("file:/Workspace/Users/your-email@domain.com/flower_photos"))
```

## 2. Download Programs and Related Documentation

1. **Obtain Image Dataset**
   - Download and extract the flowers dataset from the TensorFlow retraining tutorial.

2. **Create a Sample Set of Images**
   - Generate a reduced set of images for quick demonstrations.

```python
# Define the base directory for images
img_dir = "file:/Workspace/Users/your-email@domain.com/flower_photos"

# Create a sample set
sample_img_dir = img_dir + "/sample"
dbutils.fs.mkdirs(sample_img_dir)
files = dbutils.fs.ls(img_dir + "/tulips")[0:1] + dbutils.fs.ls(img_dir + "/daisy")[0:2]
for f in files:
    dbutils.fs.cp(f.path, sample_img_dir)
display(dbutils.fs.ls(sample_img_dir))
```

## 3. Process of Program Execution

### Image Data Handling
1. **Load Images into Spark DataFrame**
   - Load images into a Spark DataFrame using utility functions that decode them automatically.

```python
from pyspark.sql.functions import lit

# Load and label images
tulips_df = spark.read.format("image").load(img_dir + "/tulips").withColumn("label", lit(1))
daisy_df = spark.read.format("image").load(img_dir + "/daisy").withColumn("label", lit(0))

# Split data into training and test sets
tulips_train, tulips_test = tulips_df.randomSplit([0.1, 0.9])
daisy_train, daisy_test = daisy_df.randomSplit([0.1, 0.9])

# Combine training and test sets
train_df = tulips_train.unionByName(daisy_train)
test_df = tulips_test.unionByName(daisy_test)

train_df = train_df.repartition(100)
test_df = test_df.repartition(100)
```

### Transfer Learning
1. **Feature Extraction and Model Training**

```python
from pyspark.ml.classification import LogisticRegression
from pyspark.ml import Pipeline
from sparkdl import DeepImageFeaturizer

featurizer = DeepImageFeaturizer(inputCol="image", outputCol="features", modelName="InceptionV3")
lr = LogisticRegression(maxIter=20, regParam=0.05, elasticNetParam=0.3, labelCol="label")
p = Pipeline(stages=[featurizer, lr])

p_model = p.fit(train_df)
```

2. **Making Predictions on New Image Data**

```python
from sparkdl import readImages, DeepImagePredictor

image_df = readImages(sample_img_dir)

predictor = DeepImagePredictor(inputCol="image", outputCol="predicted_labels",
                               modelName="InceptionV3", decodePredictions=True, topK=10)
predictions_df = predictor.transform(image_df)

display(predictions_df.select("filePath", "predicted_labels"))
```

## 4. Screenshot of Execution Results

1. **Displaying Predictions**

```python
df = p_model.transform(image_df)
display(df.select("filePath", (1 - p1(df.probability)).alias("p_daisy")))
```

2. **Example Screenshot**

[Execution Results]([path_to_screenshot_image](https://docs.google.com/presentation/d/1kcrP6mDqw9mm3CHm8HeGok1Ogi1K3_oR08F3KETmRKo/edit?usp=sharing))

## References
- Databricks Documentation: [Deep Learning Pipelines](https://docs.databricks.com/applications/deep-learning/)
- TensorFlow: [Flowers Dataset](http://download.tensorflow.org/example_images/flower_photos.tgz)

## Appendix
- Google Slide Presentation: [Project Presentation](https://docs.google.com/presentation/d/1kcrP6mDqw9mm3CHm8HeGok1Ogi1K3_oR08F3KETmRKo/edit?usp=sharing)
](https://docs.google.com/presentation/d/1kcrP6mDqw9mm3CHm8HeGok1Ogi1K3_oR08F3KETmRKo/edit?usp=sharing)
