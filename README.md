### Deploying a tac_plus-ng application into a Kubernetes

#### Building the container image for tac_plus-ng application

Set private docker registry as environment variable 

```Bash
export DOCKER_REGISTRY="<DOCKER_REGISTRY_IP-ADDRESS/DOMAIN>:<DOCKER_REGISTRY_PORT>"
```

*As example:*

```Bash
export DOCKER_REGISTRY="dev-nexus.lan:8123"
```

> [!Tip]
> In test purpose can be used Sonatype Nexus Repository Manager in Docker running locally

Build Docker image from the Dockerfile

```Bash
docker build -f Dockerfile -t "${DOCKER_REGISTRY}/tac-plus-ng-k8s:latest" .
```

> [!NOTE]
> You should avoid using the `:latest` tag when deploying containers in production as it is harder to track which version of the image is running and more difficult to roll back properly

Check new image in list

```Bash
docker images --filter reference="${DOCKER_REGISTRY}/tac-plus-ng-k8s"
```

Manually test docker image run

```Bash
docker run -d --rm -p 49:49 -v "$(pwd)/tac_plus-ng:/usr/local/etc" -v "/var/log/tac_plus-ng:/var/log/tac_plus-ng" --env-file "$(pwd)/tac_plus-ng/tac_plus-ng.env" dev-nexus.lan:8123/tac-plus-ng-k8s
```

#### Push the container image into docker registry

Login into private private docker registry

```Bash
docker login -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD" "$DOCKER_REGISTRY_URI"
```

*As example:*

```Bash
docker login -u admin http://dev-nexus.lan:8123/repository/dev-nexus/
```

Push image into private docker registry

```Bash
docker push "${DOCKER_REGISTRY}/tac-plus-ng-k8s:latest"
```

### Deploy a tac_plus-ng application in k8s

#### Creating docker registry secret

Creating Secret `nexus-pull-secret` to pull an image from a private container image registry

```Bash
kubectl create secret docker-registry nexus-pull-secret \
  --docker-server="$DOCKER_REGISTRY" \
  --docker-username="$DOCKER_USERNAME" \
  --docker-password="$DOCKER_PASSWORD"
```

> [!NOTE]
> If run on minikube and using insecure registry, minikube should be started as:
> `minikube start --insecure-registry="192.168.1.0/24"`
> 
> Set insecure-registry value equal docker registry network in CIDR format 

#### Deploy a tac_plus-ng application in Pod

##### Create resources from manifests

```Bash
kubectl create -f resource-manifests/tac-plus-ng-configmap.yaml
```

```Bash
kubectl create -f resource-manifests/tac-plus-ng-secret-env.yaml
```

>[!Tip]
> To perform the base64 encoding use:
> `echo -n $SECRET_DATA | base64`

```Bash
kubectl create -f resource-manifests/tac-plus-ng-service.yaml
```

```Bash
kubectl create -f resource-manifests/tac-plus-ng-pod.yaml
```

> [!NOTE]
> If run on minikube to connect to LoadBalancer services need execute:
> `minikube tunnel --bind-address='192.168.1.12'`
> 
> bind-address is ip-address to bind tunnel 

To preview the object that would be sent to cluster, without really submitting it used `--dry-run=client` flag

As example preview ConfigMap object that will be created at yaml format

```Bash
kubectl create -f resource-manifests/tac-plus-ng-configmap.yaml --dry-run=client -o yaml
```

##### Check out the state of created resources

```Bash
kubectl get secrets
```

```Bash
kubectl get services
```

```Bash
kubectl get configmaps
```

```Bash
kubectl get pods
```

To show details of a specific resource or group of resources used `kubectl describe`

As example, show Pod details

```Bash
kubectl describe pod tac-plus-ng
```

##### Execute a command in a container

 Get output from running the `env` command in tac-plus-ng-app container from Pod tac-plus-ng

```Bash
kubectl exec tac-plus-ng --container tac-plus-ng-app -- env
```

Get a Shell to a Running Container

```Bash
kubectl exec --stdin --tty tac-plus-ng --container tac-plus-ng-app -- /bin/bash
```

#### Create a Deployment for tac_plus-ng application

Delete Pod `tac-plus-ng` created above

```Bash
kubectl delete pods tac-plus-ng
```

Create Deployment resource from manifest

```Bash
kubectl create -f resource-manifests/tac-plus-ng-deployment.yaml
```

Checkout created deployments

```Bash
kubectl get deployments.apps
```

##### References

> References
> 
> 1. https://www.freecodecamp.org/news/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882
> 2. https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_secret_docker-registry/
> 3. https://kubernetes.io/docs/concepts/workloads/pods/
> 4. https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
> 5. https://kubernetes.io/docs/concepts/configuration/secret/
> 6. https://kubernetes.io/docs/tutorials/services/connect-applications-service/
> 7. https://kubernetes.io/docs/concepts/configuration/configmap/
> 8. https://nixhub.ru/posts/k8s-configmap-basics/
> 9. https://ru.werf.io/guides/rails/200_real_apps/080_config.html
