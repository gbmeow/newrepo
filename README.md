# Main Tutorial 

You want to create a multi-container application on deploy it on AWS? CHECK 
You want to be a baller and run your infrastructure with only one button click? CHECK 
You want to have some fun setting up AWS? CHECK 

#Toolset 
    Here is the list of tools that will come in critical in troubleshooting - becuase we know 
    that its not all peachy. This setup should be pretty straightforward, as long as all the pieces cooporate 
    Docker, AWS, Travis, Node ...
    
| Command/Location | Description |
| --- | --- |
| cat /ect/hosts  | Check the hosts on the VM  |
| eb local run  | Run your app locally on boot2docker VM, before testing it on AWS |
| eb local status  | Shows containers IP and address and the open ports |
| eb local open  | Opens the application in a web browser and exits |
| curl http://localhost:51678/v1/tasks  | Not on your local machine, but when you are inside AWS and - see ### Inspect tasks under  "/ecs-agent", |

    
Btw, this is unrelated with the demo - but this logo is awsome :)
![Awesome Docker machine image](https://github.com/docker/machine) 

## GIT -> ElasticBeanStalk 

First thing to note, that if you are going to use use load balancer - which you will get by default 
    you must configure your Dockerrun.aws.json to - PORT: 80 

### Dockerrun.aws.json v2 - CHECKMARK
    Configuration file for Multicontainer docker 
    NOTE: v2 - is for multi-container (more then 1) - otherwise you can use the v1 - definiton 
    [Reference](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html#create_deploy_docker_v2config_dockerrun)

Generate your run json (You can also use the AWS wizard - at https://console.aws.amazon.com/elasticbeanstalk)

Here is the sample
```
<INSERT REAL FILE CONTENT>
 //you must configure your Dockerrun.aws.json to - PORT: 80 

```

TO NOTE

```
    "volumes": [
    {
      "name": "db",
      "host": {
        "sourcePath": "/src"
      }
    }
  ],
  
This needs to be defined at the top - Reasons: 
    EB will setup this as a volume - so you can share this between instances/containers 
    This volume will be independently mounted inside each container 
    Each container will specify its mount point 
     "mountPoints": [
        {
          "sourceVolume": "db",
          "containerPath": "/src/money"
        }
      ],
      
Think about it this way 
    "volumes" - defines volumes that should be created 
    containers - are only allowed to mount - "volumes" - that were created inside "volumes"
    <ACTION - EASIER Explanation>

```

[Persisting data across volumens](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_data_volumes.html)



### Navigating their API Documentation 
    It can be painful to use the API at times, I would recommend getting familiarized with the categories, 
    as these will help you narrow down your search criteria, when you are looking for specific information
        Task Definition 
        EB 
        AWS CLI 


### Test local - via EB - CHECKMARK

Check your version of EB 
NOTE: To install on Install the EB CLI on OS X 10.7 or later

```
$osxterm: curl -s https://s3.amazonaws.com/elasticbeanstalk-cli-resources/install-ebcli.py | python
```
IF: you are not running OSX or have diffrent version 
[Install EB CLI](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html)

```
Jerzys-MacBook-Air:newrepo jerzybatalinski$ eb --version
EB CLI 3.7.3 (Python 2.7.1)

```


Requirements:
    Create your project direcotry - as a Git repository

Init your EB 
```
$osxterm: eb init
Follow these steps [Steps here](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-configuration.html)
```

#### Eb local 
If you have your Dockerrun.aws.json v2
and your images are public/or private 
you can run this command 

```
$osxterm: eb local run 

```

In another tab (or if you know the IP to the VM - go directly to the webbrowser)

```
$osxterm: eb local open

```

Once this is running locally, you have a high chance that it will run on Elastic Bean Stalk - however, that is not guaranteed,
but if you can open you app - that is a good indication that it will run. 

##### TROUBLESHOOTING: If you cannot open the app 

```
$osxterm: eb local status 

%: ....
Container name: elasticbeanstalk_server_1
Container ip: 192.168.59.103
Container running: False
Exposed host port(s): None
Full local URL(s): None
...

```
If Exposed host port(s): 80 OR Full local URL(s): 192.168.59.103:80 are not available - you containers are not running 

EB LOCAL is using boot2docker - to provide you the application without provisioning any AWS reources 
So you could troubleshoot it - by ssh into the boot2docker machine (while the continer is running) 

```
$osxterm: boot2docker ssh
```


## GIT -> Travis -> ElasticBeanStalk  










##Test you EB CLI if you are testing you app locally 

```
this is how you install it 
http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html#eb-cli3-install-osx

```

## Key troubleshooting techniques - for Multi-container docker containers

### Docker commands 

```
ps - show you running contatainers
logs -- show you logs 

---other
diff      Inspect changes on a container's filesystem
events    Get real time events from the server
history   Show the history of an image
inspect   Return low-level information on a container or image
port      Lookup the public-facing port that is NAT-ed to PRIVATE_PORT
version   Show the Docker version information

```


### Inspect tasks under  "/ecs-agent",

```
I cannot see the 2 containers that should have been created 
		all I get "Name": "/ecs-agent",
			http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-agent-introspection.html
		CMD: curl http://localhost:51678/v1/tasks
        
```
If you have Tasks: [] - that means you have no containers running 

```

SSH into the remote - using your DNS (do not forget sudo)


```
[ec2-user@ip-172-31-30-105 ~]$ curl http://localhost:51678/v1/tasks
{"Tasks":[{"Arn":"arn:aws:ecs:us-east-1:362999620065:task/bf2d131c-9810-41bb-9edb-7569d19f3e35",
"DesiredStatus":"RUNNING","KnownStatus":"RUNNING","Family":"awseb-dev-env-7n2n5sxhpm","Version":"1",
"Containers":[{"DockerId":"73e047d8b8b4de8aa8dd4fdbbe5c07efed85a20118d4821cf09c79ca1d04f41c",
"DockerName":"ecs-awseb-dev-env-7n2n5sxhpm-1-db-888ad2c1e6bed5c23c00","Name":"db"},
{"DockerId":"72401d9988ab0b981136ecce83e1fff5bcb97cf06d0f68c8b0c6cd719b0d83bc",
"DockerName":"ecs-awseb-dev-env-7n2n5sxhpm-1-server-f2d6f68ac2bbd1c78e01","Name":"server"}]}]}[ec2-user@ip-172-31-30-105 ~]$ 
```

