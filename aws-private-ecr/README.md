# THESE STEPS ARE NOT SUPPORTED AT TIME OF THIS WRITING

Build custom runtime:

cloudera/dex/dex-spark-runtime-3.5.4-7.3.2.0

docker-private.infra.cloudera.com/cloudera/dex/dex-spark-runtime-3.5.4-7.3.2.0@sha256:0193352e392791b26e8af87cce64925ad9915ab435e0e2623279eca7a29fd909

Create ECR Private repository:

```
aws ecr create-repository \
  --repository-name  cde-custom-runtime-3.5.4\
  --region us-east-2
```

{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-2:981304421142:repository/cde-custom-runtime-3.5.4",
        "registryId": "981304421142",
        "repositoryName": "cde-custom-runtime-3.5.4",
        "repositoryUri": "981304421142.dkr.ecr.us-east-2.amazonaws.com/cde-custom-runtime-3.5.4",
        "createdAt": "2026-07-22T19:46:06.179000-03:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}

```
aws ecr get-login-password --region us-east-2 | \
  docker login --username AWS --password-stdin 981304421142.dkr.ecr.us-east-2.amazonaws.com/cde-custom-runtime-3.5.4 
```

```
docker build --network=host -t cde-custom-runtime-3.5.4 . -f Dockerfile
```

```
docker tag cde-custom-runtime-3.5.4:latest \
  981304421142.dkr.ecr.us-east-2.amazonaws.com/cde-custom-runtime-3.5.4:v1.0.0 
```

```
docker push \
  981304421142.dkr.ecr.us-east-2.amazonaws.com/cde-custom-runtime-3.5.4:v1.0.0 
```

```
aws ecr describe-images \
  --repository-name cde-custom-runtime-3.5.4\
  --region us-east-2
```

CDE Steps

```
cde resource create --type="custom-runtime-image"   --image-engine="spark3"   --name="cde-custom-runtime-ecr"    --image="981304421142.dkr.ecr.us-east-2.amazonaws.com/cde-custom-runtime-3.5.4:v1.0.0" --vcluster-endpoint https://vsz86c7r.cde-wbj77dp4.se-sandb.a465-9q4k.cloudera.site/dex/api/v1
```

```
cde spark submit  sparkapp.py  --runtime-image-resource-name=cde-custom-runtime-ecr   --vcluster-endpoint https://vsz86c7r.cde-wbj77dp4.se-sandb.a465-9q4k.cloudera.site/dex/api/v1
```
