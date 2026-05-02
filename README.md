# Cubix Done Kubernetes Homework (Adam Kovacs)

## Deployment command

```powershell
kubectl apply -f .\optional.yaml
```

This command creates all required resources from one YAML file: Namespace, Deployment, Service and Ingress.

## Deployed application

Namespace: optional3

Image:

```text
quay.io/drsylent/cubix/block3/optional:quarkus3
```

Environment variable:

```text
FILE_LOCATION=/deployments/optional.txt
```

Resource limits:

```text
cpu: 500m
memory: 256Mi
```

## Header endpoint responses

### 1. Through Ingress

Command:

```powershell
curl.exe http://localhost:8080/header -H "custom-header: ingress-check" -s
```

Response:

```json
{"Accept":["*/*"],"Host":["localhost:8080"],"User-Agent":["curl/8.18.0"],"X-Forwarded-For":["172.18.0.1"],"X-Forwarded-Host":["localhost:8080"],"X-Forwarded-Port":["80"],"X-Forwarded-Proto":["http"],"X-Forwarded-Scheme":["http"],"X-Real-IP":["172.18.0.1"],"X-Request-ID":["4e78dbf54b9e77375495f9541b22e3d5"],"X-Scheme":["http"],"custom-header":["ingress-check"]}
```

### 2. Through Kubernetes Service

Command:

```powershell
Start-Job -Name optional-svc-pf -ScriptBlock { kubectl port-forward -n optional3 service/optional 9090:80 } | Out-Null
Start-Sleep -Seconds 2
curl.exe http://localhost:9090/header -H "custom-header: service-check" -s
Stop-Job optional-svc-pf; Remove-Job optional-svc-pf
```

Response:

```json
{"Accept":["*/*"],"Host":["localhost:9090"],"User-Agent":["curl/8.18.0"],"custom-header":["service-check"]}
```

### 3. Through Pod directly

Command:

```powershell
$pod=kubectl get pod -n optional3 -l app=optional -o jsonpath="{.items[0].metadata.name}"
Start-Job -Name optional-pod-pf -ArgumentList $pod -ScriptBlock { param($p) kubectl port-forward -n optional3 pod/$p 9091:8080 } | Out-Null
Start-Sleep -Seconds 2
curl.exe http://localhost:9091/header -H "custom-header: pod-direct-check" -s
Stop-Job optional-pod-pf; Remove-Job optional-pod-pf
```

Response:

```json
{"Accept":["*/*"],"Host":["localhost:9091"],"User-Agent":["curl/8.18.0"],"custom-header":["pod-direct-check"]}
```

### 4. From inside the Pod

Command:

```powershell
kubectl exec -n optional3 deploy/optional -- curl http://localhost:8080/header -H "custom-header: inside-pod-check" -s
```

Response:

```json
{"Accept":["*/*"],"Host":["localhost:8080"],"User-Agent":["curl/7.61.1"],"custom-header":["inside-pod-check"]}
```

## File copy command

Local file creation:

```powershell
Set-Content -Path .\optional.txt -Encoding ASCII -NoNewline -Value "AdamKovacs"
```

Copy command:

```powershell
$pod=kubectl get pod -n optional3 -l app=optional -o jsonpath="{.items[0].metadata.name}"
kubectl cp .\optional.txt optional3/${pod}:/deployments/optional.txt -c optional
```

Check copied file:

```powershell
kubectl exec -n optional3 deploy/optional -- cat /deployments/optional.txt
```

Response:

```text
AdamKovacs
```

## Data endpoint response

Command:

```powershell
curl.exe http://localhost:8080/data -s
```

Response:

```text
AdamKovacs-verymuchsecret-6456401677771
```

## Final validation

Command:

```powershell
kubectl apply --dry-run=server -f .\optional.yaml
```

Result:

```text
namespace/optional3 unchanged (server dry run)
deployment.apps/optional unchanged (server dry run)
service/optional unchanged (server dry run)
ingress.networking.k8s.io/optional unchanged (server dry run)
```
