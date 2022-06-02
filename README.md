# Knowledge Graph Service
Service to allow for easy deployment and use of databases. Able to import and export data from live databases. WIP. Better docs in the future.


## Task List
### FastAPI
- [X] Get FastAPI server running
- [X] Get global g object working
- [ ] Get auth pre-req working
	- [X] Middleware working
	- [ ] Flesh out the auth bit
- [X] Add SOME FastAPI exception handling
- [X] Create main api file and organize how routes will be organized
- [ ] Completely solve exception handling with FastAPI (don't let any errors out)
	- [X] Catch Exception and TapisErrors 
 	- [X] Validation errors should be formatted better https://stackoverflow.com/a/69720977 Seems useful
	- [ ] Format "integrity errors" from postgres/sqlalchemy.
- [ ] Finish all endpoints
 	- [ ] /pods
		- [X] Create
			- [X] Have models and create messages
		- [X] Get
 	- [ ] /pods/{pod_id}
		- [ ] Del
		- [ ] Put
		- [X] Get
- [X] Validate everything
### Models
- [ ] Pods
	- [X] Create basic model
	- [ ] container_status - attr, gives k8 pod status
	- [ ] data_attached - attr, get data imports
	- [ ] data_requested - attr, get data imports

### Backend
- [X] SQLModel taking care of everything
	- [X] Migrations with alembic
	- [X] Validation with SQLModel
	- [X] TapisModel gives helper functions
	- [X] Site/tenant support
- [X] Channels.py
- [X] Queues.py
- [X] Request_Utils.py
- [ ] Kubernetes_utils.py
	- [X] Remake pod creation
	- [X] Add service creation
	- [ ] Add init container functionality to pod creation
- [X] Spawner.py to create/deal with pods
	- [X] Create pod/service
- [ ] Health.py
    - [X] Monitor k8 pods, update pod in database.
	- [X] Take care of select states
	- [ ] Take care of Shutdown requested pods
	- [X] Ensure k8 pods are in database
	- [ ] Ensure database isn't missing any pods.
- [ ] Have different pod images for each database type
	- [X] Base Neo4j
	- [ ] ...
- [ ] Connect S3
- [ ] Export scripts
- [ ] Init container for S3 import
- [X] Nginx integration
  - [X] Pods with http
  - [X] Pods with tcp
  - [X] Switch between the two
- [X] Database pods deployed
- [X] Server pods deployed
- [X] Docs/Redoc/openapi.json working
### Nice to have
- [X] Makefile
- [ ] Make the README actually good
- [ ] Get config and entry and more root dir files mounted so they can be edited.
	- There's some permissions issues, for example, nothing can execute entry.sh when it's mounted. Config things as well.


# How to use.
This repository, it's Makefile, and any scripts expects an installed and running instance of Minikube along with a true install of kubectl.
This allows for deployment templating, deployment, and cleaning up along with some dev tools and special operations using the variables located in the makefile.

## Minikube installation
Our local development environment relies on running everything using Minikube. The minikube installation guide is below. Additionally, we want to use kubectl to manage the minikube instance. Note: Minikube does have a kubectl instance, we want a system wide kubectl though, so that guide is below.

#### Steps:
1. Install minikube
    - Setup guide is here: https://minikube.sigs.k8s.io/docs/start/
2. Install kubectl system wide
	- Setup guide is here: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux
	- Note: `snap install kubectl --classic` is the simplest method if it is available to you.

## Makefile
This repository relies on a Makefile to build, deploy, template, and delete everything.
The Makefile has a `make help` target, along with a `make vars` target to explain targets and vars set automatically.
To change a particular variable, please go into `Makefile` and make the change by hand.

**You must go into the Makefile and provide the service_password variable.**

```
# Get all commands and descriptions with:
cd /pods_service
make help
```

Generally devs will use `make clean up` over and over again. This will clean up dir and pods, build image, and deploy with minikube.

### Explanation of what's happening during `up`.
The Makefile `up` target is the most complex. This is a light explainer.
`up` takes the deployment-template directory, copies it, replaces (with sed) variables using the Makefile variables (such as image tag, k8 namespace, service_password).
Once the new deployment directory is created. We then run `./burnup`, which starts the following pods: `main`, `health`, `spawner`, `postgres`, `nginx`, `rabbitmq`.
The `main` pod contains the server and also initializes the postgres database (using an alembic migration) and rabbitmq (using rabbitmqadmin script).

### Dev Containers
You can use dev containers with Minikube with the Makefile!

Steps:
- Install minikube, VSCode, Kubernetes VSCode plugin, and Remote-Containers VSCode plugin (0.231.X does NOT work).
- With Kubernetes plugin in the VSCode sidebar:
  - clusters -> minikube -> Nodes -> minikube -> right-click pods-api pod -> Attach Visual Studio Code
    - This could error out. Read the error, but a lot of the time, you just need to refresh the cluster page because you forgot that you redeployed.
- Bonus fun:
  - Setting `DEV_TOOLS` to true in the Makefile will:
    - Mounts `{minikube}/pods_service` to the `pods-api` pod. Meaning you can work with persistent changes in the repo.
      - Particular Mounts:
        - `pods_service/service` -> `/home/tapis/service`
        - `pods_service/entry.sh` -> `/home/tapis/entry.sh`
      - Mounts with minikube require users first mount their volumes to the minikube internal mounts, do that with:
        - `minikube mount ~/pods_service:/pods_service`
        - You will have to keep the command running, no daemon mode.
	- Start Jupyter Lab from within main. Link to lab should be in Make stdout under the up target.
