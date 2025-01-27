# Configuración de Ambientes Multi-Entorno con Minikube y ArgoCD

Este documento describe los pasos para configurar un entorno local multi-entorno (`dev` y `prod`) utilizando Minikube y ArgoCD. Incluye la instalación de ArgoCD CLI (`argocd.exe`) y la configuración de los servicios Kubernetes.

---

## **1. Instalación de Minikube y Kubectl**

1. **Instalar Minikube:**
   - Sigue la [guía oficial](https://minikube.sigs.k8s.io/docs/start/) para instalar Minikube en tu sistema.
   - Inicia Minikube con los siguientes parámetros:
     ```bash
     minikube start --driver=virtualbox --cpus=4 --memory=4096
     ```

2. **Instalar kubectl:**
   - Sigue la [guía oficial de Kubernetes](https://kubernetes.io/docs/tasks/tools/) para instalar `kubectl`.

3. **Verifica que Minikube esté corriendo:**
   ```bash
   kubectl get nodes
   ```

4. (Opcional) Exponer el dashboard de Minikube:
   ```bash
   minikube dashboard
   ```

---

## **2. Instalación de ArgoCD en Kubernetes**

1. **Crear el espacio de nombres para ArgoCD:**
   ```bash
   kubectl create namespace argocd
   ```

2. **Instalar ArgoCD en el clúster:**
   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Exponer el servicio de ArgoCD:**
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

4. **Obtener las credenciales iniciales de acceso:**
   - Usuario: `admin`.
   - Contraseña:
     ```bash
     kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
     ```

---

## **3. Instalación de la CLI de ArgoCD (`argocd.exe`)**

1. **Descargar la CLI:**
   ```bash
   curl -LO https://github.com/argoproj/argo-cd/releases/latest/download/argocd-windows-amd64.exe
   ```

2. **Renombrar el ejecutable:**
   ```bash
   rename argocd-windows-amd64.exe argocd.exe
   ```

3. **Mover el ejecutable a una ruta del sistema (Windows):**
   ```bash
   move argocd.exe C:\Windows\System32
   ```

4. **Verificar la instalación:**
   ```bash
   argocd version
   ```

---

## **4. Configuración de los Espacios de Nombres**

1. **Crear los espacios de nombres `dev` y `prod`:**
   ```bash
   kubectl create namespace dev
   kubectl create namespace prod
   ```

---

## **5. Configuración del Repositorio y Aplicaciones en ArgoCD**

1. **Agregar el repositorio Git:**
   ```bash
   argocd repo add https://github.com/tu-usuario/multi-env-deployment.git
   ```

2. **Crear las aplicaciones:**
   - Para `dev`:
     ```bash
     argocd app create nginx-dev \
       --repo https://github.com/tu-usuario/multi-env-deployment.git \
       --path dev \
       --dest-server https://kubernetes.default.svc \
       --dest-namespace dev
     ```

   - Para `prod`:
     ```bash
     argocd app create nginx-prod \
       --repo https://github.com/tu-usuario/multi-env-deployment.git \
       --path prod \
       --dest-server https://kubernetes.default.svc \
       --dest-namespace prod
     ```

3. **Sincronizar las aplicaciones:**
   - Para `dev`:
     ```bash
     argocd app sync nginx-dev
     ```
   - Para `prod`:
     ```bash
     argocd app sync nginx-prod
     ```

---

## **6. Validación de los Servicios y Pods**

1. **Ver los espacios de nombres creados:**
   ```bash
   kubectl get namespaces
   ```

2. **Listar recursos en `dev`:**
   ```bash
   kubectl get all -n dev
   ```

3. **Listar recursos en `prod`:**
   ```bash
   kubectl get all -n prod
   ```

4. **Ver eventos recientes en `dev`:**
   ```bash
   kubectl get events -n dev
   ```

5. **Ver servicios expuestos en `dev`:**
   ```bash
   kubectl get svc -n dev
   ```

---

## **7. Pruebas de Escalado y Recuperación**

1. **Eliminar un pod y observar su recuperación:**
   ```bash
   kubectl delete pod <pod-name> -n dev
   kubectl get pods -n dev -w
   ```

2. **Escalar el número de réplicas en `dev`:**
   ```bash
   kubectl scale deployment nginx-app --replicas=5 -n dev
   kubectl get pods -n dev
   ```

3. **Reiniciar todos los pods en `dev`:**
   ```bash
   kubectl delete pods --all -n dev
   ```

---

## **8. Notas Finales**
- Asegúrate de que los manifiestos en los directorios `dev` y `prod` tengan configuraciones específicas por entorno.
- Usa los `ConfigMap` y `Secrets` para manejar configuraciones dinámicas y credenciales.

---

¡Listo! Ahora tienes un ambiente multi-entorno funcional configurado con Minikube y ArgoCD. Si necesitas agregar más detalles o resolver problemas, ¡no dudes en preguntar! 😊



## **8. Actualizar certificados**

 ```bash
kubectl delete deployment nginx-app -n dev
kubectl delete svc nginx-service -n dev

kubectl apply -f dev/configmap.yaml
kubectl apply -f dev/deployment.yaml
kubectl apply -f dev/service.yaml


minikube service nginx-service -n dev


kubectl delete deployment nginx-app -n prod
kubectl delete svc nginx-service -n prod

kubectl apply -f prod/configmap.yaml
kubectl apply -f prod/deployment.yaml
kubectl apply -f prod/service.yaml
kubectl get pods -n prod

minikube service nginx-service -n prod

  ```

---
# Logs en tiempo real

```bash

kubectl logs -f deployment/nginx-app-stable -n prod --tail=10
kubectl logs -f deployment/nginx-app-canary -n prod --tail=10
  ```

---
