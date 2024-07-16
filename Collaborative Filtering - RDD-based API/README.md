# Collaborative Filtering Recommendation System

## Overview
This project focuses on building a collaborative filtering recommendation system using PySpark and Google Cloud Dataproc. Collaborative filtering predicts the interests of a user by collecting preferences from many users.

## Problem Statement
The goal is to provide accurate movie recommendations by predicting user ratings for movies they haven't watched yet, based on historical user-movie interaction data.

## Scope
The project involves:
- Data preparation
- Model training using Alternating Least Squares (ALS) in PySpark
- Evaluation of the model's performance
- Deployment on Google Cloud Dataproc

## Table of Contents
1. [Setup the Environment](https://github.com/ASD-Are/Big_Data/blob/main/Collaborative%20Filtering%20-%20RDD-based%20API/CS570_week8_q2_20073_Aron_Dagniew.pdf)
2. [Download Programs and Related Documentation](https://github.com/ASD-Are/Big_Data/blob/main/Collaborative%20Filtering%20-%20RDD-based%20API/CS570_week8_q2_20073_Aron_Dagniew.pdf)
3. [Process of Program Execution](https://github.com/ASD-Are/Big_Data/blob/main/Collaborative%20Filtering%20-%20RDD-based%20API/CS570_week8_q2_20073_Aron_Dagniew.pdf)
4. [Screenshot of Execution Results](https://github.com/ASD-Are/Big_Data/blob/main/Collaborative%20Filtering%20-%20RDD-based%20API/CS570_week8_q2_20073_Aron_Dagniew.pdf)

## Setup the Environment
1. Open Google Cloud Console.
2. Activate Cloud Shell.
3. Authenticate with Google Cloud Platform (GCP).
4. Create a Google Cloud Storage bucket to store data.

## Download Programs and Related Documentation
- Clone the repository: `git clone https://github.com/ASD-Are/Big_Data`
- Download the MovieLens dataset: [MovieLens 100k Dataset](https://files.grouplens.org/datasets/movielens/ml-100k/u.data)

## Process of Program Execution

### Data Preparation
1. **Create the `u.data` File**: 
   ```bash
   vim u.data
   ```
   Populate `u.data` with your data in the format (UserID, MovieID, rating, Timestamp).

2. **Transform the Raw Data**:
   ```bash
   cat u.data | tr -s ' ' | cut -d' ' -f1-3 | tr ' ' ',' > u_data_transformed.csv
   ```
   This command trims extra spaces, extracts the first three fields (UserID, MovieID, rating), and replaces spaces with commas. The transformed data is saved in `u_data_transformed.csv`.

3. **Upload the Transformed Data**:
   ```bash
   gsutil cp u_data_transformed.csv gs://YOUR_BUCKET_NAME/
   ```

### Create PySpark Script
- Create a PySpark script `recommendation_example.py` with the following content:
  ```python
  from pyspark import SparkContext
  from pyspark.mllib.recommendation import ALS, Rating

  if __name__ == "__main__":
      sc = SparkContext(appName="PythonCollaborativeFilteringExample")
      data = sc.textFile("gs://YOUR_BUCKET_NAME/u_data_transformed.csv")
      ratings = data.map(lambda l: l.split(','))\
                    .map(lambda l: Rating(int(l[0]), int(l[1]), float(l[2])))
      
      rank = 10
      numIterations = 10
      model = ALS.train(ratings, rank, numIterations)
      
      testdata = ratings.map(lambda p: (p[0], p[1]))
      predictions = model.predictAll(testdata).map(lambda r: ((r[0], r[1]), r[2]))
      
      ratesAndPreds = ratings.map(lambda r: ((r[0], r[1]), r[2])).join(predictions)
      MSE = ratesAndPreds.map(lambda r: (r[1][0] - r[1][1])**2).mean()
      print("Mean Squared Error = " + str(MSE))
  ```

### Create Dataproc Cluster
1. Create a Dataproc cluster:
   ```bash
   gcloud dataproc clusters create spark-cluster --region us-west1 --zone us-west1-a --single-node
   ```

### Submit PySpark Job
1. Submit the PySpark job to the Dataproc cluster:
   ```bash
   gcloud dataproc jobs submit pyspark gs://YOUR_BUCKET_NAME/recommendation_example.py --cluster=spark-cluster --region=us-west1
   ```

## Screenshot of Execution Results
- After running the job, the output will display the Mean Squared Error (MSE) of the model.
  ```
  Mean Squared Error = 0.48419423210378404
  ```

## Conclusion
- Successfully built a collaborative filtering recommendation system using PySpark and Google Cloud Dataproc.
- Data preparation involved transforming and uploading the MovieLens dataset to Google Cloud Storage.
- Model training utilized the ALS algorithm on user-movie interaction data.
- The model's performance was evaluated using Mean Squared Error (MSE), achieving a value of 0.48419423210378404, indicating reasonable prediction accuracy.


## Appendix
- [Screenshot and environment set up references](https://github.com/ASD-Are/Big_Data/blob/main/Collaborative%20Filtering%20-%20RDD-based%20API/CS570_week8_q2_20073_Aron_Dagniew.pdf)
- [Google Slide Presentation]()
## References
- [MovieLens Dataset](https://files.grouplens.org/datasets/movielens/ml-100k/u.data)
- [Collaborative Filtering - RDD-based API](https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html)
