# Docker & Kubernetes Project

## Content ##
1. Flask app with Docker (Python)
2. Deploy Flask app on K8S locally (Minikube)
3. Deploy Flask app on Google Kubernetes Engine (GKE)


## Project 1 - Flask app with Docker ## 
1. Installation Python, Flask & Docker Desktop
2. Create Flask project directory
```bash
mkdir flask-app
cd flask-app
```
3. Create Python virtual environment
```bash
python3 -m venv venv
source venv\Scripts\activate
```
<img src="https://github.com/user-attachments/assets/5e69cab8-5cc9-4a57-9aa0-9c9e97e8b2ed" width="200" />

4. Create Flask application
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Docker!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
5. Create Dockerfile
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . /app
RUN pip install flask
EXPOSE 5000
ENV FLASK_APP=app.py
CMD ["python", "app.py"]
```
6. Build Docker Image
```bash
docker build -t flask-docker-app .
```
7. Run Docker container
```bash
docker run -p 5000:5000 flask-docker-app
```
8. Testing on "localhost:5000" & you should see this

<img src="https://github.com/user-attachments/assets/82456aba-42a3-4934-90c3-de1d57df029c" width="300" />

## Project 2 - Flask app on Minikube locally ##
1. Start minikube - "minikube start"
2. Create deployment.yaml & service.yaml
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```
4. Load the Docker image into Minikube "minikube image load flask-app"
5. Expose deployment
```bash
kubectl expose deployment flask-kube-demo --type=LoadBalancer --port=5000"
```
6. Run Minikube service URL 
```bash
minikube service flask-app --url
```

## Project 3 - Flask app on GKE ##
1. Enable Google Artifact Registry either from API Library or command-line (Cloud Shell)
2. Create docker repository in Artifact Registry
```bash
gcloud artifacts repositories create flask-app \
    --repository-format=docker \
    --location=us-central1 \
    --description="Hello Docker Container"
```
3. Push Docker Image
    - Configure Docker to authenticate request to Artifact Registry. "gcloud auth configure-docker us-central1-docker.pkg.dev"
    - Tag Docker local Image. "docker tag flask-kube-demo us-central1-docker.pkg.dev/docker-k8s-431619/flask-app/flask-app"
    - Push Image to Artifact Registry. "docker push us-central1-docker.pkg.dev/docker-k8s-431619/flask-app/flask-app"

4. Set up Google Artifact Registry
    - Create GKE cluster
   ```bash
   gcloud container clusters create flask-app \
   --num-nodes=3 \
   --region=us-central1
   ```
    - Get authentication credentials for the cluster
   ```bash
   gcloud container clusters get-credentials flask-app --region=us-central1
   ```
5. Deploy the Application to GKE
   - Create GKE Cluster
   ```bash
   gcloud container clusters create flask-app-cluster \
    --num-nodes=3 \
    --region=us-central1
   ```
   - Get authentication credentials for the cluster
   ```bash
   gcloud container clusters get-credentials flask-app-cluster --region=us-central1
   ```
   - Update deployment.yaml
   ```bash
     containers:
      - name: flask-app
        image: us-central1-docker.pkg.dev/docker-k8s-431619/flask-app/flask-app
        ports:
        - containerPort: 5000
   ```
   - Apply the deployment to GKE
   ```bash
   kubectl apply -f deployment.yaml
   ```
   - Expose the deployment
   ```bash
   kubectl expose deployment flask-app --type=LoadBalancer --port=5000
   ```
   - Get external IP of service
   ```bash
   kubectl get service flask-app
   ```
