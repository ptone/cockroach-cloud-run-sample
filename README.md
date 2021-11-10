# Connecting Cloud Run to Cockroach Cloud
## Before you begin

 - Create a cluster in GCP at https://cockroachlabs.cloud/
 - Follow the instructions to get to be able to connect to the database from your local environment

## Setup steps

Set an environment variable for your project
    
    export PROJECTID=<your project>
    gcloud config set project $PROJECTID


Create service account and secrets:

    gcloud iam service-accounts create cockroach-demo

    cp $HOME/.postgresql/root.crt cockroach-cluster.crt


    echo "postgresql://USERNAME:xxxx@free-tier.gcp-us-central1.cockroachlabs.cloud:26257/defaultdb?sslmode=verify-full&sslrootcert=/var/secrets/cluster-cert.crt&options=--cluster%3Dgcp-demo-ptone-2993"  | gcloud secrets create cockroach-connection-uri --data-file=-  --replication-policy=automatic

Note above, that this Connection string references the server cert at a specific path we will place the cert in for the docker image

    gcloud secrets add-iam-policy-binding cockroach-connection-uri \
        --member=serviceAccount:cockroach-demo@${PROJECTID}.iam.gserviceaccount.com \
        --role=roles/secretmanager.secretAccessor

Build and push the image:

    gcloud builds submit --tag gcr.io/$PROJECTID/cockroach-run-demo

or 

    docker build -t gcr.io/$PROJECTID/cockroach-run-demo .
    docker push gcr.io/$PROJECTID/cockroach-run-demo

Deploy:

    gcloud beta run deploy cockroach-demo --region us-central1 --allow-unauthenticated \
    --project $PROJECTID \
    --platform managed \
    --service-account=cockroach-demo@${PROJECTID}.iam.gserviceaccount.com \
      --set-secrets="DB_URI=cockroach-connection-uri:latest" \
      --image=gcr.io/${PROJECTID}/cockroach-run-demo:latest 


When the deploy operation finishes, you should now have a fully application taking advantage of fully managed platforms for both compute and data.

