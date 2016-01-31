# Main Tutorial 

You want to create a multi-container application on deploy it on AWS? CHECK 
You want to be a baller and run your infrastructure with only one button click? CHECK 
You want to have some fun setting up AWS? CHECK 

#Toolset - Commands (that will save your ass)   
Here is the list of tools that will come in critical in troubleshooting - becuase we know that its not all peachy.
This setup should be pretty straightforward, as long as all the pieces cooporate Docker, AWS, Travis, Node ...
    
| Command/Location | Description |
| --- | --- |
| cat /ect/hosts  | Check the hosts on the VM  |
| docker inspect <containerName> | Check important info about container  |
| eb local run  | Run your app locally on boot2docker VM, before testing it on AWS |
| eb local status  | Shows containers IP and address and the open ports |
| eb local open  | Opens the application in a web browser and exits |
| curl http://localhost:51678/v1/tasks  | Not on your local machine, but when you are inside AWS and - see ### Inspect tasks under  "/ecs-agent", |


## Navigating AWS API Documentation 
It can be painful to use the API at times, I would recommend getting familiarized with the categories, as 
these will help you narrow down your search criteria, when you are looking for specific information.
    
| Command/Location | Description |
| --- | --- |
| Task Definition | This is your Dockerrun.aws.json v2 (reference is below) - file that setups up tasks required for AWS to setup your instances |
| EB | cli that commnicates with Elastic bean - and helps you setup an envrionment via Commands line | 
    
