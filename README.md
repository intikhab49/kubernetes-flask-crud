<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:172554,50:1D4ED8,100:38BDF8&height=170&section=header&text=Kubernetes%20Flask%20CRUD&fontSize=48&fontColor=ffffff&fontAlignY=38&desc=Flask%20%C2%B7%20PostgreSQL%20%C2%B7%20NGINX%20%C2%B7%20Minikube%20%C2%B7%20Docker%20Swarm&descSize=17&descAlignY=60&animation=fadeIn" width="100%" alt="Kubernetes Flask CRUD — Flask, PostgreSQL, NGINX, Minikube, Docker Swarm"/>

# Deploy a Flask + PostgreSQL CRUD App on Kubernetes with an NGINX Reverse Proxy

**A three-tier web app — NGINX → Flask → PostgreSQL — packaged as containers and deployed to Minikube with NetworkPolicies, or to Docker Swarm as a stack.**

<p>
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes"/>
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker"/>
  <img src="https://img.shields.io/badge/NGINX-009639?style=for-the-badge&logo=nginx&logoColor=white" alt="NGINX"/>
  <img src="https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white" alt="Flask"/>
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL"/>
  <img src="https://img.shields.io/badge/Gunicorn-499848?style=for-the-badge&logo=gunicorn&logoColor=white" alt="Gunicorn"/>
</p>

</div>

---

The middle step of my container-orchestration learning path — it takes the [Flask/PostgreSQL CRUD dashboard](https://github.com/intikhab49/flask-postgres-crud-dashboard) and runs it **behind NGINX on Kubernetes**. For the extended version with a logging microservice and Prometheus monitoring, see **[kubernetes-flask-microservices](https://github.com/intikhab49/kubernetes-flask-microservices)**.

## 🏗️ Architecture

```mermaid
flowchart LR
    U(["🌐 Client"]) -->|"NodePort 30242"| N["NGINX<br/>reverse proxy"]
    N --> W["web<br/>Flask + Gunicorn :5000"]
    W -->|"NetworkPolicy:<br/>web → postgres only"| P[("PostgreSQL 14<br/>persistent storage")]
```

**What it demonstrates**

- 🐳 Custom Docker images for the Flask app and NGINX
- ☸️ Kubernetes Deployments + Services, exposed through a `NodePort`
- 🔒 `NetworkPolicy` rules: only NGINX can reach web, only web can reach Postgres
- ⏳ Startup script that waits for PostgreSQL readiness before initializing the schema and starting Gunicorn
- 🐝 The same stack deployed to Docker Swarm with an automated redeploy script
- 🔄 CRUD REST API + Bootstrap web UI with a health check

## ☸️ Deploy on Kubernetes (Minikube)

```bash
git clone https://github.com/intikhab49/kubernetes-flask-crud.git
cd kubernetes-flask-crud

minikube start
docker build -t myapp-web:latest .
docker build -t nginx-custom:latest ./nginx
minikube image load myapp-web:latest
minikube image load nginx-custom:latest

kubectl apply -f postgres-deployment.yaml
kubectl apply -f web-deployment.yaml
kubectl apply -f nginx-deployment.yaml
kubectl apply -f web-to-postgres-policy.yaml -f network-policy.yaml

minikube service nginx --url
```

## 🐝 Deploy on Docker Swarm

```bash
docker swarm init
./deploy.sh          # redeploys docker-compose.yml as the "my-app" stack
curl http://localhost:9000/users
```

The compose file pulls the web and NGINX images from a local registry at `localhost:5000` and publishes NGINX on port **9000**.

## 📡 REST API

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | Web UI |
| GET | `/users` | List users |
| POST | `/users` | Create a user `{"name", "email"}` |
| PUT | `/users/{id}` | Update a user |
| DELETE | `/users/{id}` | Delete a user |
| GET | `/health` | Health check |

## 🗂️ Project structure

```
app.py · models.py · templates/ · static/          # Flask app + UI
Dockerfile · start.sh                              # web image + DB-wait startup
nginx/                                             # NGINX image and config
postgres-deployment.yaml · web-deployment.yaml · nginx-deployment.yaml
web-to-postgres-policy.yaml · network-policy.yaml  # NetworkPolicies
docker-compose.yml · deploy.sh                     # Docker Swarm stack
```

---

<div align="center">

**Built by [Intikhab Azam](https://github.com/intikhab49)** — AI & automation engineer · backend · DevOps

<sub>Keywords: Kubernetes Flask tutorial · deploy Flask on Kubernetes · Minikube · NGINX reverse proxy · PostgreSQL · Kubernetes NetworkPolicy · Docker Swarm stack · Gunicorn · DevOps project</sub>

</div>
