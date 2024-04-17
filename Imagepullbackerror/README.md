# Kubernetes Troubleshooting

## Image pullbackoff
Initial error := ErrorImagePull -> k8s will wait for some time (Backoff delay-> incremental error) -> 
Image pullbackoff

### Scenario 1: 

- Due to invalid image name
- If the image is deleted and you are trying to deploy a deleted image.
You have created a deployment and deployed your application which is pointing to that non-existing
container image. It will result in image pullbackoff error.

### Scenario 2:

- The container image which you are trying to deploy is private (secure).
Not all images in Docker hub are public. There are private images too. Nobody can download the image 
without his/her authorization.
You can use add `imagePullSecrete: <DockerCredetials>` to pull the private images.
If you want to pull a image from Elastic Kubernetes Service. You can add the same above line, but using 
your secrete name as AWS credentials.


#### First you need to identify image pullbackoff error occured thrugh which of the scenarios.

You can do that by using -
- `describe` command
- `event` command

you can use `kubectl get pods` to check whether the image is throwing an error or not.

#### How to recreate the issue.
1. Just change the name of the image in you yaml file so that the name is invalid.
2. Use `kubectl apply -f <file_name.yml>`
3. Use `kubectl get pods -w` to monitor in live mode.

![alt text]()

#### How to pull a private repository.
In order to pull a private repo in docker hub, you need to provide your docker username, password and 
email in the terminal.

`kubectl create secret docker-registry demo --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>`

where:

 ``<your-registry-server>`` is your Private Docker Registry FQDN. Use https://index.docker.io/v1/ for DockerHub.
 
 ``<your-name>`` is your Docker username.
    
 ``<your-pword>`` is your Docker password.
    
 ``<your-email>`` is your Docker email.
    

Execute the above command in the terminal.

In your deployment.yml file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: <dockerUsername>/<image>:<tagname>
        ports:
        - containerPort: 80
      imagePullSecrets:
        - name: demo
```

