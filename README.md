# Configuration  Steps
## Prequite
* aws cli installed
* kubectl installed

1. Create a iam user and attach inline policy as below. 
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetRepositoryPolicy",
                "ecr:DescribeRepositories",
                "ecr:ListImages",
                "ecr:DescribeImages",
                "ecr:BatchGetImage",
                "ecr:GetLifecyclePolicy",
                "ecr:GetLifecyclePolicyPreview",
                "ecr:ListTagsForResource",
                "ecr:DescribeImageScanFindings"
            ],
            "Resource": "*"
        }
    ]
}
```

2. Get the security credentials for the user created
```
AWS_ACCESS_KEY_ID="replaceme"
AWS_SECRET_ACCESS_KEY="replaceme"
AWS_DEFAULT_REGION="region"
```

3. Create a kubernetes secret for the ecr registry
```
kubectl create secret generic ecr-creds-test --namespace replaceme  --from-literal=AWS_DEFAULT_REGION=replaceme --from-literal=AWS_SECRET_ACCESS_KEY=replaceme --from-literal=AWS_ACCESS_KEY_ID=replaceme --from-literal=ACCOUNT=replaceme  --from-literal=SECRET_NAME=regcred
```

4. There is a `cronjob.yaml` manifest in zip folder which containers cronjob resource.
`NB: replace namesapce section with your application namespace` 
Apply the resource

```
kubectl apply -f cronjob.yaml && kubectl get cronjobs
```

5. for the first time you need to trigger job manually as first job will trigger after 6hours
```
kubectl create job --from=cronjob/ecr-cred-helper ecr-cred-helper-manual
```

6. By now a job pod needs to running and a secret regcred should have been created
```
kubectl get secret --namespace replaceme
```

7. Finally update deployment with the secret regcred as below,as imagePullsecrets parameter:

 `For example`
 ```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: test
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: 852421539909.dkr.ecr.ap-southeast-1.amazonaws.com/app:1.0
        imagePullPolicy: Always
        ports:
          - name: web
            containerPort: 80
      imagePullSecrets:
          - name: regcred
```




