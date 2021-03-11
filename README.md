# SimpleWebKV

[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)

A simple web KV Store in Flask with GET SET and WATCH

# Running the Project

**PRE-REQS**: Have docker installed on the system setup and working fine. This project uses makefile so ensure that make utils are installed. 

Also ensure that you have kustomize, skaffold and minikube installed locally.


```bash
git clone https://github.com/jkdihenkar/SimpleWebKV.git
cd SimpleWebKV
make kube_deploy
```

Check the deployment:

```
[jay@localhost SimpleWebKV]$ make kube_deploy
Generating tags...
 - jkdihenkar/simplewebkv -> jkdihenkar/simplewebkv:e21fc7a-dirty
Checking cache...
 - jkdihenkar/simplewebkv: Not found. Building
Found [minikube] context, using local docker daemon.
Building [jkdihenkar/simplewebkv]...
Sending build context to Docker daemon  13.82kB
Step 1/7 : FROM python:3.9.0
 ---> 0affb4652fc0
Step 2/7 : EXPOSE 5000
...
...
Successfully tagged jkdihenkar/simplewebkv:e21fc7a-dirty
Tags used in deployment:
 - jkdihenkar/simplewebkv -> jkdihenkar/simplewebkv:ad46245b9c2ec65df1ecf543cc2ae8c4dcf33bb9e80ffb94fffb84091d5cb557
Starting deploy...
WARN[0018] image [jkdihenkar/simplewebkv] is not used by the deployment 
 - service/simplewebkv configured
 - deployment.apps/simplewebkv configured
Waiting for deployments to stabilize...
 - test-ci:deployment/simplewebkv: creating container simplewebkv
    - test-ci:pod/simplewebkv-7959c9cdc6-j6qbj: creating container simplewebkv
 - test-ci:deployment/simplewebkv is ready.
Deployments stabilized in 5.661649676s
You can also run [skaffold run --tail] to get the logs

```

Get the service from Minikube or your Kube Cluster:

```
$ kubectl -n test-ci get service -o wide
NAME          TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE   SELECTOR
simplewebkv   NodePort   10.98.155.85   <none>        8991:32444/TCP   88m   app=simplewebkv

$ minikube service simplewebkv --url -n test-ci
http://192.168.49.2:32444

```

Simple test the project is running fine.

```
$ curl -d '{"arg1":"val1"}' -XPUT http://192.168.49.2:32444/put/hello  
{"result":"success"}
```

### Testing the Watch Feature

Once the app is running - Open a new terminal and run - 
```
curl -N http://192.168.49.2:32444/watch
```

Now do some operations from another terminal window - 

```
curl -d '{"arg1":"val1"}' -XPUT http://192.168.49.2:32444/put/hello
```

You will start seeing events of the "curl -N watch" terminal.

### Description of the Contents of the Code files

* `simplekv.py` : core KV store functionalities
* `simplekv_server.py` : flask kvstore service
* `simplekv_cli_client.py` : CLI that interacts with the flask store service
* `test_simplekv_server.py` : Tests for kvstore web service
* `test_simplekv.py` : Test for core kv store utils

## Simple CLI Client Usgaes

Connect to the running container with - 

```
docker exec -it f59dc4e /bin/bash
```

And test out the CLI - 

```

# PUT
$ python simplekv_cli_client.py put  hello 456
success

# GET
$ python simplekv_cli_client.py get hello
456

# WATCH
$ python simplekv_cli_client.py watch
PUT :: INIT STORE cli_store
PUT :: Set hello = 456 in cli_store
...
...

# Unsupported command
$ python simplekv_cli_client.py set hello 456
ERROR: Cannot find key: set
Usage: simplekv_cli_client.py <command>
  available commands:    get | put | watch

For detailed information on this command, run:
  simplekv_cli_client.py --help

```

== END OF DOC == nothing
