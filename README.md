# Configuration  Steps
## Prequite
* aws cli installed
* kubectl installed

1. Make sure your SSO account has the following permissions. 
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

2. Authenticate aws cli . Go to SSO page and Copy the and paste the aws secret keys for 
```
export AWS_ACCESS_KEY_ID="replaceme"
export AWS_SECRET_ACCESS_KEY="replaceme"
export AWS_SESSION_TOKEN="replaceme"
```
3. Make sure authentication works 
```
aws sts get-caller-identity
```

4. Export env for your account. copy paste with you values
```
export AWS_ACCOUNT=<aws account id>
export AWS_REGION=<ECR region>
export NAMESPACE=<namespace of your application>
```
5. Create a kubernetes secret for the ecr registry
```
kubectl create secret docker-registry regcred  \
--docker-server=${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com \
--docker-username=AWS \
--docker-password=$(aws ecr get-login-password) \
  --namespace=$NAMESPACE || true
```

6. Once the secret is created,  update your k8s deployment to use this secret, as imagePullsecrets parameter:

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




