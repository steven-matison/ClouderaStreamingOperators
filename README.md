# ClouderaStreamingOperators
This reposistory is used in installing [Cloudera Streaming Operators](https://cldr-steven-matison.github.io/blog/Cloudera-Streaming-Operators/).


## Prerequisites
  
* [Docker](https://docs.docker.com/desktop/)
* [Minikube](https://minikube.sigs.k8s.io/docs/)
* [Helm](https://helm.sh/)
* [Cloudera Operator License](https://lighthouse.cloudera.com/)
  
## Resources

* [Cloudera Streams Messaging (CSM) 1.6 Docs](https://docs.cloudera.com/csm-operator/1.6/index.html)
* [Cloudera Streaming Analytics (CSA) 1.5 Docs](https://docs.cloudera.com/csa-operator/1.5/index.html)
* [Cloudera Flow Management (CFM) 3.0 Docs](http://docs.cloudera.com/cfm-operator/3.0.0/index.html)

## Terminal Commands

### Minikube and Helm
```terminal
# Start minikube
minikube start --memory 16384 --cpus 6

# Install cert manager
helm install cert-manager jetstack/cert-manager --version v1.16.3 --namespace cert-manager --create-namespace --set installCRDs=true

# Login to Cloudera Helm Registry
helm registry login container.repository.cloudera.com

# Update Helm Repos
helm repo update
```

### Setup Namespaces & Secrets
```terminal
kubectl create namespace cld-streaming
kubectl create secret generic cfm-operator-license --from-file=license.txt=./license.txt -n cld-streaming
kubectl create secret docker-registry cloudera-creds --docker-server=container.repository.cloudera.com --docker-username=[username] --docker-password=[password] -n cld-streaming
kubectl create namespace cfm-streaming
kubectl create secret generic cfm-operator-license --from-file=license.txt=./license.txt -n cfm-streaming
kubectl create secret docker-registry cloudera-creds --docker-server=container.repository.cloudera.com --docker-username=[username] --docker-password=[password] -n cfm-streaming
```

### Install Kafka Cloudera Strimzi Operator
```terminal
helm install strimzi-cluster-operator --namespace cld-streaming --set 'image.imagePullSecrets[0].name=cloudera-creds' --set-file clouderaLicense.fileContent=./license.txt --set watchAnyNamespace=true oci://container.repository.cloudera.com/cloudera-helm/csm-operator/strimzi-kafka-operator --version 1.6.0-b99
```

### Install Flink CSA Operator
```terminal
kubectl create -f https://github.com/jetstack/cert-manager/releases/download/v1.8.2/cert-manager.yaml
kubectl wait -n cert-manager --for=condition=Available deployment --all
helm install csa-operator --namespace cld-streaming \
    --version 1.5.0-b275 \
    --set 'flink-kubernetes-operator.imagePullSecrets[0].name=cloudera-creds' \
    --set 'ssb.sse.image.imagePullSecrets[0].name=cloudera-creds' \
    --set 'ssb.sqlRunner.image.imagePullSecrets[0].name=cloudera-creds' \
    --set 'ssb.mve.image.imagePullSecrets[0].name=cloudera-creds' \
    --set 'ssb.database.imagePullSecrets[0].name=cloudera-creds' \
    --set-file flink-kubernetes-operator.clouderaLicense.fileContent=./license.txt \
    oci://container.repository.cloudera.com/cloudera-helm/csa-operator/csa-operator 
```

### Install NiFI CFM Operator
```terminal
helm install cfm-operator oci://container.repository.cloudera.com/cloudera-helm/cfm-operator/cfm-operator \
  --namespace cfm-streaming \
  --version 2.11.0-b57 \
  --set installCRDs=true \
  --set image.repository=container.repository.cloudera.com/cloudera/cfm-operator \
  --set image.tag=2.11.0-b57 \
  --set "image.imagePullSecrets[0].name=cloudera-creds" \
  --set "imagePullSecrets={cloudera-creds}" \
  --set "authProxy.image.repository=container.repository.cloudera.com/cloudera_thirdparty/hardened/kube-rbac-proxy" \
  --set "authProxy.image.tag=0.19.0-r3-202503182126" \
  --set licenseSecret=cfm-operator-license \
  --set-file clouderaLicense.fileContent=./license.txt
```

### Install NiFi 1.28.1 CR
```terminal
kubectl apply -f nifi-cluster-2.11-nifi1x.yaml -n cfm-streaming
```

### Install NiFi 2.4.0 CR
```terminal
kubectl apply -f nifi-cluster-2.11-nifi2x.yaml -n cfm-streaming
```

### Install Kafka 
```terminal
kubectl apply --filename kafka-eval.yaml,kafka-nodepool.yaml --namespace cld-streaming
```

### Open the UIs
```terminal
minikube service mynifi-web --namespace cfm-streaming
```
#### Cloudera Surveyor
```terminal
minikube service cloudera-surveyor-service --namespace cld-streaming
```

#### Schema Registry
```terminal
minikube service schema-registry-service --namespace cld-streaming
```

#### SQL Stream Builder
```terminal
minikube service ssb-sse --namespace cld-streaming
```

### Uninstall Commands
```terminal
helm uninstall cfm-operator --namespace cfm-streaming
```

```terminal
helm uninstall cloudera-surveyor --namespace cld-streaming
```

```terminal
helm uninstall strimzi-operator --namespace cld-streaming
```

```terminal
helm uninstall schema-registry --namespace cld-streaming
```

```terminal
helm uninstall csa-operator --namespace cld-streaming
```

```terminal
minikube delete
```