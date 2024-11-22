# Case Study for DevSecOps role

## Instructions
1. Build an application using the programming language of your choice in the `app/` folder which runs a webservice on Port 5000 and responds with "Hello World!" to any requests (e.g. curl http://localhost:5000/).

I have used Python as language and with Flask I created a small API with a single endpoint (/) which returns the requested text: "Hello World!".

2. Build a Dockerfile which packages the app into a container and runs the previously written app on startup.

I've created a Dockerfile that uses a Python image, copies the app files, ensures that Flask is installed and exposes the container on port 5000. 

You can test the image locally on Docker with these 2 commands from the root (where the Dockerfile is located):

```bash
docker build -t case-study .
docker run -p 5000:5000 case-study
```

3. Build a Helmchart in `helmchart/` that runs the docker container in a deployment with a service and ingress in front of it.
I have filled the helmchart folder with the Chart.yaml, values.yaml and templates folder, which contains the manifests templates for Kubernetes (deployment, ingress and services yamls).

In order to test it, I have used Minikube. In order for the Minikube K8s cluster to have access to the case-study image, we need to run these commands which build the image inside Minukube's local repo of images since we don't want it to try to pull the image from the internet:

```bash
eval $(minikube docker-env)
docker build -t case-study .
```

Also, I set the imagePullPolicy to Never in the Helm chart.

Now the helm chart can be installed with this command:

```bash
helm install case-study helmchart
```

4. Orchestrate the helmchart deployment in `helmfile.d/` and add all helm charts needed to serve the application starting from a brand new kind/k3s/minikube cluster.

I've created a helmfile.yaml inside folder helmfile.d  taking care of specifying the correct relative path of the helm chart: ../helmchart

In order to apply the helmchart in the cluster use this command:

```bash
helmfile sync
```
You can visit the resulting app with these 2 methods assuming you use Minikube:

# Method 1:

Run this command and you will get the an IP and port that you can visit:

```bash
minikube service case-study-service
```




# Method 2:

Alternatively, if you want to use a specific url locally, you can create a tunnel with these commands:

```bash
minikube addons enable ingress
minikube tunnel
```
Then go to your hosts file (etc/hosts in Linux or Mac) and add this line:

```bash
127.0.0.1       casestudy.local.helloworld.io
```



## Guidelines

- The ingress should listen to the Domain/Host: casestudy.local.helloworld.io (DNS is statically pointing to 127.0.0.1 so you will need to setup a minikube/k3s/kind environment that listens on localhost).
- The ingress should listen on HTTP only and forward the requests to the setup service.
- You might need to publish your Dockerimage on dockerhub.com or leave instructions on how the k8s cluster can pull the image.
- Your custom helmchart for the "Hello World!" application can be referenced using relative paths (e.g. `chart: ../helmchart/mychart`).
- If you add other helmcharts with ingresses you can use *.local.helloworld.io (e.g. grafana.local.helloworld.io)

## Scoring

Run "helmfile apply ." and then try to access casestudy.local.helloworld.io in a browser. Success requirements are:

- `app/` contains working code
- `helmchart/` contains working helmchart
- `helmfile.d/` contains full orchestration of setup
- Can successfully open casestudy.local.helloworld.io and received "Hello World!" as response
- Elegant code, full instrumentation, test cases, surprising solutions