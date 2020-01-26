# GitOps via Flux with Hyper-V on minikube

## Prerequisits

Note: [Chocolatey](https://chocolatey.org/) will make the setup process easier by a lot and is recommended.

### Hyper-V and Docker

1. Activate the Hyper-V hypervisor via the windows feature menu.
2. Download and install Docker for Windows from [hub.docker.com] (you will need a Docker Hub account for that)
3. Create a network switch which sahres the itnernet connection with the minikube-vm.

### kubectl

1. Install kubectl with chocolatey via `choco install kubernetes-cli`

Alternatively you can install the binaries directly.

2. Download [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows), v1.17.0 can be downloaded directly via [this link](https://storage.googleapis.com/kubernetes-release/release/v1.17.0/bin/windows/amd64/kubectl.exe)
3. Copy the downloaded executable to any folder you like ( f.e. C:\kubectl\)
4. Add the folder to your PATH environmental variable

### Install and start minikube

1. Install minikube via ``choco install minikube``
2. Start minikube with the correct driver and switch by executing `minikube start --vm-driver hyperv --hyperv-virtual-switch "<the name of your switch>"` (this may take a while, especially at first start)
3. Check the status with `minikube status` to see if kubectl is configured correctly

### Install fluxctl

1. Install fluxctl via ``choco install fluxctl``

Alternatively you can install the binaries directly:

2. Download [kubectl binaries](https://github.com/fluxcd/flux/releases)
3. Copy the downloaded executable to any folder you like ( f.e. C:\fluxctl\)
4. Add the folder to your PATH environmental variable


## Configuring and applying Flux to Kubernetes Cluster
This repository comes with an example configuration for a flux deployment inside your kubernetes cluster, as well as a example deployment which you can run as little webservice.

1. Verify that your minikube cluster is started and ready with ``kubectl get all``
2. If you want to configure your own repository open the ``flux-deployment.yaml`` file inside the flux folder. You can change the repository and the path inside the repository which the flux container shall monitor. Please note that repo has to be given as Git ssh url with the format ``git@github.com:<USER>/<REPO>.git``.
3. Apply the flux deployment to the minikube cluster via ``kubectl apply -f ./flux/``. Keep in mind that this command must be run from the main directory of the repository, as all deployments inside the flux container need to be deployed.
4. Check fi the deployments have been deployed by executing ``kubectl get pods`` which should list two pods, one named ``flux`` and one named ``memchached``.

## Configuring the Git repo to be accessed by the flux container

1. Execute ``fluxctl identity`` to get the public key of the Flux Operator.
2. In case of Github, you can add the ssh key you are getting in return to either the repository or your account. To do that go into the settings of the repository (or your account) and navigate to the subsetting ``Deploy keys``, where you can add the Flux' operator key.

## Add a configuration for you service deployment

The ``deploy`` subdirectory in this repository is empty. Copy and paste the two yml configuration files from the ``example`` directory into this subdirectory and push yoru changes to the repository. You should be able to see the Flux cluster doing its work by deploying the service and the example deployment inside your cluster.

You can check if the webservice is working properly by obtaining minikubes IP adress via ``minikube ip`` and the port is default configured to be 31111. Type ``<minikube ip>:31111`` into your browser and you should see the JaxRS Aries webpage. With ``<minikube ip>:31111/World`` it should return ``Hello World`` 

## Automate the deplyoment

1. List all workloads via ``fluxctl list-workloads``
2. Automate the deplyoment of new updates with ``fluxctl automate -w deployment/<your_deployment>