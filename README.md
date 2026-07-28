# THESE STEPS HAVE BEEN TESTED

**List all the image repositories.**

curl -u <userid>:<password> https://container.repository.cloudera.com/v2/_catalog

**List all tags for a specifc image:**

curl -u <userid>:<password> https://container.repository.cloudera.com/v2/cloudera/dex/dex-spark-runtime-3.5.4-7.3.2.0-compat/tags/list

**Build custom runtime:**

cloudera/dex/dex-spark-runtime-3.5.4-7.3.2.0

docker-private.infra.cloudera.com/cloudera/dex/dex-spark-runtime-3.5.4-7.3.2.0@sha256:1234567890123456789012345678901234567890123456789012345678901234

**Create ECR Private repository:**

```
aws ecr create-repository \
  --repository-name  cde-custom-runtime-3.5.4\
  --region us-east-2
```

{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-2:1234567891011:repository/cde-custom-runtime-3.5.4",
        "registryId": "xxxxxxxxx",
        "repositoryName": "cde-custom-runtime-3.5.4",
        "repositoryUri": "1234567891011.dkr.ecr.us-east-2.amazonaws.com/cde-custom-runtime-3.5.4",
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
  docker login --username AWS --password-stdin 1234567891011.dkr.ecr.us-east-2.amazonaws.com/cde-custom-runtime-3.5.4 
```

```
docker build --network=host -t cde-custom-runtime-3.5.4 . -f Dockerfile
```

```
docker tag cde-custom-runtime-3.5.4:latest \
  1234567891011.dkr.ecr.us-east-2.amazonaws.com/cde-custom-runtime-3.5.4:v1.0.0 
```

```
docker push \
  1234567891011.dkr.ecr.us-east-2.amazonaws.com/cde-custom-runtime-3.5.4:v1.0.0 
```

```
aws ecr describe-images \
  --repository-name cde-custom-runtime-3.5.4\
  --region us-east-2
```

**CDE Steps**

```
cde resource create --type="custom-runtime-image"   --image-engine="spark3"   --name="cde-custom-runtime-ecr"    --image="1234567891011.dkr.ecr.us-east-2.amazonaws.com/cde-custom-runtime-3.5.4:v1.0.0" --vcluster-endpoint https://xxxxxx.cde-xxxxxx.se-sandb.xxxxx.cloudera.site/dex/api/v1
```

```
cde spark submit  sparkapp.py  --runtime-image-resource-name=cde-custom-runtime-ecr   --vcluster-endpoint https://xxxxxx.cde-xxxxxx.se-sandb.xxxxx.cloudera.site/dex/api/v1
```
