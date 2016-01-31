# Main Tutorial 

We are gathered here today, because we want to have multiple containers running on Amazon Web Services (AWS) and Elastic Beanstalk.
Reason for using Elastic Beanstalk, is beacuse it integrates with Travis - and we want our automatic builds and deploys.

I wanted to share some common tools (really just commands) that will help us troubleshoot - when (NOT if) we 
find ourselves in hot water. These are good to know, as they will help you extract some information about 
your containers and give you ways to try to troubleshoot. 

#Toolset - Commands   
Here is the list of tools that will come in critical in troubleshooting. The tutorial we are about to do, deploys a 
simple Node server app, however these will come useuful when you are trying to standup your apps :) 
    
| Command/Location | Description |
| --- | --- |
| cat /ect/hosts  | Check the hosts on the VM  |
| docker inspect <container Name|| ID> | JSON data about your container  |
| eb local run  | Run your app locally on boot2docker VM, before testing it on AWS |
| eb local status  | Shows containers IP and address and the open ports |
| eb local open  | Opens the application in a web browser and exits |
| curl http://localhost:51678/v1/tasks  | Not on your local machine, but when you are inside AWS and - see ### Inspect tasks under  "/ecs-agent", |


## Navigating AWS API Documentation 
It maybe difficult to find certain references inside the API documentation at times, I would recommend becoming familiar with the following terms, as 
these will help you narrow down your search criteria, when you are looking for specific information.
    
| Command/Location | Description |
| --- | --- |
| Task Definition | This is your Dockerrun.aws.json v2 (reference is below) - file that setups up tasks required for AWS to setup your instances |
| EB | cli that commnicates with Elastic Beanstalk - and helps you setup an envrionment via Commands line | 
    
