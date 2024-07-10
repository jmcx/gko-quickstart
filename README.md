# gko-quickstart

Get started quickly with Gravitee Kubernetes Operator (GKO).

This quickstart assumes that you already have an APIM environment available somewhere, and that you would now like to use GKO to manage APIs in APIM in an automated & declarative way. 
This is what the majority of Gravitee users are doing with GKO today.

## Start a Kubernetes cluster

You can do this locally on your machine or using a cloud provider. 

There are a number of ways to do this on your machine, you can use kind, k3d, or Minikube as examples. 

With Minikube, you'll need to first make sure have Docker on your machine, then install Minikube. Once installed, you can start a cluster in the terminal with the following command:

```sh
minikube start
```

## Install GKO

To keep things simple, install GKO in the default namespace with the following command:

```sh
helm install graviteeio-gko graviteeio/gko
```

## Connect GKO to APIM

### Create a ManagementContext resource

Create a file called `management-context-1.yaml` and enter the following contents:

```yaml
apiVersion: gravitee.io/v1alpha1
kind: ManagementContext
metadata:
  name: management-context-1
spec:
  baseUrl: <APIM management API URL>
  environmentId: DEFAULT
  organizationId: DEFAULT
  auth:
    bearerToken: xxxx-yyyy-zzzz
```

We'll fill out the details later.

### Create a service account and token in APIM for GKO

* To do this, head to the organisation settings in APIM, create a new user, and choose Service Account.
* The service account email is optional. 
* Next, ensure that this service account has the API_PUBLISHER role on the desired environment. This will provide GKO with the minimum set of required permissions in order to be able to manage APIs, applications, and other required assets in APIM.
* From the newly created service account, scroll to the Tokens section at the bottom of the page and create a new token
* Make sure to immediately copy your new personal access token as you won’t be able to see it again.
* You can now use this token as credentials in the ManagementContext you created in the previous step

### Update the ManagementContext

Update the `baseUrl` to point to your installation's management API URL.

Update the `bearerToken` to use the token you created in the previous step.

Create the ManagementContext resource with the following command:

```sh
kubectl apply -f management-context-1.yaml
```

## Create an API with GKO

Create a file called `echo-api-declarative.yaml` and enter the following contents:

```yaml
apiVersion: gravitee.io/v1alpha1
kind: ApiDefinition
metadata:
  name: echo-api-declarative
spec:
  name: "Echo API Declarative"
  contextRef: 
    name: "management-context-1"
    namespace: "default"
  version: "1"
  state: "STARTED"
  lifecycle_state: "PUBLISHED"
  description: "Gravitee Kubernetes Operator sample"
  plans:
    - name: "KEY_LESS"
      description: "FREE"
      security: "KEY_LESS"
  proxy:
    virtual_hosts:
      - path: "/echo-declarative"
    groups:
      - endpoints:
          - name: "Default"
            target: "http://gravitee-echo-api.gravitee.svc.cluster.local:80"
  local: false
```

Notice that this API definition references the ManagementContext we just created. 

Create the resource with the following command:

```sh
kubectl apply -f echo-api-declarative-1.yaml
```

