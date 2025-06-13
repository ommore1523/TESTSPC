# TESTSPC



# Explain About OIDC and IRSA with simple example
# Objective: Give aws services access to pods


# Lets Learn It by example :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: s3-test-pod
spec:
  containers:
    - name: aws-cli
      image: amazon/aws-cli:latest
      command: ["sleep", "infinity"]
```

- Login into Pod and Try to check if you can do following command;
- `kubectl exec -it s3-test-pod -- aws s3 ls`

- `OBSERVATION` : you are not able to do it



Now,
- Enable OIDC Provider for Your EKS Cluster
- create IAM Policy : all s3 bucket access
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::*",
        "arn:aws:s3:::*/*"
      ]
    }
  ]
}

```
- Attach This Policy to an IAM Role (IRSA Role)
    1. Go to IAM Console → Roles → Create role.

    2. Trusted entity: Select Web identity.

    3. Choose the OIDC provider for your EKS cluster.

    4. Audience: sts.amazonaws.com.

    5. Permissions: Attach the `FullS3AccessIRSA` policy you just created.

    6. Click Next, name the role `EKS-IRSA-FullS3Access`, and click Create role.


- Create Service account with attached role
    ```yaml
    # irsa-s3-sa.yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
    name: irsa-s3
    namespace: default
    annotations:
        eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/EKS-IRSA-FullS3Access   # make changes here
    ```

- kubectl apply -f irsa-s3-sa.yaml


- Create one pod with this new Service Account attached with it and same image
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: s3-test-pod
    spec:
    serviceAccountName: irsa-s3  # <---- service account
    containers:
        - name: aws-cli
        image: amazon/aws-cli
        command: ["sleep", "3600"]
    ```