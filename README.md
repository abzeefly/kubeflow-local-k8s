# Running Kubeflow pipelines on your local machine using Kubernetes and Kind
Template to run kubeflow pipelines on a local k8s cluster.
We have a kubeflow pipeline with 3 containerized steps: Ingest, Preprocess and Train. Each of these componentes have a Dockerfile and requirements.txt file to define the container environment for that component. 
The specifications for each components can be found in the components.yaml file.

Pipeline.py orchestrates these componenets into steps and runs the pipeline on the local Kubeflow instance. Steps to setup your local Kubeflow instance are below,

# Steps:
### Install kind (macOs)
 ```brew install kind```

### Create a kind cluster
```kind create cluster --name kind```

### Use your created kind cluster as context
```kubectl config use-context kind-kind```

### Check kubeclt is setup correctly:
```kubectl get pods```

### Setup Kubernetes dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl create serviceaccount -n kubernetes-dashboard admin-user
kubectl create clusterrolebinding -n kubernetes-dashboard admin-user --clusterrole cluster-admin --serviceaccount=kubernetes-dashboard:admin-user
token=$(kubectl -n kubernetes-dashboard create token admin-user)
echo $token
```
### At this point save your token and start a new terminal to run the Kubernetes dahsboard
```kubectl proxy```

### Setup Kubeflow to run on your kind clust
```
export PIPELINE_VERSION=2.0.0
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/cluster-scoped-resources?ref=$PIPELINE_VERSION"
kubectl wait --for condition=established --timeout=60s crd/applications.app.k8s.io
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/env/platform-agnostic-pns?ref=$PIPELINE_VERSION"
```

### Start your local Kubeflow UI by port-forwarding
```kubectl port-forward -n kubeflow svc/ml-pipeline-ui 8080:80```

### Install all python dependencies to run pipeline.py
```pip3 install -r kfp/requirements.txt```

### Run your Kubeflow pipeline on your local Kubenetes Cluster !!
```python3 kfp/pipeline.py```

