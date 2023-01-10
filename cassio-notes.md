# 2_docker_sql
## steps
* go to folder 2_docker_sql
* build the processing image `docker build -t taxi_ingest:v001 .`
* run docker compose `docker compose up -d`
* run the processing image
```
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

docker run -it \
    --network=2_docker_sql_default \
    taxi_ingest:v001 \
        --user=root \
        --password=root \
        --host=pgdatabase \
        --port=5432 \
        --db=ny_taxi \
        --table_name=yellow_taxi_trips \
        --url=${URL}
```

# 1_terraform_gcp
## setup prep
* Install terraform : https://www.terraform.io/downloads
* Create account in Cloud Provider account: https://console.cloud.google.com/
* Create a project in GCP
* IAM & Admins > Service Account > create service account
    * name it
    * grant viewer permission
    * done
* click on 3 dots in the SA > manage keys > keys tab > create new key > json
* it downloads the key file
* install gcloud SDK -> https://cloud.google.com/sdk/docs/install-sdk
    * extract at documents
    * also do the login step login to the new created project
    * every time want to login, must open terminal on instalation folder
* in the terminal create a variable pointing to credential json file
```
export GOOGLE_APPLICATION_CREDENTIALS="/Users/cassiobolba/Documents/GitHub/taxi-pipeline/google_credentials.json"
```
* test it, authenticate it with login in google
```
gcloud auth application-default login
```
* this is OAuth
* go to IAM > IAM > edit the servie accoun user > add 2 roles: Storage Admin , Storage Object Admin and Big Query Admin
* Enable these APIs for your project:
    * https://console.cloud.google.com/apis/library/iam.googleapis.com
    * https://console.cloud.google.com/apis/library/iamcredentials.googleapis.com

## using tf
* files
    * .terraform-version : define version
    * main.tf : used to manage resources
    * variables.tf : store variables
* main commands
    * terraform init: Initializes & configures the backend, installs plugins/providers, & checks out an existing configuration from a version control
    * terraform plan: Matches/previews local changes against a remote state, and proposes an Execution Plan.
    * terraform apply: Asks for approval to the proposed plan, and applies changes to cloud
    * terraform destroy: Removes your stack from the Cloud