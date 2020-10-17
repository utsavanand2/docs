# inlets-operator reference documentation

inlets-operator adds public LoadBalancers to your local Kubernetes clusters.

Install the inlets-operator using a single command with [arkade](https://get-arkade.dev/). arkade runs against any Kubernetes cluster and wraps the helm command-line.

Install using arkade:

```bash
arkade install inlets-operator \
 --provider $PROVIDER \ # Name of the cloud provider to provision the exit-node on.
 --region $REGION \ # Used with cloud providers that require a region.
 --zone $ZONE \ # Used with cloud providers that require zone (e.g. gce).
 --token-file $HOME/Downloads/key.json \ # Token file/Service Account Key file with the access to the cloud provider.
 --license $LICESNE # inlets-pro license file. (Omit if using inlets OSS)
 ```

Install using helm:

Checkout the inlets-operator helm chart [README](https://github.com/inlets/inlets-operator/blob/master/chart/inlets-operator/README.md) to know more about the values that can be passed to `--set` and to see provider specific example commands.

```bash
# Create a secret to store the service account key file
kubectl create secret generic inlets-access-key --from-file=inlets-access-key=key.json

# Add and update the inlets-operator helm repo
helm repo add inlets https://inlets.github.io/inlets-operator/

# Update the local repository
helm repo update

# Install inlets-operator with the required fields
helm upgrade inlets-operator --install inlets/inlets-operator \
  --set provider=$PROJECTID,zone=$ZONE,region=$REGION,projectID=$PROJECTID,inletsProLicense=$LICENSE
```

View the code on GitHub: [inlets/inlets-operator](https://github.com/inlets/inlets-operator)

## Create exit node on DigitalOcean

Install with inlets PRO:

```bash
arkade install inlets-operator \
 --provider digitalocean \
 --region lon1 \
 --token-file $HOME/Downloads/do-access-token \
 --license $(cat $HOME/inlets-pro-license.txt)
```

Install with inlets OSS:

```bash
arkade install inlets-operator \
 --provider digitalocean \
 --region lon1 \
 --token-file $HOME/Downloads/do-access-token
```

## Create exit node on EC2

To use the instructions below you must have the AWS CLI configured with sufficient permissions to create users and roles.

* Create a AWS IAM Policy with the following:

Create a file named `policy.json` with the following content

```json 
{
    "Version": "2012-10-17",
    "Statement": [  
        {
            "Effect": "Allow",
            "Action": [
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:DescribeInstances",
                "ec2:DescribeImages",
                "ec2:TerminateInstances",
                "ec2:CreateSecurityGroup",
                "ec2:CreateTags",
                "ec2:DeleteSecurityGroup",
                "ec2:RunInstances",
                "ec2:DescribeInstanceStatus"
            ],
            "Resource": ["*"]
        }
    ]
}
```

Create the policy in AWS 

```bash 
aws iam create-policy --policy-name inlets-automation --policy-document file://policy.json
```


* Create an IAM user

```bash 
aws iam create-user --user-name inlets-automation
```

* Add the Policy to the IAM user

We need to use the policy arn generated above, it should have been printed to the console on success. It also follows the format below.

```bash 
export AWS_ACCOUNT_NUMBER="Your AWS Account Number"
aws iam attach-user-policy --user-name inlets-automation --policy-arn arn:aws:iam::${AWS_ACCOUNT_NUMBER}:policy/inlets-automation
```

* Generate an access key for your IAM User 

The below commands will create a set of credentials and save them into files for use later on.

> we are using [jq](https://stedolan.github.io/jq/) here. It can be installed using the link provided.
> Alternatively you can print ACCESS_KEY_JSON and create the files manually.

```bash 
ACCESS_KEY_JSON=$(aws iam create-access-key --user-name inlets-automation)
echo $ACCESS_KEY_JSON | jq -r .AccessKey.AccessKeyId > access-key
echo $ACCESS_KEY_JSON | jq -r .AccessKey.SecretAccessKey > secret-access-key
```

Install with inlets PRO:

```bash
arkade install inlets-operator \
 --provider ec2 \
 --region eu-west-1 \
 --token-file $HOME/Downloads/access-key \
 --secret-key-file $HOME/Downloads/secret-access-key \
 --license $(cat $HOME/inlets-pro-license.txt)
```

Install with inlets OSS:

```bash
arkade install inlets-operator \
 --provider ec2 \
 --region eu-west-1 \
 --token-file $HOME/Downloads/access-key \
 --secret-key-file $HOME/Downloads/secret-access-key \
```

## Create exit node on Google Compute Engine

If you do not have arkade installed, get it from [here](https://get-arkade.dev/)

It is assumed that you have gcloud installed and configured on your machine.
If not, then follow the instructions [here](https://cloud.google.com/sdk/docs/quickstarts)

To get your service account key file, follow the steps below:

```bash
# Get current projectID
export PROJECTID=$(gcloud config get-value core/project 2>/dev/null)

# Create a service account
gcloud iam service-accounts create inlets \
  --description "inlets-operator service account" \
  --display-name "inlets"

# Get service account email
export SERVICEACCOUNT=$(gcloud iam service-accounts list | grep inlets | awk '{print $2}')

# Assign appropriate roles to inlets service account
gcloud projects add-iam-policy-binding $PROJECTID \
  --member serviceAccount:$SERVICEACCOUNT \
  --role roles/compute.admin

gcloud projects add-iam-policy-binding $PROJECTID \
  --member serviceAccount:$SERVICEACCOUNT \
  --role roles/iam.serviceAccountUser

# Create inlets service account key file
gcloud iam service-accounts keys create key.json \
  --iam-account $SERVICEACCOUNT
```

Install with inlets PRO:

```bash
arkade install inlets-operator \
    --provider gce \
    --project-id $PROJECTID \
    --zone us-central1-a \
    --token-file key.json \
    --license $(cat $HOME/inlets-pro-license.txt)
```

Install with inlets OSS:

```bash
arkade install inlets-operator \
    --provider gce \
    --project-id $PROJECTID \
    --zone us-central1-a \
    --token-file key.json
```

## Create exit node on Linode

Install using helm:

```bash
kubectl apply -f ./artifacts/crds/

# Create a secret to store the service account key file
kubectl create secret generic inlets-access-key --from-literal inlets-access-key=<Linode API Access Key>

# Add and update the inlets-operator helm repo
helm repo add inlets https://inlets.github.io/inlets-operator/

helm repo update

# Install inlets-operator with the required fields
helm upgrade inlets-operator --install inlets/inlets-operator \
  --set provider=linode,region=us-east
```

You can also install the inlets-operator using a single command using [arkade](https://get-arkade.dev/), arkade runs against any Kubernetes cluster.

Install with inlets PRO:

```bash
arkade install inlets-operator \
 --provider linode \
 --region us-east \
 --access-key <Linode API Access Key> \
 --license $(cat $HOME/inlets-pro-license.txt)
```

Install with inlets OSS:

```bash
arkade install inlets-operator \
 --provider linode \
 --region us-east \
 --access-key <Linode API Access Key>
```

## Create exit node on Azure
Prerequisites:

* You will need `az`. See [Install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

Generate Azure auth file 
```sh
az ad sp create-for-rbac --sdk-auth > ~/Downloads/client_credentials.json
```

Install using helm:
```bash
kubectl apply -f ./artifacts/crds/
kubectl create secret generic inlets-access-key --from-file=inlets-access-key=~/Downloads/client_credentials.json
helm repo add inlets https://inlets.github.io/inlets-operator/
helm repo update
helm upgrade inlets-operator --install inlets/inlets-operator \
  --set provider=azure,region=eastus,subscriptionID=<Azure Subscription ID> 
```
