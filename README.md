# Part 1: Preparing docker image

## 1. Run TensorFlow serving docker image
```
docker run -d --name serving_base tensorflow/serving
```
## 2. Create another version of docker image containing your model

```
docker cp saved_model_riga serving_base:/models/rre
docker commit --change "ENV MODEL_NAME rre" serving_base $USER/rre_serving
```

## 3. Kill original container
```
docker kill serving_base
docker rm serving_base
```
## 4. Run model locally
```
docker run -p 8501:8501 -t $USER/rre_serving &
```

## 5. Query the REST API
### Model status
```
curl -X GET http://localhost:8501/v1/models/rre
```

### Model metadata
```
curl -X GET http://localhost:8501/v1/models/rre/metadata
```

### Predictions
```
curl -X POST -H "Content-Type: application/json" -d @./sample_request.json http://localhost:8501/v1/models/rre:predict
```

# Part 2: Deploy image to Kubernetes cluster

```
gcloud auth login --project rigadsclub-demo
```

## 1. Create a cluster:
```
gcloud container clusters create demo-cluster --num-nodes 1 --zone=europe-west1-b
```

## 2. Set the default cluster for gcloud container command and pass cluster credentials to kubectl:
```
gcloud config set container/cluster demo-cluster
gcloud container clusters get-credentials demo-cluster
```

## 3. Tag the image using the Container Registry format and our project name:
```
docker tag $USER/rre_serving eu.gcr.io/rigadsclub-demo/rre
```

## 4. Configure Docker to use gcloud as a credential helper:
```
gcloud auth configure-docker
```

## 5. Push the image to the Registry:
```
docker push eu.gcr.io/rigadsclub-demo/rre
```

## 6. Cluster setup

### Deployment:
```
kubectl apply -f ./manifests/riga-ds-club-api-deployment.yaml
```

### Service:
```
kubectl apply -f ./manifests/riga-ds-club-api-service.yaml
```
### Ingress: 
```
kubectl apply -f ./manifests/riga-ds-club-api-ingress.yaml
```

### Bonus track

## Assigning IP for the API
```
gcloud compute project-info update --default-network-tier STANDARD
gcloud compute addresses create riga-ds-club-api-address --region europe-west1 --network-tier STANDARD
gcloud compute addresses describe riga-ds-club-api-address --region europe-west1

kubectl create ns nginx
helm install --name nginx-ingress stable/nginx-ingress --namespace nginx --set controller.service.loadBalancerIP=35.206.188.185

```