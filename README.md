## Kubernetes: NGINX + Ingress (Orbstack) — Guia de Estudo

Este repositório contém manifests simples para estudar objetos básicos do Kubernetes em um cluster local (ex.: Orbstack no macOS):

- Deployment com `nginx` (3 réplicas)
- Service `ClusterIP` expondo o `nginx`
- Ingress (classe `nginx`) roteando `/` para o Service
- Um Pod avulso de Node.js para praticar logs e inspeção

### Arquivos

- `deployment-orbstack.yaml`: cria um `Deployment` chamado `nginx` com 3 réplicas do container `nginx:latest` escutando na porta 80.
- `service-clusterip.yaml`: cria um `Service` do tipo `ClusterIP` chamado `nginx-clusterip`, direcionando tráfego para os Pods que possuem o label `app: nginx` na porta 80.
- `ingress-nginx.yaml`: cria um `Ingress` (classe `nginx`) roteando o caminho `/` para o `Service` `nginx-clusterip` na porta 80.
- `pod-nodejs-orbstack.yaml`: cria um `Pod` standalone (`nodejs-pod-dec`) com `node:18-alpine` que imprime "Rodando..." a cada 5s (útil para testar `kubectl logs`).

### Pré‑requisitos

- `kubectl` instalado e configurado para apontar para seu cluster local
  - Verifique: `kubectl config current-context`
  - Em Orbstack, o contexto geralmente aparece como `orbstack`
- Um Ingress Controller ativo com `ingressClassName: nginx` (ex.: ingress-nginx)
  - Se não estiver disponível, você pode instalar rapidamente com Helm (opcional):

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

Observação (Orbstack): Em muitos setups locais, o Ingress é exposto em `localhost:80/443`. Caso contrário, use o endereço mostrado em `kubectl get ingress`.

### Como aplicar os recursos

Na raiz deste diretório, execute:

```bash
kubectl apply -f deployment-orbstack.yaml
kubectl apply -f service-clusterip.yaml
kubectl apply -f ingress-nginx.yaml
# Opcional (Pod Node.js para treinar logs):
kubectl apply -f pod-nodejs-orbstack.yaml
```

### Verificando o estado

```bash
kubectl get deploy,po,svc,ingress -o wide
kubectl describe deploy/nginx | sed -n '1,80p'
kubectl describe svc/nginx-clusterip | sed -n '1,80p'
kubectl describe ingress/ingress-nginx | sed -n '1,120p'
```

Logs do Pod Node.js (opcional):

```bash
kubectl logs -f pod/nodejs-pod-dec
```

### Testando o acesso HTTP

- Via Ingress:
  - Se o Ingress estiver em `localhost`: `curl -i http://localhost/`
  - Caso contrário, descubra o ADDRESS: `kubectl get ingress ingress-nginx` e teste com `curl -i http://<ADDRESS>/`

- Via port-forward (útil para testar mesmo sem Ingress):

```bash
kubectl port-forward svc/nginx-clusterip 8080:80
# Em outro terminal:
curl -i http://localhost:8080/
```

### Escalando e atualizando

```bash
# Escalar réplicas (Deployment já inicia com 3):
kubectl scale deploy/nginx --replicas=5

# Atualizar imagem (exemplo):
kubectl set image deploy/nginx nginx=nginx:1.27-alpine

# Ver rollout:
kubectl rollout status deploy/nginx
kubectl rollout history deploy/nginx
```

### Dicas de troubleshooting

- Verifique Pods e eventos:
  - `kubectl get pods -A`
  - `kubectl describe pod <nome-do-pod>`
  - `kubectl get events -A --sort-by=.metadata.creationTimestamp`
- Confirme se o Ingress Controller está em execução (ex.: namespace `ingress-nginx`):
  - `kubectl get pods -n ingress-nginx`
- Cheque se o `Service` seleciona os Pods certos: labels devem bater (`app: nginx`).
- Em ambientes locais, portas 80/443 podem estar ocupadas por outro processo. Libere-as ou use `port-forward`.

### Limpeza

```bash
kubectl delete -f ingress-nginx.yaml || true
kubectl delete -f service-clusterip.yaml || true
kubectl delete -f deployment-orbstack.yaml || true
kubectl delete -f pod-nodejs-orbstack.yaml || true
```

### Referências úteis

- Documentação Kubernetes — Ingress: https://kubernetes.io/docs/concepts/services-networking/ingress/
- ingress-nginx — Guia de instalação: https://kubernetes.github.io/ingress-nginx/deploy/
- Orbstack — Kubernetes: https://orbstack.dev/docs/kubernetes


