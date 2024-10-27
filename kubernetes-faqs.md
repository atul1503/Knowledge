## Always put version tag on each image that you push to a container registry

**Reason** This is because if you want to revert the changes, you would know which commit was responsible for the bug. But if you tag each image with the git version tag, we will know exactly which code is buggy.

## What to do on existing CI/CD pipeline?

`Ans` If the pipeline is already set then run the test cases and if everything runs fine then update the deployment.yaml file with the new git tag as container image tag and push to ecr and then use
```
kubectl apply -f deployment.yaml
```
to redeploy deployment object of your app. This will force kubernetes to fetch your specified image from repository.

## How to monitor status of updates of deployment?

`Ans` We use `kubectl rollout status deployment/my-app` to check the status of changes done to a deployment like whether all replicas have been updated with the new changes or not etc.

## How to rollback?
`Ans` use `kubectl rollout undo deployment/my-app` to rollback the changes to the latest previous deployment.

## How does kubernetes know of any change?

`Ans` It detects change only when the configuration files that we `apply` with kubectl have some change from the configuration when the last `apply` ran successfully.  

## How to refer to my k8s cluster when I run kubernetes cli tools like `kubectl`?

`Ans` To do this:
1. First install `kubectl` using either `brew` or from `curl`.
2. Move kubeconfig yaml file to your `$HOME/.kube/config` folder. Or you can export `KUBECONFIG` env variable with the path of your kubeconfig yaml file.

## How to write a basic deployment yaml comfiguration?
`Ans` Like this:

```
apiVersion: apps/v1 # deployment uses apps/v1
type: Deployment
metadata:
  name: MyDeployment
  labels:
    app: mera
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dusra
  template:
    metadata:
      labels:
        app: dusra
    spec:
      containers:
        - name: mycont
          image: ubuntu:alpine
          command: ["sleep","5000"]
```

For deployment, inside spec it only need to know:
1. How many replicas of pods.
2. How to select existing pods: For that we have `spec.selector.matchLabels` to select pods based on labels on those pods. Name should not be provided.
3. How to create pods: For that we have `spec.template`, in which you have specificy `metadata` of pod and its own specs. In their metadata no need to provide `name`.

The spec of pods should have `containers` to specify what containers should run in a pod and what should be their image and what they will run.


## How to write a basic service yaml configuration file?

`Ans` Like this:

```
apiVersion: v1
kind: Service
metadata:
  name: myservice
  labels: # give label if you want to
    key: value
spec:
  type: ClusterIP #clusterIP for pod to pod and LoadBalancer for external communications via cloud provided external load balancer .
  selector:
    app: myapp # This should match the labels of the Pods you want to expose.
  ports:
    - protocol: TCP #or udp or whatver, TCP alwasy works.
      port: 80 #port of the service.
      targetPort: #port of the pod to which requests will go.

```
in service, `spec.selector` is required since you want the service to select pods which it will expose. And this selector doesnot need `matchLabels` and you directly specify labels. `matchSelector` and other matchers are only needed in deployment config.
