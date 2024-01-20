# k8s-cli-notes

Find the Current Context of Kubectl 
```
kubectl config current-context
```

Find All Context
```
kubectl config get-contexts

```

Switch the current Context
```
kubectl config use-context <Context Name>
```


Kubectl Config Set-Context Namespace
```
kubectl config set-context --current --namespace <Namespace Name>
```

Kubectl Config Set-Context Context, User, Namespace
```
kubectl config set-context <Context name> --user=<User Name> --namespace <Namespace Name>
```

Find All Pods in the current namespace
```
kubectl get po
```

Get special Pods yaml information
```
kubectl get po <Pod Name> -o yaml
```

Describe Pod information
```
kubectl describe po mongodb-0
```

Find Endpoints in the current namespace
```
kubectl get ep 
```

Create Service Account
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: <Service Account Name>
  namespace: <Namespace Name>
EOF
```
OR
```
kubectl create serviceaccount -n <Namespace Name> <Service Account Name>
```

Create Service Account Token
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: <Service Account Token Name>
  namespace: <Namespace>
  annotations:
    kubernetes.io/service-account.name: <Service Account Name> 
type: kubernetes.io/service-account-token
EOF
```

Clusterrolebinding 
```
kubectl create clusterrolebinding <clusterrolebinding Name> --clusterrole=cluster-admin --serviceaccount=<Namespace Name>:<Service Account Name>
```

Get Clusterrolebinding
```
kubectl get clusterrolebinding 
```
OR
```
kubectl get clusterrolebinding -o wide
```


Create special service account kubeconfig

```
# create /.kube/config

# The script returns a kubeconfig for the ServiceAccount given
# you need to have kubectl on PATH with the context set to the cluster you want to create the config for

# Cosmetics for the created config
clusterName='<Cluster Name>'
# your server address goes here get it via `kubectl cluster-info`
server='<Cluster Endpoint>'
# the Namespace and ServiceAccount name that is used for the config
namespace='<Service Account Namespace>'
serviceAccount='<Service Account>'

# The following automation does not work from Kubernetes 1.24 and up.
# You might need to
# define a Secret, reference the ServiceAccount there and set the secretName by hand!
# See https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#manually-create-a-long-lived-api-token-for-a-serviceaccount for details
secretName='<Service Account Token>'

######################
# actual script starts
set -o errexit


ca=$(kubectl --namespace="$namespace" get secret/"$secretName" -o=jsonpath='{.data.ca\.crt}')
token=$(kubectl --namespace="$namespace" get secret/"$secretName" -o=jsonpath='{.data.token}' | base64 --decode)

echo "
---
apiVersion: v1
kind: Config
clusters:
  - name: ${clusterName}
    cluster:
      certificate-authority-data: ${ca}
      server: ${server}
contexts:
  - name: ${serviceAccount}@${clusterName}
    context:
      cluster: ${clusterName}
      namespace: ${namespace}
      user: ${serviceAccount}
users:
  - name: ${serviceAccount}
    user:
      token: ${token}
current-context: ${serviceAccount}@${clusterName}
" > kube-config.yaml
```


Create secrets
```
# create secret
kubectl create secret generic <Secret Name> -n <Namespace Name> --from-file=./kube-config.yaml
```

Print Logs
```
# print the logs for the last 6 hours
kubectl logs --since=6h <pod_name>

# follow the logs
kubectl logs -f <pod_name>


# print the logs for a container in a pod
kubectl logs -c <container_name> <pod_name>

# print the logs for a previously failed pod
kubectl logs --previous <pod_name>
```