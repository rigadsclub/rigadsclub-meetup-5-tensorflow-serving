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