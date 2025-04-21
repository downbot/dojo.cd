cert-manager
============

To enable **cert-manager** in MicroK8s, you can follow these steps:

1. **Enable the cert-manager addon**:
   Run the following command to enable cert-manager:
   ```bash
   microk8s enable cert-manager
   ```

2. **Verify the addon is enabled**:
   Check the status of MicroK8s to ensure cert-manager is listed under the enabled addons:
   ```bash
   microk8s status
   ```

3. **Set up a ClusterIssuer**:
   After enabling cert-manager, you need to configure a `ClusterIssuer` to manage certificates. Here's an example configuration for Let's Encrypt:
   ```yaml
   apiVersion: cert-manager.io/v1
   kind: ClusterIssuer
   metadata:
     name: lets-encrypt
   spec:
     acme:
       email: your-email@example.com
       server: https://acme-v02.api.letsencrypt.org/directory
       privateKeySecretRef:
         name: lets-encrypt-private-key
       solvers:
       - http01:
           ingress:
             class: public
   ```
   Apply this configuration using:
   ```bash
   microk8s kubectl apply -f clusterissuer.yaml
   ```

4. **Deploy a service and configure Ingress**:
   Deploy your application and set up an Ingress resource with annotations for cert-manager to automatically issue TLS certificates.

Let me know if you'd like help with any specific part of this process!

