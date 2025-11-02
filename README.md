# Crossplane Azure Environment Factory
This project demonstrates how a single declarative definition can orchestrate a complex multi-service Azure environment. It shows the power of Crossplane to encapsulate infrastructure logic, promote self-service environments, and manage everything through the Kubernetes control plane.

## Overview

The project sets up a complete Azure environment including:
- Resource Groups
- Virtual Networks and Subnets
- AKS (Azure Kubernetes Service) clusters
- PostgreSQL databases with private networking
- Azure Key Vault for secrets management
- Role-based access control (RBAC) assignments

## Prerequisites

- A Kubernetes cluster (local or cloud-based)
- `kubectl` configured to access your cluster
- `helm` installed
- Azure CLI (`az`) installed and authenticated
- An Azure subscription with appropriate permissions

## Step-by-Step Setup

### 1. Install Crossplane

Crossplane is installed using Helm. Run the following commands:

```bash
# Add the Crossplane Helm repository
helm repo add crossplane-stable https://charts.crossplane.io/stable

# Update your Helm repositories
helm repo update

# Install Crossplane in the crossplane-system namespace
helm install crossplane --namespace crossplane-system --create-namespace crossplane-stable/crossplane
```

Verify the installation:

```bash
kubectl get pods -n crossplane-system
```

You should see Crossplane pods running.

### 2. Create Azure Provider Configuration and Secret

#### Step 2.1: Authenticate with Azure CLI

First, log in to Azure:

```bash
az login
```

#### Step 2.2: Create a Service Principal

Create a service principal with Owner role for your subscription. Replace `<Subscription ID>` with your actual Azure subscription ID:

```bash
az ad sp create-for-rbac --sdk-auth --role Owner --scopes /subscriptions/<Subscription ID>
```

This command outputs JSON content representing your Azure credentials. Save this output to a file named `azure-credentials.json`.

Example output:
```json
{
  "clientId": "xxx",
  "clientSecret": "xxx",
  "subscriptionId": "xxx",
  "tenantId": "xxx",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManager": "https://management.core.windows.net/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagement": "https://management.core.windows.net:8443/",
  "gallery": "https://gallery.azure.com/",
  "management": "https://management.core.windows.net/"
}
```

#### Step 2.3: Create Kubernetes Secret

Create a Kubernetes secret in the crossplane-system namespace using the credentials file:

```bash
kubectl create secret generic azure-secret -n crossplane-system --from-file=creds=./azure-credentials.json
```

#### Step 2.4: Apply Provider Configuration

Apply the provider configuration that references the secret:

```bash
kubectl apply -f provider-config.yaml
```

This creates a `ClusterProviderConfig` that Crossplane providers will use to authenticate with Azure.

### 3. Install Azure Providers

Apply the provider manifests to install all necessary Azure providers:

```bash
kubectl apply -f provider.yaml
```

This installs providers for:
- Azure core services
- Networking
- Container services (AKS)
- PostgreSQL databases
- Key Vault
- Authorization (RBAC)

### 4. Create Database Password Secret

Create a secret for the database administrator password:

```bash
kubectl create secret generic db-admin-password --from-literal=password='YourSecurePassword123!' -n app-team-a
```

**Note:** Replace `'YourSecurePassword123!'` with a secure password of your choice.

## Next Steps

After completing the setup:

1. Apply the Composite Resource Definition (XRD):
   ```bash
   kubectl apply -f composit-resource-definition.yaml
   ```

2. Apply the Composition:
   ```bash
   kubectl apply -f composition.yaml
   ```

3. Create an environment claim:
   ```bash
   kubectl apply -f environment-claim.yaml
   ```

4. Monitor the resource creation:
   ```bash
   kubectl get managed
   kubectl get composite
   ```

## Troubleshooting

- Ensure your Azure service principal has sufficient permissions
- Check Crossplane logs: `kubectl logs -n crossplane-system deployment/crossplane`
- Verify provider health: `kubectl get providers`
- Check claim status: `kubectl describe environment devenv123 -n app-team-a`
