# Prerequisit
kubectl create namespace sealed-secrets
export NAMESPACE="sealed-secrets"
export PRIVATEKEY=mytls.key
export PUBLICKEY=mytls.crt
export SECRETNAME="sealed-secrets-key"
kubectl -n "$NAMESPACE" create secret tls "$SECRETNAME" --cert="$PUBLICKEY" --key="$PRIVATEKEY"
kubectl -n "$NAMESPACE" label secret "$SECRETNAME" sealedsecrets.bitnami.com/sealed-secrets-key=active

# Deployment
flux bootstrap github --owner=leonkappes --repository=Kubernetes-Flux --personal --path bootstrap