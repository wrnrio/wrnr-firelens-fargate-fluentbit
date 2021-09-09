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
 
 or,
 use prebuilt image from `774964239202.dkr.ecr.us-east-1.amazonaws.com/wrnr-firelens:v1` (transfer charges may apply)
  
**Next Steps**
- Create a Test ECS container from ECS -> Task Definitions -> Create new Task Definition -> Type Fargate
- Ensure **_Enable Firelens Configuration_** is selected in **_Log Router Integration_** and **_Type_** is _fluentbit_ and **_Image_** is the image pushed above or the publicly available ECR repo image.
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
 - Paste the `demo-app-task-definition.json` in **_Configure via JSON_** as a starting point

**Viewing it from WRNR.io**

Once you have successfully configured Fargate and the container is running, do the following:
1. Get the IP address of the the container
2. `curl http://<ip-address>/api/health-check`
3. This should send the `nginx` access log line to WRNR.io
4. Login to WRNR.io console
5. Enter this query in the query window and see results `(#type=='accesslog')`
6. There's more documentation from the **Docs** menu

**Other steps**  

Install WRNR.io in your OwnVPC. Read the docs [here](https://github.com/wrnrio/wrnr-docker).
