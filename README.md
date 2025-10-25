# crossplane-azure-environment-factory
In this project we will demonstrate how to create Azure Managed Resources with Crossplane.

helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane --namespace crossplane-system --create-namespace crossplane-stable/crossplane


create azure provider config
az login
az ad sp create-for-rbac --sdk-auth --role Owner --scopes /subscriptions/<Subscription ID> # this command will give you a json content that represent azure credentails

kubectl create secret generic azure-secret -n crossplane-system --from-file=creds=./azure-credentials.json
kubectl apply -f provider-config.yaml

