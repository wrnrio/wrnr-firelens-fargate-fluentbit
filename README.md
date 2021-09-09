# wrnr-firelens-fargate-fluentbit
Custom build of fluentbit image to support WRNR.io attributes. Since Firelens for ECS Fargate does not support including a config file from S3 an image is needed.

**How to Build**
- Download this repo and navigate to wrnr-fluentbit-image
- Build this image<br>
  `docker build -t wrnr-firelens .`
- Sign into AWS ECR with your account
  `aws ecr get-login` or <br/>
  ``` docker login -u AWS https://<accountid>.dkr.ecr.<region>.amazonaws.com -p `aws ecr get-login-password` ```
- Tag your image
  `docker tag wrnr-firelens <accountid>.dkr.ecr.<region>.amazonaws.com/wrnr-firelens:v1`
- Push the image
  `docker push <accountid>.dkr.ecr.<region>.amazonaws.com/wrnr-firelens:v1`
  
**Next Steps**
- Ensure **_Enable Firelens Configuration_** is selected in **_Log Router Integration_** and **_Type_** is _fluentbit_ and **_Image_** is the image pushed above.
- Reference the config file path in `firelensConfiguration` key:
```
"firelensConfiguration": {
     "type": "fluentbit",
     "options": {
     "config-file-type": "file",
     "config-file-value": "/extra.conf"
     }
 }
 ```
