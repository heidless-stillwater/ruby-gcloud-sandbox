# tools/notes
### secret_key_base
```
EDITOR='subl --wait' rails credentials:edit

# generate new secret_key_base
#rake secret

# set production credentials
EDITOR='subl --wait' bin/rails credentials:edit --environment production
```

# deploy Google Cloud - overview
- ### [Running Rails on Google Cloud](https://cloud.google.com/ruby/rails)

## Deploy a Ruby app in a container image to Cloud Run
- ### [Connect to Cloud SQL for PostgreSQL from Cloud Run](https://cloud.google.com/sql/docs/postgres/connect-instance-cloud-run)

- ### [Quickstart: Deploy a Ruby service to Cloud Run with Docker](https://cloud.google.com/run/docs/quickstarts/build-and-deploy/deploy-ruby-service)

- ### [Running Rails on the Cloud Run environment](https://cloud.google.com/ruby/rails/run)

## Run Rails 7 on App Engine flexible environment 
- ### [Run Rails 7 on App Engine flexible environment](https://cloud.google.com/ruby/rails/appengine)

## Getting started with Ruby on Compute Engine 
- ### [Getting started with Ruby on Compute Engine](https://cloud.google.com/ruby/getting-started/getting-started-on-compute-engine)


## Test: Running Rails on the Cloud Run environment

```
export APP_NAME=alpha-blog
#export APP_NAME=rails

echo $APP_NAME
```

```
gcloud init
--
heidless-pfolio-deploy-5
--

gcloud config set project heidless-pfolio-deploy-5

git clone https://github.com/GoogleCloudPlatform/ruby-docs-samples.git

cd ruby-docs-samples/run/rails
bundle install

# Set up a Cloud SQL for PostgreSQL instance
gcloud sql instances create $APP_NAME-instance-0 \
    --database-version POSTGRES_12 \
    --tier db-f1-micro \
    --region europe-west2

gcloud sql databases create $APP_NAME-db-0 \
    --instance $APP_NAME-instance-0

cat /dev/urandom | LC_ALL=C tr -dc '[:alpha:]'| fold -w 50 | head -n1 > dbpassword

gcloud sql users create $APP_NAME-user-0 \
   --instance=$APP_NAME-instance-0 --password=$(cat dbpassword)

# Set up a Cloud Storage bucket
gsutil mb -l europe-west2 gs://heidless-pfolio-deploy-5-$APP_NAME-bucket-0

gsutil iam ch allUsers:objectViewer gs://heidless-pfolio-deploy-5-$APP_NAME-bucket-0

# Store secret values in Secret Manager
EDITOR='subl --wait' rails credentials:edit
--
# aws:
#   access_key_id: 123
#   secret_access_key: 345

# Used as the base secret for all MessageVerifiers in Rails, including the one protecting cookies.
secret_key_base: <GENERATED VALUE>
gcp:
  db_password: IqRvMQKgOaFkuQBeobzzNrkFbxBUYfevvFgIpyNnzQSfSAgSgj
--

# gcloud secrets delete $APP_NAME-secret-0

gcloud secrets create $APP_NAME-secret-0 --data-file config/master.key

gcloud secrets describe $APP_NAME-secret-0

gcloud secrets versions access latest --secret $APP_NAME-secret-0

gcloud projects describe heidless-pfolio-deploy-5 --format='value(projectNumber)'
--
110223146514
--

gcloud secrets add-iam-policy-binding $APP_NAME-secret-0 \
    --member serviceAccount:110223146514-compute@developer.gserviceaccount.com \
    --role roles/secretmanager.secretAccessor

gcloud secrets add-iam-policy-binding $APP_NAME-secret-0 \
    --member serviceAccount:110223146514@cloudbuild.gserviceaccount.com \
    --role roles/secretmanager.secretAccessor

# Connect Rails app to production database and storage

vi .env
--
PRODUCTION_DB_NAME: rails-db-0
PRODUCTION_DB_USERNAME: rails-user-0
CLOUD_SQL_CONNECTION_NAME: heidless-pfolio-deploy-5:europe-west2:rails-instance-0
GOOGLE_PROJECT_ID: heidless-pfolio-deploy-5
STORAGE_BUCKET_NAME: heidless-pfolio-deploy-5-rails-bucket-0
--

# Grant Cloud Build access to Cloud SQL
gcloud projects add-iam-policy-binding heidless-pfolio-deploy-5 \
    --member serviceAccount:110223146514@cloudbuild.gserviceaccount.com \
    --role roles/cloudsql.client

# Deploying the app to Cloud Run
gcloud builds submit --config cloudbuild.yaml \
    --substitutions _SERVICE_NAME=$APP_NAME-svc-0,_INSTANCE_NAME=$APP_NAME-instance-0,_REGION=europe-west2,_SECRET_NAME=$APP_NAME-secret-0

gcloud run deploy $APP_NAME-svc-0 \
     --platform managed \
     --region europe-west2 \
     --image gcr.io/heidless-pfolio-deploy-5/$APP_NAME-svc-0 \
     --add-cloudsql-instances heidless-pfolio-deploy-5:europe-west2:$APP_NAME-instance-0 \
     --allow-unauthenticated








```