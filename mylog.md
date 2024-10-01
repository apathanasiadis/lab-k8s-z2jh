## Workflow : `values_rwxrob.yaml` where extraConfig is commented out thats gives different namespaces per user
1. systemctl start, enable, open docker
2. mk start --driver=docker -p jhub  # to start minikube over a docker container
3. k cluster-info  # making sure that k8s cluster is running over the mk container
3. helm repo update  # making sure that helm is rdy
4. k create ns jhub  # create other than default namespace
5. k get namespaces  # verifying that the brand new jhub ns is there together with the default ns
6. k label ns jhub nstype=system  # labeling the jhub ns  
	- Optional: k apply -f rbac.yaml  # apply RBAC configuration  
* # helm repo update
7. helm upgrade --cleanup-on-fail --install jhub jupyterhub/jupyterhub --namespace jhub --values myvalues.yaml  # install the helm chart
8. helm list -A  # verifying the deployment of the helm chart
9. k get pods  # should show the generated pods
10. k get pods -n jhub  # or this one
11. k get namespaces  # verifying that the default and jhub +rest namespaces are there
12. k get services  # or with: -n jhub. it should show that EXTERNAL-IP is in pending status
13. mk tunnel -p jhub  # route it 
14. k get services  # should now show the actual EXTERNAL-IP
15. 127.0.0.1  # should be the one. browse it and log in with an account: (jovyan:jupyter)
16. watch -n 1 kubectl get pods  # to keep track of the pods running every 1 sec. do you see the new user holding a pod? (Must be YES if they dont keep a separate namespace)
17. log-out and log-in with different user, e.g. (test1:test1user)
18. do you see the new user holding a pod?

## Customizing Images:
1. Dockerfile from https://z2jh.jupyter.org/en/stable/jupyterhub/customizing/user-environment.html
2. docker build -t apathanasiadis/customized -f Dockerfile .
3. mk image load apathanasiadis/customized:v0 -p jhub 
4. docker push apathanasiadis/customized:v0
5. change myvalues.yaml file and upgrage helm
6. apparently it works because it's not pushed to private registry. For that another procedure should be followed.

---
### restart commands:
* k rollout restart deployment hub
* watch k get pods  
Upgrading after changing the config (or values) file
* helm upgrade --cleanup-on-fail \
  jhub jupyterhub/jupyterhub \
  --namespace jhub \
  --version=3.3.8 \
  --values myvalues.yaml
  
### clean-up commands:
* helm uninstall jhub || true
* k delete ns jhub || true
* mk delete -p jhub
* k delete pods jupyter-jovyan -n jovyan  # clean up separate namespace user's pod, where first goes the name of the **pod** and second the name of the **namespace**.

### kubectl commands:
* k get configmaps hub -n jhub -o yaml  # prints the config.py file for the namespace
* k exec -it -n jovyan jupyter-jovyan -- bash
* cd ~/.minikube/profiles/jhub -> xo config.json  # checks the minikube config file: can change the ISO file
* KUBE_EDITOR="vim" k edit deployments.apps hub -n jhub  
### helm commands:
* helm get manifest jhub
### minikube commands:
* mk addons list
### docker:
~/Repos/docker-stacks: Makefile <- e.g. all the images

---
## learning/process log:
* only one user server at a time is allowed
* containers have specific JupyterHub requirements
* images *must* include "OCI runtime"
* images *must* inlude `jupyterhub-singleuser` executable
* confirmed home directory persistence across image types
* different image across users! i.e. test1 doesnt see jovyan's files and vice versa

* Error created by plain "Ubuntu" image -> k delete pods jupyter-test1 -n jhub  # to kill the pod
	- However, the user survives, since I loged in with same username (apparently pwd doesnt matter for now) and still had my `file`.


### 20240926 - Next tasks:
* user images must be derived from standard images

* create base/sample Dockerfile for custom images
* automatically mounting shared folders: postStart

