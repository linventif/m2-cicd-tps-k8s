# TP2 : configuration et isolation

[Sujet du TP](TP2%20-%20Kubernetes.pdf).

## 1. Namespaces et configuration de l'API

`namespaces.yaml` définit les namespaces `dev` et `test`. `whoami.yaml` définit la ConfigMap `api-config`, un Deployment `api` de deux réplicas utilisant `traefik/whoami:v1.10.1`, et son Service sur le port 80. Les variables de la ConfigMap sont injectées avec `envFrom`.

Commandes à lancer depuis ce dossier, avec `kubectl` configuré pour un cluster accessible :

```bash
kubectl apply -f namespaces.yaml
kubectl -n dev apply -f whoami.yaml
kubectl -n dev rollout status deployment/api
kubectl -n dev get configmap,pods,services
kubectl -n dev exec deployment/api -- printenv WHOAMI_NAME APP_LOG_LEVEL
```

Les valeurs configurées sont `api` et `debug`. Les ressources de l'application sont créées dans `dev` ; le namespace `test` est disponible pour la suite du TP.

## 2. Page HTML dans une ConfigMap

`index.html` contient la page « Hello World ». `site.yaml` monte la ConfigMap `site` dans `/usr/share/nginx/html` d'un conteneur `nginx:1.27`. La ConfigMap doit être créée dans le même namespace avant le déploiement :

```bash
kubectl -n dev create configmap site --from-file=index.html \
  --dry-run=client -o yaml | kubectl -n dev apply -f -
kubectl -n dev apply -f site.yaml
kubectl -n dev rollout status deployment/site
kubectl -n dev port-forward deployment/site 8080:80
```

Dans un autre terminal :

```bash
curl http://127.0.0.1:8080
```

Le contenu attendu est celui de `index.html`.

## Validation réalisée

Les six manifests des TP1 et TP2 ont été analysés avec PyYAML : neuf documents YAML valides. Les sélecteurs des Deployments et Services correspondent aux labels des Pods, et les images utilisent des tags explicites.

Aucun cluster ni client `kubectl` n'était disponible pour cette vérification. Les commandes de déploiement ci-dessus restent à exécuter ; le démarrage des Pods et les réponses HTTP sur cluster ne sont pas encore validés.

## Suite du TP

Les exercices sur les Secrets et le DNS restent à réaliser. Le TP2 n'est pas encore terminé.
