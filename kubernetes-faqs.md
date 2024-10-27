## Always put version tag on each image that you push to a container registry

**Reason** This is because if you want to revert the changes, you would know which commit was responsible for the bug. But if you tag each image with the git version tag, we will know exactly which code is buggy.

## What to do on existing CI/CD pipeline?

**Ans** If the pipeline is already set then run the test cases and if everything runs fine then update the deployment.yaml file with the new git tag as container image tag and push to ecr and then use
```
kubectl apply -f deployment.yaml
```
to redeploy deployment object of your app. This will force kubernetes to fetch your specified image from repository.
