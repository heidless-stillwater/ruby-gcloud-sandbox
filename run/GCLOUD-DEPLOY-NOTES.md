# tools/notes
### secret_key_base
```
EDITOR='subl --wait' rails credentials:edit

# generate new secret_key_base
#rake secret

# set production credentials
#EDITOR='subl --wait' bin/rails credentials:edit --environment production
```


# deploy Google Cloud - overview
- ### [Running Rails on Google Cloud](https://cloud.google.com/ruby/rails)

## Deploy a Ruby app in a container image to Cloud Run
- ### [Quickstart: Deploy a Ruby service to Cloud Run with Docker](https://cloud.google.com/run/docs/quickstarts/build-and-deploy/deploy-ruby-service)

- ### [Running Rails on the Cloud Run environment](https://cloud.google.com/ruby/rails/run)

## Run Rails 7 on App Engine flexible environment 
- ### [Run Rails 7 on App Engine flexible environment](https://cloud.google.com/ruby/rails/appengine)

## Getting started with Ruby on Compute Engine 
- ### [Getting started with Ruby on Compute Engine](https://cloud.google.com/ruby/getting-started/getting-started-on-compute-engine)


## Test: Running Rails on the Cloud Run environment
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
gcloud sql instances create rails-instance-0 \
    --database-version POSTGRES_12 \
    --tier db-f1-micro \
    --region europe-west2

gcloud sql databases create rails-db-0 \
    --instance rails-instance-0

cat /dev/urandom | LC_ALL=C tr -dc '[:alpha:]'| fold -w 50 | head -n1 > dbpassword

gcloud sql users create rails-user-0 \
   --instance=rails-instance-0 --password=$(cat dbpassword)

# Set up a Cloud Storage bucket
gsutil mb -l europe-west2 gs://heidless-pfolio-deploy-5-rails-bucket-0

gsutil iam ch allUsers:objectViewer gs://heidless-pfolio-deploy-5-rails-bucket-0

# Store secret values in Secret Manager
bin/rails credentials:edit
--
# aws:
#   access_key_id: 123
#   secret_access_key: 345

# Used as the base secret for all MessageVerifiers in Rails, including the one protecting cookies.
secret_key_base: <GENERATED VALUE>
gcp:
  db_password: IqRvMQKgOaFkuQBeobzzNrkFbxBUYfevvFgIpyNnzQSfSAgSgj
--

gcloud secrets create rails-secret-0 --data-file config/master.key

gcloud secrets describe rails-secret-0

gcloud secrets versions access latest --secret rails-secret-0

gcloud projects describe heidless-pfolio-deploy-5 --format='value(projectNumber)'
--
110223146514
--







```