Btw, this is unrelated with the demo - but logo from Docker Machine  :)
![Awesome Docker machine image](https://github.com/docker/machine/blob/master/docs/img/logo.png) 


## GIT -> ElasticBeanStalk 

1. First thing to note, that if you are going to use use load balancer - which you will get by default 
    you must configure your Dockerrun.aws.json to - "hostPort": 80
    The container that will be public facing - its port mapping should be as follows

![This is your app with Load Balancer](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-multicontainer-docker-example.png)

```
"portMappings": [
        {
          "hostPort": 80,
          "containerPort": 8080
        }
      ],
```       

NOTE: If you cannot set this up, or require non-default port (meaning are unhappy with port 80) - check out 
   [Using Multiple Elastic Load Balancing Listeners](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_ecs.html)
   
        
2. We will be using the Command line in this Tutorial, however you do not need to 
    you can easily achive all the - via [WEB CONSOLE](https://console.aws.amazon.com/elasticbeanstalk)
    All you will have to do:
    1. [Login](https://console.aws.amazon.com/elasticbeanstalk/?region=us-east-1#/applications)
    2. Upper right hand corner - [Create New Application]
        1. Follow the steps that are provided by Amazon 
        NOTE: New Environment: 'Create web server' 
        2. When it asks UPLOAD you Dockerrun.aws.json via the web upload button 
            The creation of this file - we cover in step 1 (so that was rquired for you) 
             - so still do step 2 with us (and feel free to skip other steps)

3. We need some permissions 
    Since amazon using multiple services to assemble Docker for us, and beacuse we need 
    Elastic Beanstalk to speak to Amazon ECS container agent, we must add corresponding permissions in IAM. 
    Lucky for us, it is simple to do:
    
I did not want to paste in Amazon steps here, so checkout the sections:
    Create the Policy
    Create the Role (if needed)
[Click here to see above sections](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_ecstutorial.html#create_deploy_docker_ecstutorial_role)
  

1. Create a policy in IAM
```
{
  "Version": "2016-02-01",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecs:StartTask",
        "ecs:StopTask",
        "ecs:RegisterContainerInstance",
        "ecs:DeregisterContainerInstance",
        "ecs:DescribeContainerInstances",
        "ecs:DiscoverPollEndpoint",
        "ecs:Submit*",
        "ecs:Poll"
       ],
      "Resource": ["*"]
    },
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::elasticbeanstalk-*/resources/environments/logs/*"
    }
  ]
}
```

2. Create the Role
    and attach the above Policy to it 

   
4. Setup your directory and create the environment in which your application will live 
```
$ mkdir HelloWorld
$ cd HelloWorld
$ eb init -p PHP
$ echo "Hello World" > index.html
$ eb create dev-env
$ eb open
```

### []Dockerrun.aws.json v2 - CHECKMARK
This file, dfines the Amazon ECS task definition that is used to configure container instances in the environment.
        Amazon ECS Task – are "containerDefinitions" inside our Dockerrun.aws.json
       Elastic Beanstalk initiates a new task whenever an instance is added.
    Configuration file for Multicontainer docker 
    NOTE: v2 - is for multi-container (more then 1)
    [Reference](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html#create_deploy_docker_v2config_dockerrun)

Paste the below JSON 

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
    1. EB will setup this as a volume - so you can share this between instances/containers 
    This volume will be independently mounted inside each container 
    Each container will specify its mount point 
     "mountPoints": [
        {
          "sourceVolume": "db",
          "containerPath": "/src/money"
        }
      ],
    2. At least one volume is required, because EB will generate log files for our containers 
        and they will need to be stored somewhere.
    
      
To understand that is happening:
   At the top of the page -  "volumes" - defines volumes that should be created 
    it is where you create the volumes. VOLUMES are created and will persist data, even if you deleted the container
    iteself, the volumes are independent of container lifecycle.
    
   Inside the "containerDefinitions": 
     it is where  - are only allowed to mount - "volumes" - that were created above 
        here is not where you are creating volumens - it is only for mounting 
        

you are good - you have your Dockerrun.aws.json


### []Test you EB CLI if you are testing you app locally 

Note - if you do not have latest EB - then upgrade. REASON: I ran into some configuration issues, beacuse of 
and older version. Do not burn the hours, just update.

Install EB 
```
$osxterm: eb --version
EB CLI 3.7.3 (Python 2.7.1)
```

To install EB on OSX
```
$osxterm: curl -s https://s3.amazonaws.com/elasticbeanstalk-cli-resources/install-ebcli.py | python
```

[Install EB CLI for other platforms](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html)


Once you have EB installed - lets run it 

```
$osxterm: eb local run 

```

In another tab (or if you know the IP to the VM - go directly to the webbrowser)

```
$osxterm: eb local open

```

Once this is running locally, you have a high chance that it will run on Elastic Bean Stalk - however, that is not guaranteed,
but if you can open you app - that is a good indication that it will run. 

If it runs on your local machine 

```
$osxterm: eb deploy

```

#### TROUBLESHOOTING: If you cannot open the app 

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
$: docker ps 
```

If you ran into another issue - 
    Go over the table at the top of this page - and see if you can troubleshoot it 
    If not, submit your issue in the Comments 
    

## GIT -> Travis -> ElasticBeanStalk  

The travis layer is next, our expectation (or where Travis is going to help us:     
    1. Run tests
    2. Build our docker image and push it to public repository (this can be private too)
    3. Send our Dockerrun.aws.json - to EB 
    4. Turn green - when sucess
    
## The Setup 
We need to work with Travis - once you synced your account on https://travis-ci.org/ with your Git repo, the rest is
2 simple steps away 

## We Create a file  .travis.yml 

And fill out the sections, once manually and the second by command helper.

From Line:1 of the file - paste this is in: 
```
sudo: required
language: node_js
node_js:
- '4.1'
git:
  depth: 1
env:
  DOCKER_COMPOSE_VERSION: 1.5.0
services:
- docker

before_install:
- docker login -e="$DOCKER_EMAIL" -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
- docker build -t georgebatalinski/docker-centos-simple-server-two:latest .
- docker push georgebatalinski/docker-centos-simple-server-two

```

2nd part - authanticated and secrete keys repo - names - save your selft time and let travis generate that for you 

```
$osxterm: travis setup elasticbeanstalk
```

## On your local machine now, set these ENV vars - that tavis will use 

Jerzys-MacBook-Air:newrepo jerzybatalinski$ travis env set DOCKER_EMAIL george.batalinski@gmail.com
[+] setting environment variable $DOCKER_EMAIL
Jerzys-MacBook-Air:newrepo jerzybatalinski$ travis env set DOCKER_USERNAME georgebatalinski
[+] setting environment variable $DOCKER_USERNAME
Jerzys-MacBook-Air:newrepo jerzybatalinski$ travis env set DOCKER_PASSWORD 
[+] setting environment variable $DOCKER_PASSWORD


Now 
```
git commit -am 'first build via docker on elasticbean'
git push origin master
```

Now you just wait for it to run green  - isn't that your favourite color?  





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

SSH into the remote - using your DNS 
    and see if containers are running 

```
[ec2-user@ip-172-31-30-105 ~]$ curl http://localhost:51678/v1/tasks
{"Tasks":[{"Arn":"arn:aws:ecs:us-east-1:362999620065:task/bf2d131c-9810-41bb-9edb-7569d19f3e35",
"DesiredStatus":"RUNNING","KnownStatus":"RUNNING","Family":"awseb-dev-env-7n2n5sxhpm","Version":"1",
"Containers":[{"DockerId":"73e047d8b8b4de8aa8dd4fdbbe5c07efed85a20118d4821cf09c79ca1d04f41c",
"DockerName":"ecs-awseb-dev-env-7n2n5sxhpm-1-db-888ad2c1e6bed5c23c00","Name":"db"},
{"DockerId":"72401d9988ab0b981136ecce83e1fff5bcb97cf06d0f68c8b0c6cd719b0d83bc",
"DockerName":"ecs-awseb-dev-env-7n2n5sxhpm-1-server-f2d6f68ac2bbd1c78e01","Name":"server"}]}]}[ec2-user@ip-172-31-30-105 ~]$ 
```

