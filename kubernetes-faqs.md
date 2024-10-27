## Always put version tag on each image that you push to a container registry

**Reason** This is because if you want to revert the changes, you would know which commit was responsible for the bug. But if you tag each image with the git version tag, we will know exactly which code is buggy.

## What to do on existing CI/CD pipeline?

**Ans** If the pipeline is already set then run the test cases and if everything runs fine then update the deployment.yaml file with the new git tag as container image tag and push to ecr and then use
```
kubectl apply -f deployment.yaml
```
to redeploy deployment object of your app. This will force kubernetes to fetch your specified image from repository.

## How to monitor status of updates of deployment?

**Ans** We use `kubectl rollout status deployment/my-app` to check the status of changes done to a deployment like whether all replicas have been updated with the new changes or not etc.

## How to rollback?
**Ans** use `kubectl rollout undo deployment/my-app` to rollback the changes to the latest previous deployment.

## How does kubernetes know of any change?

**Ans** It detects change only when the configuration files that we `apply` with kubectl have some change from the configuration when the last `apply` ran successfully.  

## How to refer to my k8s cluster when I run kubernetes cli tools like `kubectl`?

**Ans** To do this:
1. First install `kubectl` using either `brew` or from `curl`.
2. Move kubeconfig yaml file to your `$HOME/.kube/config` folder. Or you can export `KUBECONFIG` env variable with the path of your kubeconfig yaml file.
