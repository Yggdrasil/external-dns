# Set up environment variables
export PROJECT_NAME=<your-google-cloud-project-name>
export CUSTOM_DOMAIN=<your-custom-domain-used-in-knative>
export CLUSTER_NAME=<knative-cluster-name>
export CLUSTER_ZONE=<knative-cluster-zone>

# Create a new service account for Cloud DNS admin role.
# Name of the service account you want to create.
export CLOUD_DNS_SA=cloud-dns-admin

gcloud --project $PROJECT_NAME iam service-accounts \
    create $CLOUD_DNS_SA \
    --display-name "Service Account to support ACME DNS-01 challenge."

# Bind the role dns.admin to the newly created service account.
# Fully-qualified service account name also has project-id information.
export CLOUD_DNS_SA=$CLOUD_DNS_SA@$PROJECT_NAME.iam.gserviceaccount.com

gcloud projects add-iam-policy-binding $PROJECT_NAME \
    --member serviceAccount:$CLOUD_DNS_SA \
    --role roles/dns.admin

# Download the secret key file for your service account.
gcloud iam service-accounts keys create ~/key.json \
    --iam-account=$CLOUD_DNS_SA

# Upload the service account credential to your cluster. This command uses the secret name cloud-dns-key, but you can choose a different name.
kubectl create secret generic cloud-dns-key \
    --from-file=key.json=$HOME/key.json

# Delete the local secret
rm ~/key.json

# Deploy this repo to the cluster
