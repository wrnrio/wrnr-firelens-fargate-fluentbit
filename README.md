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
#### Configuring the container agent or container
The following environment variables that are default for 'es' apply here too.
```
"logConfiguration": {
    "logDriver": "awsfirelens",
    "options": {
        "Path": "/api/v1/fluentbit",
        "Type": "my_type",
        "TLS.Verify": "off",
        "Port": "443",
        "Host": "<your docker host running WRNR.io or native host>",  
        "Index": "my_index",
        "TLS": "On",
        "Name": "es"
    }
}
 ```
 
 Note a few things:<br>
 - The Path is configured to `/api/v1/fluentbit` so WRNR knows where it's coming from
 - The Type is 'es' so fluentbit can send it like it's Elastic Search

There are a few tags that WRNR.io requires to know the content type and what parser to apply to it.<br>
The tags are WRNR_TYPE, WRNR_PARSER, WRNR_TAGS. They are explained below.

The container or log router **must** define the following WRNR_ variables:<br>
- WRNR_TYPE:`<value>` so we know what type it is. Valid values are `syslog3164, accesslog, customlog, trace, kubernetes`

The container or log router may define these environment variables that get sent by fluentbit as additional keys:<br>
- WRNR_PARSER:`<value>` a custom parser to apply to this type. Only valid for `customlog` and `accesslog`.
- WRNR_TAGS:`<tags>` in the format `"tag1=value1, tag2=value2"` to send custom tags. <br>Eg: `region=west, role="db", runmode=prod, app="credit finance"`

A complete config for log router may look like this:
```
{
    "image": "774964239202.dkr.ecr.us-east-1.amazonaws.com/wrnr-firelens:v1",
    "firelensConfiguration": {
        "type": "fluentbit",
        "options": {
            "config-file-type": "file",
            "config-file-value": "/extra.conf"
        }
    },
    "name": "log_router"
    "environment": [
        {
            "name": "WRNR_PARSER",
            "value": "nginx"
        },
        {
            "name": "WRNR_TYPE",
            "value": "accesslog"
        }
    ]  
}
 ```
 
We have provided a sampe definition. Paste the `demo-app-task-definition.json` in **_Configure via JSON_** as a starting point

Note: If sending JSON data, none of the tags are needed. It can set the endpoint to `/api/v1/json`. To send JSON data to a custom parser the endpoint is `/api/v1/json/<parser>` where `parser` is defined and applied to end point and is active.

**Viewing it from WRNR.io**

Once you have successfully configured Fargate and the container is running, do the following:
1. Get the IP address of the the container
2. `curl http://<ip-address>/api/health-check`
3. This should send the `nginx` access log line to WRNR.io
4. Login to WRNR.io console
5. Enter this query in the query window and see results `(#type=='accesslog')`
6. There's more documentation from the **Docs** menu

**Other steps**  

Install WRNR.io in your OwnVPC. Read the docs [here](https://github.com/wrnrio/wrnr-docker-ownvpc).
