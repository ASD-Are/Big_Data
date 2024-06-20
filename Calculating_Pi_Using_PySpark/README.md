# Pi Calculation using PySpark

## Description
This project demonstrates how to calculate the value of Pi using PySpark. The process involves generating random (x, y) coordinates and determining the number of points that fall inside a unit circle. The ratio of points inside the circle to the total number of points is used to estimate the value of Pi.

## Steps

### Step 1: Create the PySpark Script
1. Create and edit the Python script `calculate_pi.py`:
    ```bash
    vi calculate_pi.py
    ```
2. Add the following code to `calculate_pi.py`:
    ```python
    import argparse
    import logging
    from operator import add
    from random import random

    from pyspark.sql import SparkSession

    logger = logging.getLogger(__name__)
    logging.basicConfig(level=logging.INFO, format='%(levelname)s: %(message)s')

    def calculate_pi(partitions, output_uri):
        def calculate_hit(_):
            x = random() * 2 - 1
            y = random() * 2 - 1
            return 1 if x ** 2 + y ** 2 < 1 else 0

        tries = 100000 * partitions

        logger.info("Calculating pi with a total of %s tries in %s partitions.", tries, partitions)

        with SparkSession.builder.appName("My PyPi").getOrCreate() as spark:
            hits = spark.sparkContext.parallelize(range(tries), partitions).map(calculate_hit).reduce(add)
            pi = 4.0 * hits / tries

            logger.info("%s tries and %s hits gives pi estimate of %s.", tries, hits, pi)

            if output_uri is not None:
                df = spark.createDataFrame([(tries, hits, pi)], ['tries', 'hits', 'pi'])
                df.write.mode('overwrite').json(output_uri)

    if __name__ == "__main__":
        parser = argparse.ArgumentParser()
        parser.add_argument('--partitions', default=2, type=int, help="The number of parallel partitions to use when calculating pi.")
        parser.add_argument('--output_uri', help="The URI where output is saved, typically an S3 bucket.")
        args = parser.parse_args()

        calculate_pi(args.partitions, args.output_uri)
    ```

3. Upload the script to a Google Cloud Storage bucket:
    ```bash
    gsutil cp calculate_pi.py gs://big_data_calculating_pi/
    ```

### Step 2: Authenticate with Google Cloud
1. Authenticate your Google Cloud account to access necessary resources:
    ```bash
    gcloud auth login
    ```

### Step 3: Submit the PySpark Job
1. Submit the PySpark job to a Dataproc cluster:
    ```bash
    gcloud dataproc jobs submit pyspark gs://big_data_calculating_pi/calculate_pi.py \
        --cluster=pyspark \
        --region=us-central1 \
        -- \
        --partitions=4 \
        --output_uri=gs://big_data_calculating_pi/pi-output
    ```

### Step 4: Verify the Output
1. List the output files in the specified bucket:
    ```bash
    gsutil ls gs://big_data_calculating_pi/pi-output/
    ```

2. Display the contents of the output files to verify the calculated value of Pi:
    ```bash
    gsutil cat gs://big_data_calculating_pi/pi-output/*.json
    ```

    **Expected Output**:
    ```json
    {"tries":400000,"hits":314484,"pi":3.14484}
    ```
## Testing
![image (17)](https://github.com/ASD-Are/Big_Data/assets/93379106/992d2ddb-2b29-44f8-b601-e4ede693b065)
![image (18)](https://github.com/ASD-Are/Big_Data/assets/93379106/5199f4a1-9b91-4426-9dd3-bdd65e27ee3d)
![image (8)](https://github.com/ASD-Are/Big_Data/assets/93379106/09709fb5-33ce-4f92-b43c-579a1c082776)
![image (9)](https://github.com/ASD-Are/Big_Data/assets/93379106/e85ca51d-ca73-466d-87ae-9619019bfe69)
![image (3)](https://github.com/ASD-Are/Big_Data/assets/93379106/367cc8a0-3719-41d9-ae27-9cb4154763d7)
![image (4)](https://github.com/ASD-Are/Big_Data/assets/93379106/182e6204-6ed1-43dd-bcf6-790cd8103989)
![image (5)](https://github.com/ASD-Are/Big_Data/assets/93379106/650dea7f-53e0-42fc-91d0-5cec24234cd0)
![image (16)](https://github.com/ASD-Are/Big_Data/assets/93379106/0f8aec39-62a4-4d4e-bfa3-10f5b98de30a)
![image (7)](https://github.com/ASD-Are/Big_Data/assets/93379106/ae478b08-a55d-4bc3-8ff8-592f3e700185)
![image (10)](https://github.com/ASD-Are/Big_Data/assets/93379106/f2dc9a8f-59e0-4307-8667-57cecc1830a4)
![image (12)](https://github.com/ASD-Are/Big_Data/assets/93379106/8f947658-03f7-4e5e-b56b-3ca7ea4acc1f)
![image (11)](https://github.com/ASD-Are/Big_Data/assets/93379106/d1ea0e30-57bc-49d4-86aa-73ba83abe527)
![image (13)](https://github.com/ASD-Are/Big_Data/assets/93379106/12a13647-1f6f-4750-a59f-fff734807660)
![image (14)](https://github.com/ASD-Are/Big_Data/assets/93379106/b180b77e-b600-4fa4-a9cf-c50e1c7e33c0)
**Results**
![image (15)](https://github.com/ASD-Are/Big_Data/assets/93379106/01b71afb-601c-480a-8233-d33b7dba6a2c)


## Conclusion
- Successfully implemented Pi calculation using PySpark.
- Demonstrated the advantages of using PySpark for big data processing.
- Estimated the value of Pi through parallel processing and distributed computing.

## References
1. [Pi Calculation using PySpark](https://hc.labnet.sfbu.edu/~henry/npu/classes/learning_spark/key_value_pair/slide/pi.html)

## Additional Resources
- GitHub: [Pi Calculation using PySpark](https://github.com/ASD-Are/Big_Data/tree/main/Calculating_Pi_Using_PySpark)
- Google Slide: [Pi Calculation using PySpark](https://docs.google.com/presentation/d/1z9DnACVbe5ixpwXpqXMmEAtkZQBTf5dgHIsWHiXh_RE/edit?usp=sharing)

Thank you!
