Sealed Secrets
==============

[Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) is an open-source project started at Bitnami and is used to encrypt Kubernetes secrets. Once encrypted, you can store this encrypted secret in your Git repository safely. It allows DevOps practices without exposing sensitive data.

Only the Sealed Secrets controller can decrypt these encrypted secrets.


```bash
kubectl apply -f controller.yaml
```

Client examples
---------------

```bash
kubeseal --cert certificate.pem -f secret.yaml -o yaml -w sealed-secret.yaml
```

```bash
echo -n 'password' | kubeseal --cert certificate.pem --raw -n namespace-name --name secret-name
```


---

Private key in https://keepersecurity.com/vault/#detail/CdHp88EjnNk3eEo0aCWnIA

