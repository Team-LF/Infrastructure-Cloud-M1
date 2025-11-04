# Étape 1 : Image Python légère
FROM python:3.11-slim

# Étape 2 : Dossier de travail
WORKDIR /app

# Étape 3 : Copier les dépendances
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Étape 4 : Copier le reste du code
COPY . .

# Étape 5 : Exposer le port d’écoute
EXPOSE 8080

# Étape 6 : Commande de démarrage
CMD ["python", "app.py"]