Btw, this is unrelated with the demo - but logo from Docker Machine is Awsome :)
![Awesome Docker machine image](https://github.com/docker/machine/blob/master/docs/img/logo.png) 


## STEP0 --- FROM git TO Elastic Beanstalk in 5 minutes
In the first part of this tutorial, we will begin by setting a sample application, and deploying it to AWS via 
Elastic Beanstalk. After we are done with STEP0, we will add travis. 

### Clone our example repo to your local machine 

```
git clone https://github.com/georgebatalinski/newrepo.git
```
        
1. We will be using the Command line in this Tutorial, however you can easily achive all of this - via [WEB CONSOLE](https://console.aws.amazon.com/elasticbeanstalk)

All you will have to do:
    1. [Login](https://console.aws.amazon.com/elasticbeanstalk/?region=us-east-1#/applications)
    2. Upper right hand corner - [Create New Application]
        1. Follow the steps that are provided by Amazon 
        NOTE: New Environment: 'Create web server' 
        2. When it asks UPLOAD - click on 'Upload' button - and select only your Dockerrun.aws.json 
            If you decided to go via WEB CONSOLE
                skip to ### []Dockerrun.aws.json v2

2. We will need to setup permissions, in order to allow Elastic Beanstalk to create/manage our instances.
Since AWS uses multiple services to assemble Docker for us, Elastic Beanstalk will need to speak to Amazon ECS container agent, and you we need 
to have the corresponding permissions in IAM. 

Lucky for us, it is simple to do:
    
    1. Paste this into the Policy area in IAM
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
    

If you are new to how to create and add policies 
[Click here for step by step guide](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_ecstutorial.html#create_deploy_docker_ecstutorial_role)
  


   
3. cd into our cloned directory above - and create the environment inside Elastic Beanstalk - eb will do the talking

```
$ cd rate-instructor 
$ eb init
$ eb create dev-env
```

You got it - maybe it took a bit longer over our allocated time of 5 minutes -- but it was not our fault, it was 
all Elastic Beanstalk provisioning the 'dev-env' (it takes time to spin the environment - aren't you glad we did not have 
to do this manually)


### [] Create a file - Dockerrun.aws.json (NOTE: v2 - is for multi-container (more then 1 container)) 
This file, defines the Amazon ECS task definition that are used to configure container instances in the environment.
Amazon ECS Task – are "containerDefinitions" inside our Dockerrun.aws.json, which are run by Elastic Beanstalk 
whenever an instance is added.
[Reference](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html#create_deploy_docker_v2config_dockerrun)


1. Into your newly created - Dockerrun.aws.json -  copy/paste the below JSON 

Dockerrun.aws.json
```
{
  "AWSEBDockerrunVersion": 2,
    "volumes": [
    {
      "name": "db",
      "host": {
        "sourcePath": "/src"
      }
    }
  ],
  "containerDefinitions": [
    {
      "name": "server",
      "image": "georgebatalinski/docker-centos-simple-server-two:latest",
      "essential": true,
      "memory": 128,
      "portMappings": [
        {
          "hostPort": 80, //must be - PORT: 80 - See load balancer below for explanation
          "containerPort": 8080
        }
      ],
      "mountPoints": [
        {
          "sourceVolume": "db",
          "containerPath": "/src/money" //We are only mounting - "volumes" NOT creating new volumes
        }
      ],
      "links": [
        "db"
      ]
    },
    {
      "name": "db",
      "image": "mongo",
      "essential": true,
      "memory": 128,
      "portMappings": [
        {
          "hostPort": 27017,
          "containerPort": 27017
        }
      ],
      "mountPoints": [
        {
          "sourceVolume": "db",
          "containerPath": "/src/money"
        }
      ]
    }
  ]
}
 
```

The biggies:

```
    "volumes": [
    {
      "name": "db",
      "host": {
        "sourcePath": "/src"
      }
    }
  ],
```
Elastic Beanstalk will setup create an empty volume 
and we get benefit for free: 
    1. We can share the volume between containers 
    2. This volume will be independently mounted inside each container 
    4. This volume will persist, even when our containers become stopped - helpful for backup
        making volumes independent of container lifecycle.    
   

LOAD BALANCER - loves port 80
You get the load balancer by default, however it likes to listen to "hostPort": 80
In your Dockerrun.aws.json you can setup the port mapping to like so:

```
"portMappings": [
        {
          "hostPort": 80,
          "containerPort": 8080
        }
      ],
```       
![This is your app with Load Balancer](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-multicontainer-docker-example.png)

NOTE: If you cannot set this up, or require non-default port (meaning are unhappy with port 80) - check out 
[Using Multiple Elastic Load Balancing Listeners](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_ecs.html)
   

### []Test you EB CLI if you are testing you app locally 

If you do not have latest eb cli - then upgrade. REASON: I ran into some configuration issues, beacuse of 
and older version. Do not burn the hours, just update.

To install EB on OSX
```
$osxterm: curl -s https://s3.amazonaws.com/elasticbeanstalk-cli-resources/install-ebcli.py | python
```

[Install EB CLI for other platforms](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html)



Once you have EB installed - lets run/open the project on our local machine.
eb local will use boot2docker - to provide you the application without provisioning any AWS reources.

```
$osxterm: eb local run 
$osxterm: eb local open

```

If it runs on your local machine - you are ready for the big leauges. Move it over to ElasticBeanstalk on AWS.

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

eb local is using boot2docker - to provide you the application without provisioning any AWS reources 
To get into this Virtual Machine on - use ssh. 

```
$osxterm: boot2docker ssh
$: docker ps 
```

If you ran into another issue - Go over the table at the top of this page - and see if you can troubleshoot it 
If not, submit your issue in the Comments 
    

## STEP1 --- FROM git TO Elastic Beanstalk TO travis in 5 minutes 

The missing piece is Travis right now, so lets set it up.

The travis layer is next, our expectation (or where Travis is going to help us):     
    1. Run tests
    2. Build our docker image and push it to public repository [Private Registry Push](https://docs.travis-ci.com/user/docker/)
    3. Move our Dockerrun.aws.json  TO Elastic Beanstalk
    4. Turn green - when sucess :)
    
### The Setup 
Sign up for a Travis account - 
Once you synced your account on https://travis-ci.org/ with your Git repo, the rest is
2 simple steps away 


1. We Create a file  .travis.yml in our local repo

From Line:1 of the file - copy/paste this into .travis.yml
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

2. We are going to add our AWS_ACCESS_KEY_ID && AWS_SECRET_ACCESS_KEY - 
by running the command below, we will be promted for this information and travis will encrypt it for us 

WARNING: Do not upload your AWS_ACCESS_KEY_ID && AWS_SECRET_ACCESS_KEY without encyption to public repos,
or you may get unwanted charges from AWS. (since if others have your root keys - they can use your account as they please)

```
$osxterm: travis setup elasticbeanstalk
%: ....
now you will be promted 
choose to encrypt
....

```

## On your local machine now, set these ENV vars - that Travis will use to publish your docker repo

```
$osxterm: travis env set DOCKER_EMAIL <youremail>
$osxterm: travis env set DOCKER_USERNAME <yourusername>
$osxterm: travis env set <yourpassword>
```


You are setup, push the changes to the repository, and travis will run automatically 
```
git commit -am 'first build via docker on elasticbean'
git push origin master
```

Check your Travis dashboard 
    and see the GREEN status 

Isn't that your favourite color?  :)


## General troubleshooting techniques - for docker containers

### Docker specific 

```
ps - show you running contatainers
logs -- show you logs 
diff      Inspect changes on a container's filesystem
events    Get real time events from the server
history   Show the history of an image
inspect   Return low-level information on a container or image
port      Lookup the public-facing port that is NAT-ed to PRIVATE_PORT

```


### Inspect tasks under  "/ecs-agent",
If you can SSH into your instance, and you see "/ecs-agent" and no other containers running, then you know that somehow your containers are not being build. 

```
$osxterm: sudo docker ps
$osxterm: curl http://localhost:51678/v1/tasks
```
[Introspection](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-agent-introspection.html)

