1. Create VM instance
```
gcloud beta compute --project=qwiklabs-gcp-03-9c07e1fcda1a instances create bloghost --zone=us-central1-a --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --metadata=startup-script=apt-get\ update$'\n'apt-get\ install\ apache2\ php\ php-mysql\ -y$'\n'service\ apache2\ restart --maintenance-policy=MIGRATE --service-account=680444148736-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=bloghost --reservation-affinity=any

gcloud compute --project=qwiklabs-gcp-03-9c07e1fcda1a firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```

3. Create a Cloud Storage bucket
```
export LOCATION=US

gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID
```

4. Retrieve a banner image from a publicly accessible Cloud Storage location:
```
gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png
```

5. Copy the banner image to your newly created Cloud Storage bucket:
```
gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```

6. Modify the Access Control List of the object you just created so that it is readable by everyone:
```
gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```

7. Create the Cloud SQL instance

```
gcloud sql instances create blog-db --root-password=password --database-version=MYSQL_5_7 --zone=us-central1-a
```

8. Create a User
```
gcloud sql users create blogdbuser --instance=blog-db --host=% --password=password
```

9. Configure application to use Cloud SQL
```
cd /var/www/html

echo '<html>
<head><title>Welcome to my excellent blog</title></head>
<body>
<h1>Welcome to my excellent blog</h1>
<?php
 $dbserver = "CLOUDSQLIP";
$dbuser = "blogdbuser";
$dbpassword = "DBPASSWORD";
// In a production blog, we would not store the MySQL
// password in the document root. Instead, we would store it in a
// configuration file elsewhere on the web server VM instance.

$conn = new mysqli($dbserver, $dbuser, $dbpassword);

if (mysqli_connect_error()) {
        echo ("Database connection failed: " . mysqli_connect_error());
} else {
        echo ("Database connection succeeded.");
}
?>
</body></html>' > index.php
```

10. Start the web server
```
sudo service apache2 restart
```

11. Request the index page
```
curl 34.72.124.42/index.php
```

12. Replace placeholders with connection IP and password
```
sed -i -e 's/CLOUDSQLIP/34.121.22.79/g' index.php
sed -i -e 's/DBPASSWORD/password/g' index.php
```

13. Restart web server
```
sudo service apache2 restart
```

14. Configure an Application to use Cloud Storage
```
cd /var/www/html

echo '<html>
<head><title>Welcome to my excellent blog</title></head>
<body>
<h1>Welcome to my excellent blog</h1>
<img src='https://storage.googleapis.com/qwiklabs-gcp-03-9c07e1fcda1a/my-excellent-blog.png'>
<?php
 $dbserver = "CLOUDSQLIP";
$dbuser = "blogdbuser";
$dbpassword = "DBPASSWORD";
// In a production blog, we would not store the MySQL
// password in the document root. Instead, we would store it in a
// configuration file elsewhere on the web server VM instance.

$conn = new mysqli($dbserver, $dbuser, $dbpassword);

if (mysqli_connect_error()) {
        echo ("Database connection failed: " . mysqli_connect_error());
} else {
        echo ("Database connection succeeded.");
}
?>
</body></html>' > index.php
```

15. Restart the server
```
sudo service apache2 restart
```

