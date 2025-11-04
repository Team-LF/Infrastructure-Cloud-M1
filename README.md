#  TP — Déployer une application web 100 % Serverless sur Azure

Ce projet fait partie du TP **"Déployer une application web 100 % serverless"**.  
L’objectif est de créer une **API Flask**, la **conteneuriser avec Docker**, la **publier sur Azure Container Registry (ACR)**, puis la **déployer sur Azure Container Apps**.

---

##  Étape 1 — Créer une API Flask minimale

###  Créer le projet

```bash
mkdir tp-flask-api
cd tp-flask-api
python3 -m venv venv
source venv/bin/activate
pip install flask
````

###  Fichier `app.py`

```python
import os
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/hello", methods=["GET"])
def hello():
    return jsonify({"message": "Bonjour depuis Flask sur Azure Container Apps !"})

if __name__ == "__main__":
    port = int(os.environ.get("PORT", 8080))
    app.run(host="0.0.0.0", port=port)
```

###  Fichier `requirements.txt`

```
flask==3.0.3
```

### Tester localement

```bash
python app.py
```

 Accéder à [http://localhost:8080/hello](http://localhost:8080/hello)

---

## Étape 2 — Créer une image Docker

###  Fichier `Dockerfile`

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8080
CMD ["python", "app.py"]
```

###  Construire et tester l’image

```bash
docker build -t tp-flask-api .
docker run -p 8080:8080 tp-flask-api
```

 [http://localhost:8080/hello](http://localhost:8080/hello)

---

##  Étape 3 — Publier sur Azure Container Registry (ACR)

```bash
az login
az group create --name tp-serverless-rg --location francecentral

az acr create \
  --resource-group tp-serverless-rg \
  --name tpregistrydemo \
  --sku Basic

az acr login --name tpregistrydemo
docker tag tp-flask-api tpregistrydemo.azurecr.io/tp-flask-api:latest
docker push tpregistrydemo.azurecr.io/tp-flask-api:latest
```

---

##  Étape 4 — Déployer sur Azure Container Apps

```bash
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights

az containerapp env create \
  --name tp-env \
  --resource-group tp-serverless-rg \
  --location francecentral

az containerapp create \
  --name tp-flask-api \
  --resource-group tp-serverless-rg \
  --environment tp-env \
  --image tpregistrydemo.azurecr.io/tp-flask-api:latest \
  --target-port 8080 \
  --ingress external \
  --registry-login-server tpregistrydemo.azurecr.io \
  --registry-username $(az acr credential show -n tpregistrydemo --query "username" -o tsv) \
  --registry-password $(az acr credential show -n tpregistrydemo --query "passwords[0].value" -o tsv)
```

---

### Récupérer l’URL publique

```bash
az containerapp show \
  --name tp-flask-api \
  --resource-group tp-serverless-rg \
  --query properties.configuration.ingress.fqdn \
  -o tsv
```

Exemple :

```
tp-flask-api.francecentral.azurecontainerapps.io
```

### Tester l’API

```bash
curl https://tp-flask-api.francecentral.azurecontainerapps.io/hello
```

Résultat :

```json
{"message": "Bonjour depuis Flask sur Azure Container Apps !"}
```

*Nolan LE ROYER*
