# Week 1
## 2_docker_sql
### steps
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

## 1_terraform_gcp
### setup prep
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
export GOOGLE_APPLICATION_CREDENTIALS="/Users/cassiobolba/Documents/Zoomcamp/week_1_basics_n_setup/1_terraform_gcp/ny-taxi-374008-ffbe77c5507c.json"
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

# Week 2
## Aiflow Concepts
* webserver:
* UI: 
* Scheduler: trigger and schedule dags
* worker: execute the dags according to scheduler order
* Metadata DB: Store data about dag states and other infos
* DAG directory: folder with dag files

## Considerations
### Why build own image?
* To install google cloud SDK on airflow container
* To install custom python libraries to be used

## Steps
* on folder week_2* create the folder google/credentials and place the credentials there
```
mkdir -p google/credentials/
```
* replace the variables GCP_PROJECT_ID and GCP_GCS_BUCKET on docker compose
* run `docker compose up` to start

## setup nofrills
* doing in another folder `airflow-lite`
* change the values in the env file, and rename it to .env, not .env-example ( or do what they ask)
* run the image

## debug
### google/credentials/google_credentials.json not found
Solution was to map the full file path in my machine, under volumes, in the x-airflow-common section.
```yml
  volumes:
    - ./dags:/opt/airflow/dags
    - ./logs:/opt/airflow/logs
    - ./plugins:/opt/airflow/plugins
    - /Users/cassiobolba/Documents/Zoomcamp/week_2_data_ingestion/airflow/google/credentials/:/.google/credentials:ro
```
And in my mac, I needed to go to `system preferences > security > privacy tab > files and folders` , find docker in the list and check the box to allow it to access.

## Ingest to Local PG
* change the dag mappingfrom `dags` to `dags_local`
* run `docker networks ls` to discover airflow docker network
* add in the compose file running pg the config to map the airflow network

# Week 3 
## Big Query
* De Facto DW in GCP
* Can process massive amount of data
* Price is:
    * on demand: $5 per TB processed
    * flat rate: 100 slots -> $2000 / month -> 400TB 
* Partitioning
    * paritition a table by creation_date, and use creation data on where clause
    * it improves performance and cost
* Clustering
    * pre defined grouping of very common filter used in queries
    * it can then cluster by near any column
    * use categorical values
    * ex: dimenions with very good data distribution