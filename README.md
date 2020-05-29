# Kubeflow Pipelines Backend for Tekton

Kubeflow Pipelines backend for Tekton. This project is a follow-on to [KFP-Tekton Compiler project](https://github.com/kubeflow/kfp-tekton), and will allow to take the compiled Tekton yaml to Kubeflow Pipelines (KFP) engine and run it through KFP API and UI using the SDK

This project is in very early development phase. Instructions will be laid out as we make more progress with implementation.

# Getting Started
## Prequisites
1. [Install Tekton](https://github.com/tektoncd/pipeline/blob/master/docs/install.md#installing-tekton-pipelines-on-kubernetes) v0.11.3 or above
2. [Install Kubeflow](https://www.kubeflow.org/docs/started/getting-started/) if you want to leverage the Kubeflow stack
3. Clone this repository
    ```
    git clone github.com/kubeflow/kfp-tekton-backend
    cd kfp-tekton-backend
    ```

## Install Tekton KFP with pre-built images
1. Remove the old version of KFP from the Kubeflow stack. Also clean up the old webhooks if you were using KFP 0.4 or above.
    ```shell
    kubectl delete -k manifests/kustomize/env/platform-agnostic
    kubectl delete MutatingWebhookConfiguration cache-webhook-kubeflow
    ```

2. Once the old KFP deployment is removed, run the below command to deploy the new KFP with Tekton backend.
    ```shell
    kubectl apply -k manifests/kustomize/env/platform-agnostic
    ```

    Check the new KFP deployment, it should take about 5 to 10 minutes.
    ```shell
    kubectl get pods -n kubeflow
    ```

    Now go ahead and access the pipeline in the Kubeflow dashboard. It should be accessible from the istio-ingressgateway which is the
    `<public_ip>:31380`

## Building the source code
### Prerequites
1. [NodeJS 12 or above](https://nodejs.org/en/download/)
2. [Golang 1.13 or above](https://golang.org/dl/)

### Frontend
The dev instructions is under the [frontend](/frontend) directory. Below are the commands for building the frontend docker image.
```shell
cd frontend
npm run docker
```

### Backend
The KFP backend with Tekton needs to modify the api-server and persistent agent. 
1. To build these two images, clone this repository under the [GOPATH](https://golang.org/doc/gopath_code.html#GOPATH) and rename it to `pipelines`. 
    ```shell
    cd $GOPATH/src/go/github.com/kubeflow
    git clone github.com/kubeflow/kfp-tekton-backend
    mv kfp-tekton-backend pipelines
    cd pipelines
    ```

2. For local binary builds, we can use the `go build` commands
   ```shell
   go build -o apiserver ./backend/src/apiserver
   go build -o agent ./backend/src/agent/persistence
   ```

3. For Docker builds, we can use the below `docker build` commands
   ```shell
   docker build -t api-server -f backend/Dockerfile .
   docker build -t persistenceagent -f backend/Dockerfile.persistenceagent .
   ```

4. Then you can push the images and modify the Kustomization to use your own built images.
    
   Modify the `newName` under the `images` section in `manifests/kustomize/base/kustomization.yaml`.

   Now you can follow the [Install Tekton KFP with pre-built images](#install-tekton-kfp-with-pre-built-images) instructions to install your own KFP backend.
