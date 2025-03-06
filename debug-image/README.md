## Build images


### Build image with `buildah`

```bash
buildah build -t alpineph .
buildah images # to list images
```

#### Push image to Container Registry

Open tunnel from local port 32000 to cluster node port 32000

```bash
buildah push --tls-verify=false localhost/alpineph localhost:32000/alpineph
```

#### Start Ephemeral Container

```bash
kubectl debug -n <namespace> -it pod/<pod-name> --image=localhost:32000/alpineph --profile=general
```
