# Reto Práctico #3 – Despliegue GitOps de Microservicio con ArgoCD y GitHub Actions

Repositorio: https://github.com/rsanmartin/product-service-argocd.git

## 1. Descripción del proyecto
Este proyecto implementa un flujo **GitOps** para desplegar un microservicio **Node.js** (product-service) en Kubernetes usando **ArgoCD** para sincronización automática y **GitHub Actions** como pipeline CI/CD.  
El servicio expone un endpoint de salud en **/api/health** y se conecta a una base de datos **PostgreSQL** mediante variables de entorno gestionadas con **ConfigMap** y **Secret**.

## 2. Arquitectura implementada (GitOps)
Componentes principales:
- **GitHub Repo**: contiene código + manifiestos Kubernetes (`k8s/`) + configuración ArgoCD.
- **GitHub Actions (CI/CD)**:
  - Ejecuta pruebas
  - Construye y publica imagen Docker en Docker Hub
  - Actualiza el tag de la imagen en `k8s/deployment.yaml`
  - Hace commit/push al repositorio (GitOps puro)
- **Docker Hub**: almacena la imagen del microservicio.
- **ArgoCD**: detecta cambios en el repositorio y sincroniza automáticamente el estado del clúster.
- **Kubernetes (Minikube)**: ejecuta el microservicio como Deployment + Service + ConfigMap + Secret.

Flujo:
1) Commit en `main` → 2) GitHub Actions build/push → 3) GitHub Actions actualiza manifiestos y hace commit →  
4) ArgoCD detecta el cambio en Git → 5) ArgoCD sincroniza → 6) App queda Synced/Healthy.

## 3. Manifiestos Kubernetes
Los manifiestos están en el directorio `k8s/`:
- `deployment.yaml`: Deployment con 2 réplicas, RollingUpdate, requests/limits, liveness/readiness probes a `/api/health`, variables desde ConfigMap/Secret.
- `service.yaml`: Service tipo ClusterIP con puerto 80 redireccionando al 3000 del contenedor.
- `configmap.yaml`: variables no sensibles (DB_NAME, DB_HOST, DB_PORT).
- `secret.yaml`: credenciales sensibles (DB_USER, DB_PASSWORD en base64).

## 4. Flujo CI/CD (GitHub Actions)
Workflow: `.github/workflows/deploy.yml`

Acciones realizadas:
1. Instala dependencias
2. Ejecuta pruebas
3. Login a Docker Hub
4. Build & Push de imagen con tag (SHA corto) y `latest`
5. Actualiza el `image:` en `k8s/deployment.yaml` con el tag nuevo
6. Commit y push del manifiesto actualizado al repo

## 5. Configuración ArgoCD
Archivo: `argocd-application.yaml`
- Apunta al directorio `k8s/` del repo
- Sincronización automática habilitada
- Auto-prune y self-heal habilitados
- Despliegue al namespace: `product`

## 6. Instrucciones de uso y verificación (Paso 7)
### 6.1 Verificar estado de recursos
```bash
kubectl get pods -n product
kubectl get svc -n product