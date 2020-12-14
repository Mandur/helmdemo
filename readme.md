# Understanding Kubernetes and Helm

The goal of these exercices is to give more experience about helm and Kubernetes. Steps manually executed here are typically one the CI are running.

# Prerequisites
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
* [Docker Desktop](https://www.docker.com/products/docker-desktop)
* [Git](https://git-scm.com/)
* (Optional) [VS Code](https://code.visualstudio.com/) with the [Kubernetes extension](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools)

Run the two below commands and don’t forget to add the result to your path:

az aks install-cli
az acr helm install-cli --client-version 3.4.1

On docker desktop, go to setting, Kubernetes, toggle enable Kubernetes.

# Kubernetes
## Experiment with pods
1. Run a pod with the demo application with the command 
`kubectl run nginxdemos/hello`
2. Observe the status of your pods deployment with kubectl get pods or in VS code
3. Once the pod is ready, get its ip address in VS code or with `kubectl get pods -o wide`
4. Open your browser and go to the url, observe the website.
5. Simulate a failure, (Kill the pod) with VS code or with kubectl delete pod hello. Observe that nothing is on the website anymore
6. Run a new pod with the demo application with the command 
`kubectl run nginxdemos/hello`
7. Get the new pod ip address with kubectl get pods -o wide. Observe that the IP adress changed.
8. Observe that everything in Kubernetes is in yaml by writing `kubectl get pods -o yaml`. You will get the yaml definition of your pod

## Namespace isolation
1. Run kubectl get ns
2. Create a namespace using kubectl create ns <ns name>
3. Check that when running kubectl get pods -n <ns name> you don't get any result. All Kubernetes commands are namespace scoped

## Make the stuff reliable with deployment
1. observe the [deployment file](./kubernetes/deployment.yaml)
2. deploy this file using `kubectl apply -f` 
3. Go to the website and observe the things running
4. Scale this deployment to run on three pods and observe where they run (kubectl get pods -o wide)
5. Kill a random pod and observe that the pod get recreated.
6. Observe that each pod has its own ip address and there is no redundancy present.
7. Upgrade the version to 0.3 in your plain yaml
8. Observe your application don't upgrade to the new version as the new pods don't come in as healthy as this version don't exit
9. see the different version of your app using kubectl rollout history
10. rollback the deploymen using kubectl rollout undo

## Understand intra cluster networking with services
1. Create a [service](https://kubernetes.io/docs/concepts/services-networking/service/) in the yaml file to apply an immutable ip address on your deployment. Hint: use the labels to match your filters.
2. Observe the yaml structure with kubectl get svc -o yaml. Observe that service have internal ip address.
3. Access the website on the service ip (taken from above) and observe you can access your pod in round robin.
4. simulate random pod crash and see you can still access your service.

## (Optional) Debugging Kubernetes
1. Use port forward to expose a remote Kubernetes website localy: kubectl port-forward <svc> 80 8080
2. ssh into running pod with `kubectl exec -it <podname> bash`. Check that you can ping another pod from the cluster

# Helm

## Enter helm
1. Observe the very simple helm chart present in the helm folder.
2. Deploy a helm release named demo using helm upgrade demo helm/demo --install
3. Observe that what is in the template folder get deployed. Use helm ls to get information about what got deployed.
4. Move the service yaml you built earlier to the template folder
5. Run again helm upgrade demo helm/demo --install
6. Notice the service got deployed

## Experience with helm and failure
1. Access the application on the service IP
2. Now set the application to use version 0.3 and upgrade using helm upgrade demo helm/demo
3. Observe the application is still here
4. Do kubectl get pods to understands what is happening
5. Rollback to the previous deployment using helm rollback demo 1
6. Observer the application is sane again
7. Do Kubectl get secret to observe how helm save these changes (those are base 64 deployed secrets)

## Change the helm
1. Parametrize the replicas number and the image name to deploy nginx
2. Redeploy the application
3. Use the command `helm template ` to understand what helm produce and what he sends to kubernetes

## Understand the "public" repository concept 

1. Add the nginx helm repository home with helm repo add nginx-stable https://helm.nginx.com/stable
2. add it and run helm repo update
3. run helm install nginx nginx-stable/nginx-ingress

## Add the private helm repo
1. Add private helm repo using az acr helm repo -n <name>
2. run helm repo update

## Search an app on your repos
1. Let's find the nginx ingress to install. Run helm search repo nginx
2. Notice all the differrent result
3. Include a dependency on the nginx ingress as part of your chart. Hint. YOu need to do that in your chart.yaml