1. Create a Cloud SQL instance
```
gcloud sql instances create rentals --root-password=password --database-version=MYSQL_5_7 --zone=europe-west2-a
```

2. In Cloud SQL, click rentals to view instance information.
 ```
 gcloud sql instances describe rentals
 ```

3. Find the Connect to this instance box on the page and click on connect using Cloud Shell
```
gcloud sql connect rentals --user=root
```
4. Run the below command
```
SHOW DATABASES;
```

5. Copy and paste the below SQL statement you analyzed earlier paste it into the command line
```
CREATE DATABASE IF NOT EXISTS recommendation_spark;

USE recommendation_spark;

DROP TABLE IF EXISTS Recommendation;
DROP TABLE IF EXISTS Rating;
DROP TABLE IF EXISTS Accommodation;

CREATE TABLE IF NOT EXISTS Accommodation
(
  id varchar(255),
  title varchar(255),
  location varchar(255),
  price int,
  rooms int,
  rating float,
  type varchar(255),
  PRIMARY KEY (ID)
);

CREATE TABLE  IF NOT EXISTS Rating
(
  userId varchar(255),
  accoId varchar(255),
  rating int,
  PRIMARY KEY(accoId, userId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);

CREATE TABLE  IF NOT EXISTS Recommendation
(
  userId varchar(255),
  accoId varchar(255),
  prediction float,
  PRIMARY KEY(userId, accoId),
  FOREIGN KEY (accoId)
    REFERENCES Accommodation(id)
);

SHOW DATABASES;
```

6. Run the following command to show our tables
```
USE recommendation_spark;
SHOW TABLES;
```

7. Run the following query
```
SELECT * FROM Accommodation;
```

8. Stage Data in Google Cloud Storage
```
echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID"
gsutil mb gs://$DEVSHELL_PROJECT_ID

echo "Copying data to our storage from public dataset"
gsutil cp gs://cloud-training/bdml/v2.0/data/accommodation.csv gs://$DEVSHELL_PROJECT_ID
gsutil cp gs://cloud-training/bdml/v2.0/data/rating.csv gs://$DEVSHELL_PROJECT_ID

echo "Show the files in our bucket"
gsutil ls gs://$DEVSHELL_PROJECT_ID

echo "View some sample data"
gsutil cat gs://$DEVSHELL_PROJECT_ID/accommodation.csv
```
9. Import accommodation data
```
gcloud sql import csv rentals gs://$DEVSHELL_PROJECT_ID/accommodation.csv --database=recommendation_spark --table=Accommodation --user=root
```

10. Import user rating data
```
gcloud sql import csv renta
ls gs://$DEVSHELL_PROJECT_ID/rating.csv --database=recommendation_spark --table=Rating
```
11. Explore Cloud SQL data

```
gcloud sql connect rentals --user=root
```

12. Query the ratings data:
```
USE recommendation_spark;
SELECT * FROM Rating
LIMIT 15;
```

13. Use a SQL aggregation function to count the number of rows in the table
```
SELECT COUNT(*) AS num_ratings
FROM Rating;
```

14. What's the average review of our accommodations?
```
SELECT
    COUNT(userId) AS num_ratings,
    COUNT(DISTINCT userId) AS distinct_user_ratings,
    MIN(rating) AS worst_rating,
    MAX(rating) AS best_rating,
    AVG(rating) AS avg_rating
FROM Rating;
```

15. Run the below query to see which users have provided the most ratings
```
SELECT
    userId,
    COUNT(rating) AS num_ratings
FROM Rating
GROUP BY userId
ORDER BY num_ratings DESC;
```

16. Create Dataproc Cluster
```
gcloud dataproc clusters create rentals --region=global --zone=europe-west2-a --master-machine-type=n1-standard-2 --worker-machine-type=n1-standard-2
```

17. Copy and paste the below bash script into your Cloud Shell
```
echo "Authorizing Cloud Dataproc to connect with Cloud SQL"
CLUSTER=rentals
CLOUDSQL=rentals
ZONE=us-central1-a
NWORKERS=2

machines="$CLUSTER-m"
for w in `seq 0 $(($NWORKERS - 1))`; do
   machines="$machines $CLUSTER-w-$w"
done

echo "Machines to authorize: $machines in $ZONE ... finding their IP addresses"
ips=""
for machine in $machines; do
    IP_ADDRESS=$(gcloud compute instances describe $machine --zone=$ZONE --format='value(networkInterfaces.accessConfigs[].natIP)' | sed "s/\['//g" | sed "s/'\]//g" )/32
    echo "IP address of $machine is $IP_ADDRESS"
    if [ -z  $ips ]; then
       ips=$IP_ADDRESS
    else
       ips="$ips,$IP_ADDRESS"
    fi
done

echo "Authorizing [$ips] to access cloudsql=$CLOUDSQL"
gcloud sql instances patch $CLOUDSQL --authorized-networks $ips
```

18. Copy over the model code by executing the below in Cloud Shell
```
gsutil cp gs://cloud-training/bdml/v2.0/model/train_and_apply.py train_and_apply.py
cloudshell edit train_and_apply.py
```

19. In train_and_apply.py, find line 30: CLOUDSQL_INSTANCE_IP and paste your Cloud SQL IP address you copied earlier
```
sed -i -e 's/<paste-your-cloud-sql-ip-here>/34.89.85.64/g' train_and_apply.py
```

20. Find line 33: CLOUDSQL_PWD and type in your Cloud SQL password
```
sed -i -e 's/<type-your-cloud-sql-password-here>/password/g' train_and_apply.py
```

21. Copy this file to your Cloud Storage bucket using this Cloud Shell command:
```
gsutil cp train_and_apply.py gs://$DEVSHELL_PROJECT_ID
```

22. Run your ML job on Dataproc
```
gcloud dataproc jobs submit pyspark gs://$DEVSHELL_PROJECT_ID/train_and_apply.py --cluster=rentals
```

23. Explore inserted rows with SQL
```
gcloud sql connect rentals --user=root

USE recommendation_spark;

SELECT COUNT(*) AS count FROM Recommendation;
```

24. Find the recommendations for a user:
```
SELECT
    r.userid,
    r.accoid,
    r.prediction,
    a.title,
    a.location,
    a.price,
    a.rooms,
    a.rating,
    a.type
FROM Recommendation as r
JOIN Accommodation as a
ON r.accoid = a.id
WHERE r.userid = 10;
```
