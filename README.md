# Loki Integration on AWS STS with the S3 located in the seperate account.
Here the Assumption is Loki is deployed in Account A and AWS S3 is deployed in account B
Documentation use
- https://docs.aws.amazon.com/eks/latest/userguide/cross-account-access.html
- https://aws.amazon.com/blogs/containers/cross-account-iam-roles-for-kubernetes-service-accounts/
- https://aws.amazon.com/blogs/containers/fine-grained-iam-roles-for-red-hat-openshift-service-on-aws-rosa-workloads-with-sts/

### Steps
 1. Create the S3 object storage in the second AWS account.
 2. Made the changes in the secret for s3 bucket to use it on the loki
 ```
 oc create -f logging-loki-s3.yaml
 ```
 3. Establish the Trust relationship between the Loki service accounts and Role which we are planning to use in the second account.
 - Create an IAM OIDC provider for the cluster in Account B 
 - Assign IAM roles to the service accounts.
 - Annotate the service account on the cluster with the ROLE arn
 
 4. Create an IAM OIDC Provider for the cluster in Account B
 ```
 Find the OIDC URL and Audience from the Cluster A and make the new OIDC provider in CLuster B with the same OIDC url and audience.
 ```
 5. Assign IAM roles to the service accounts
 ```
 Create a new IAM role for the loki access and give full s3 access to it 
 Create trust_policy.yaml with the role ARN of the OIDC provider from the Account B.
 Add all the service accounts which need access to this account in the StringEquals to establish the trust relationship between the service account and the role 
 ```
 ```
 oc create -f trust_policy.yaml
 ```
 
 6. Annotate the Loki service acocunts with the Account B role ARN 
 ```
   oc annotate serviceaccount logging-loki eks.amazonaws.com/role-arn=arn:aws:iam::AccountB:role/<RoleName>
   oc annotate serviceaccount logging-loki-ruler eks.amazonaws.com/role-arn=arn:aws:iam::AccountB:role/<RoleName>
   oc annotate serviceaccount logcollector eks.amazonaws.com/role-arn=arn:aws:iam::AccountB:role/<RoleName>

e.g 
oc annotate serviceaccount logging-loki-ruler eks.amazonaws.com/role-arn=arn:aws:iam::617563276761:role/loki
 ```
 7. Redeploy the loki pods

 8. Check inside the loki pods if env variable is showing the Account B labels.
