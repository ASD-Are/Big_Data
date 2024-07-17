This readme document outlines the steps to set up and run a movie recommendation system using PySpark's Collaborative Filtering with the Alternating Least Squares (ALS) algorithm on Google Cloud Platform (GCP). The project leverages scalable infrastructure to handle large datasets efficiently.

### Set Up the Environment

1. **Google Cloud Platform (GCP) Setup**:
   - Create a Google Cloud account.
   - Set up a Google Cloud Storage (GCS) bucket, for instance, `big_data_ml_recommendation_sys`.
   - Create a Google Cloud Dataproc cluster named `spark` in the `us-central1` region and `us-central1-a` zone.

2. **Google Colab Setup**:
   - Access Google Colab and set up a notebook for initial experimentation.

### Download Programs and Related Documentation

1. **Datasets**:
   - `movies.csv`: Contains `movieId`, `title`, and `genres`.
   - `ratings.csv`: Contains `userId`, `movieId`, `rating`, and `timestamp`.
   - `tags.csv`: Contains `userId`, `movieId`, `tag`, and `timestamp`.

2. **Code**:
   - Download the PySpark Collaborative Filtering with ALS code from GitHub:
     ```
     https://github.com/snehalnair/als-recommender-pyspark/blob/master/Recommendation_Engine_MovieLens.ipynb
     ```

3. **Modification and Testing**:
   - Modify the notebook as needed and save it as a `.py` file for GCP execution.
   - Example modification link: [Colab Notebook](https://colab.research.google.com/drive/1MB2pqvSFtKhll9W8vFQb-UF2D63e-g_P?usp=sharing)

### Process of Program Execution

1. **Google Colab**:
   - Upload the datasets (`movies.csv`, `ratings.csv`, `tags.csv`) to your Google Drive.
   - Upload the modified `.ipynb` file to Google Colab.
   - Experiment and test the PySpark code in Colab.

2. **Google Cloud Storage (GCS)**:
   - Upload the input files to your GCS bucket:
     ```shell
     gsutil cp movies.csv gs://big_data_ml_recommendation_sys/
     gsutil cp ratings.csv gs://big_data_ml_recommendation_sys/
     gsutil cp Recommendation_Engine_MovieLens.py gs://big_data_ml_recommendation_sys/
     ```

3. **Google Cloud Dataproc**:
   - Modify the PySpark script to use GCS paths.
   - Submit the PySpark job to the Dataproc cluster:
     ```shell
     gcloud dataproc jobs submit pyspark gs://big_data_ml_recommendation_sys/Recommendation_Engine_MovieLens.py \
     --cluster=spark \
     --region=us-central1 \
     -- \
     --input_path_movies=gs://big_data_ml_recommendation_sys/movies.csv \
     --input_path_ratings=gs://big_data_ml_recommendation_sys/ratings.csv
     ```

### Screenshot of Execution Results

- Screenshots should be taken at various stages, including:
  - Uploading datasets to GCS.
  - Modifying and running the PySpark script in Google Colab.
  - Submitting and monitoring the PySpark job in Google Cloud Dataproc.
  - Viewing the recommendation results.

### Appendix
[Google Slide](https://docs.google.com/presentation/d/1MslhW6L9oKag5c4O-vUINS5lUSUxacKLWuiYr20ygyA/edit?usp=sharing)
[Screenshot and clear steps in pdf](https://github.com/ASD-Are/Big_Data/blob/main/Collaborative%20Filtering%20-%20Implementation%203/CS570_week8_h2_q4_20073_Aron_Dagniew.pdf)
