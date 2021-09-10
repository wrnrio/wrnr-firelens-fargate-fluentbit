## A sample app to test WRNR firelens fluentbit integration. ##
We are going to use this [sample](https://github.com/awslabs/ecs-nginx-reverse-proxy/tree/master/reverse-proxy) to run in an ECS Fargate container. Follow instructions to build and push images to ECS repository.

- Create a Test ECS container from ECS -> Task Definitions -> Create new Task Definition -> Type Fargate
- Ensure **_Enable Firelens Configuration_** is selected in **_Log Router Integration_** and **_Type_** is _fluentbit_ and **_Image_** is the image pushed above or the publicly available ECR repo image.

#### Configuring the `reverseproxy` container ####
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
- The Name is `es` so fluentbit can send it like it's ElasticSearch
- The Path is configured to `/api/v1/fluentbit` so WRNR knows where it's coming from

#### Configuring the `log-router` container ####

There are a few metadata tags that WRNR.io requires to know the content type and what parser to apply to it.<br>
The tags are WRNR_TYPE, WRNR_PARSER, WRNR_TAGS. They are explained below.

The container or log router **must** define the following WRNR_ metadata:<br>
- WRNR_TYPE:`<value>` so we know what type it is. Valid values are `syslog3164, accesslog, customlog, trace, kubernetes`

The container or log router may define these environment variables that get sent by fluentbit as additional metadata:<br>
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
 
Tip: Paste the `demo-app-task-definition.json` in **_Configure via JSON_** for the sample app.

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
