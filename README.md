## Tutorial 
    Use load balancing - PORT: 80 


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

SSH into the remote - using your DNS (do  not forget sudo)


LOCAL 
    ???

```


[ec2-user@ip-172-31-30-105 ~]$ curl http://localhost:51678/v1/tasks
{"Tasks":[{"Arn":"arn:aws:ecs:us-east-1:362999620065:task/bf2d131c-9810-41bb-9edb-7569d19f3e35",
"DesiredStatus":"RUNNING","KnownStatus":"RUNNING","Family":"awseb-dev-env-7n2n5sxhpm","Version":"1",
"Containers":[{"DockerId":"73e047d8b8b4de8aa8dd4fdbbe5c07efed85a20118d4821cf09c79ca1d04f41c",
"DockerName":"ecs-awseb-dev-env-7n2n5sxhpm-1-db-888ad2c1e6bed5c23c00","Name":"db"},
{"DockerId":"72401d9988ab0b981136ecce83e1fff5bcb97cf06d0f68c8b0c6cd719b0d83bc",
"DockerName":"ecs-awseb-dev-env-7n2n5sxhpm-1-server-f2d6f68ac2bbd1c78e01","Name":"server"}]}]}[ec2-user@ip-172-31-30-105 ~]$ 


### EB 
Run your app locally, before testing it on AWS

```
run -- runs local 
status --- look for containers and the open ports
open -- see if you can open 


```

### Check the hosts on the VM 
```
/ect/hosts - you should see the connected containers here 
```


### Awesome Docker machine image    
https://github.com/docker/machine