# Infisical

Sync secrets from [Infisical](https://www.infisical.com/) to your Kubernetes cluster using [External Secrets Operator](https://external-secrets.io/).

### Install from chart repository

````bash
alias helm='microk8s helm'

helm repo add external-secrets https://charts.external-secrets.io

helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace
```


## Authentication

In order for the operator to fetch secrets from Infisical, it needs to first authenticate with Infisical.

To authenticate, you can use Universal Auth from Machine identities.

## Storing Your Machine Identity Secrets

Once you have generated a pair of Client ID and Client Secret, you will need to store these credentials in your cluster as a Kubernetes secret.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: universal-auth-credentials
type: Opaque

stringData:
  clientId: <machine identity client id>
  clientSecret: <machine identity client secret>
```

## Secret Store

You will then need to create a generic `SecretStore`.

```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: infisical
spec:
  provider:
    infisical:
      auth:
        universalAuthCredentials:
          clientId:
            key: clientId
            namespace: default
            name: universal-auth-credentials
          clientSecret:
            key: clientSecret
            namespace: default
            name: universal-auth-credentials
      # Details to pull secrets from
      secretsScope:
        projectSlug: first-project-fujo
        environmentSlug: dev # "dev", "staging", "prod", etc..
        # optional
        secretsPath: / # Root is "/"
        # optional
        recursive: true # Default is false
      # optional
      hostAPI: https://app.infisical.com
```

> [!NOTE]
> For ClusterSecretStore, be sure to set namespace in universalAuthCredentials.clientId and universalAuthCredentials.clientSecret.

## Fetch Individual Secret(s)

To sync one or more secrets individually, use the following YAML:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: infisical-managed-secrets
spec:
  secretStoreRef:
    kind: SecretStore
    name: infisical

  target:
    name: auth-api

  data:
    - secretKey: API_KEY
      remoteRef:
        key: API_KEY
```

## Fetch All Secrets

To sync all secrets from an Infisical, use the following YAML:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: infisical-managed-secrets
spec:
  secretStoreRef:
    kind: SecretStore
    name: infisical

  target:
    name: auth-api

  dataFrom:
    - find:
        name:
          regexp: .*
```

## Filter By Prefix/Name

To filter secrets by `path` (path prefix) and `name` (regular expression).

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: infisical-managed-secrets
spec:
  secretStoreRef:
    kind: SecretStore
    name: infisical

  target:
    name: auth-api

  dataFrom:
    - find:
        path: DB_
```

Source:
- https://external-secrets.io/latest/provider/infisical/
