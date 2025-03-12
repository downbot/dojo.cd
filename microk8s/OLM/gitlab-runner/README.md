The GitLab Runner operator manages the lifecycle of GitLab Runner in Kubernetes or Openshift clusters. The operator aims to automate the tasks needed to run your CI/CD jobs in your container orchestration platform.

## Prerequisites

  For Kubernetes cluster, install cert-manager:

  ```shell
  kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.7.1/cert-manager.yaml
  ```

## GitLab Runner version

This version of **GitLab Runner Operator** ships with **GitLab Runner v17.9.0**.

To use a different version of **GitLab Runner** change the [`runnerImage` and `helperImage` properties](https://docs.gitlab.com/runner/configuration/configuring_runner_operator.html#operator-properties).

## Usage

 To link a GitLab Runner instance to a self-hosted GitLab instance or the hosted [GitLab](https://gitlab.com), you first need to:

 1. Create a secret containing the `runner-token` from your GitLab project.

  ```
  cat > gitlab-runner-secret.yml << EOF
  apiVersion: v1
  kind: Secret
  metadata:
    name: gitlab-runner-secret
  type: Opaque
  # Only one of the following fields must be set. The Operator will fail to register Runner if both are provided.
  # # NOTE: runner-registration-token is deprecated and will be removed in GitLab 18.0. You should use runner-token instead.
  stringData:
    runner-token: REPLACE_ME # your project runner token
    # runner-registration-token: "" # your project runner secret
  EOF
  ```

  ```
  oc apply -f gitlab-runner-secret.yml
  ```

 2. Create the Custom Resource Definition (CRD) file and include the following information. The tags value must be openshift for the job to run.

   ```
   cat > gitlab-runner.yml << EOF
   apiVersion: apps.gitlab.com/v1beta2
   kind: Runner
   metadata:
     name: gitlab-runner
   spec:
     gitlabUrl: https://gitlab.example.com
     buildImage: alpine
     token: gitlab-runner-secret
     tags: openshift
   EOF
   ```

  ```
  oc apply -f gitlab-runner.yml
  ```

## Full documentation

- [Install GitLab Runner Operator](https://docs.gitlab.com/runner/install/operator.html)
- [Configurating Runner Operator](https://docs.gitlab.com/runner/configuration/configuring_runner_operator/)

