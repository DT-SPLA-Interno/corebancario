# Demo: Bank of Anthos en AKS (namespace: `corebancario`)

## 1) Prerrequisitos
- Azure CLI (`az`), `kubectl`, `git`.

## 2) Clonar el repositorio
```bash
git clone https://github.com/GoogleCloudPlatform/bank-of-anthos.git
cd bank-of-anthos
```

## 3) Crear namespace y secreto JWT
```bash
kubectl create namespace corebancario
kubectl -n corebancario apply -f ./extras/jwt/jwt-secret.yaml
```

## 4) Desplegar los manifiestos base
```bash
kubectl -n corebancario apply -f ./kubernetes-manifests
```

## 5) Desactivar envío de métricas/trazas a Google Cloud (post-deploy)
```bash
kubectl -n corebancario set env deploy -l application=bank-of-anthos   ENABLE_METRICS=false ENABLE_TRACING=false
```

## 6) Verificar despliegue
```bash
kubectl -n corebancario get pods
kubectl -n corebancario get svc
kubectl -n corebancario get svc frontend -o jsonpath='{.status.loadBalancer.ingress[0].ip}'; echo
# Abre http://<IP> en el navegador
```

## 7) (Opcional) Aumentar tráfico de prueba
```bash
kubectl -n corebancario scale deploy/loadgenerator --replicas=2
```

---

### Instalación de Dynatrace (Operator para AKS)
Sigue la guía oficial: **Install Dynatrace Operator add-on for AKS**  
https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/deployment/marketplaces/aks-dto

## 8) Reiniciar todos los Pods del demo (tras instalar Dynatrace u otros cambios)
```bash
# Reinicia todos los Deployments etiquetados como parte del demo
kubectl -n corebancario rollout restart deploy -l application=bank-of-anthos
# Verifica
kubectl -n corebancario get pods
```

---

## FAQ
**¿Si hago `kubectl delete -f ./kubernetes-manifests` se pierde la configuración de las variables (`ENABLE_METRICS/ENABLE_TRACING=false`)?**  
Sí. Ese comando elimina los Deployments. Al volver a aplicar los manifiestos del repositorio, las variables regresarán a los valores definidos en YAML (por defecto `true`).  
Para mantenerlos en `false`, usa una de estas opciones:
- Re‑ejecuta el paso **5** (`kubectl set env ...`) después de aplicar los manifiestos, **o**
- Usa un overlay de **Kustomize**/Helm donde queden en `false` como código (recomendado para reproducibilidad).
