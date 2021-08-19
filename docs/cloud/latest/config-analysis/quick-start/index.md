import Alert from '@material-ui/lab/Alert';

# Automated Configuration Analysis with the Ambassador DCP

Edge Stack and Emissary-ingress are managed declaratively. This approach lends itself well to a [GitOps workflow](docs/edge-stack/latest/topics/concepts/gitops-continuous-delivery/). Traditionally, adopting a GitOps workflow requires an extensive amount of engineering. With the Ambassador Developer Control Plane, you can quickly and easily adopt a GitOps workflow without any custom engineering.

In this quick start, we'll walk through how you can configure Edge Stack or Emissary-ingress for a GitOps workflow using the DCP. The DCP will automatically detect and resolve configuration issues _before_ your changes go live.

<div class="docs-article-toc">
<h3>Contents</h3>

* [Prerequisites](#prerequisites)
    * [Kubernetes Environment](#kubernetes-environment)
* [1. Connect your cluster to Ambassador Cloud](#1-connect-your-cluster-to-ambassador-cloud)
* [2. Fork the demo repository](#2-fork-the-demo-repository)
* [3. Apply the manifests in your cluster and see the service being reported in the service catalog](#3-apply-the-manifests-in-your-cluster-and-see-the-service-being-reported-in-the-service-catalog)
* [4. Configure GitHub integration](#4-configure-github-integration)
* [5. Create a Pull Request](#5-create-a-pull-request)
* [6. Review & merge Pull Request](#6-review--merge-pull-request)
* [What's Next?](#whats-next)

</div>

## Prerequisites

### Kubernetes Environment

To get started with automated configuration analysis, ensure you have ***Edge Stack or Emissary-ingress version 2.0 or later installed*** in your cluster. You can check your existing version using the commands below (adjust your namespace as necessary):

```
# Find the namespace and name of your Edge Stack of Emissary-ingress deployment
kubectl get deploy --all-namespaces -l product=aes

# Check the image. This should be version 2.0.0-ea or greater
kubectl get deploy --namespace $YOUR_NAMESPACE $YOUR_DEPLOYMENT -o jsonpath='{.spec.template.spec.containers[0].image}'
```

If you do not have the latest version installed, you can:

* [Install the latest version of Edge Stack](/docs/edge-stack/2.0/topics/install/)
* [Upgrade Edge Stack to the latest version](/docs/edge-stack/2.0/topics/install/upgrading/)

  <Alert severity="info">
  Ensure you have Edge Stack or Emissary-ingress 2.0 or later.
  </Alert>

## 1. Connect your cluster to Ambassador Cloud

<Alert severity="info">
  If you followed the <a href="/docs/edge-stack/2.0/tutorials/getting-started/">Edge Stack quick start</a>, you can skip this step.
</Alert>

1. Log in to [Ambassador Cloud](https://app.getambassador.io/cloud/) with your preferred identity provider.

2. At the top, click **Add Services** then click **Connection Instructions** in the `Connect your installation` section.

3. Follow the prompts to name the cluster and click **Generate a Cloud Token**.

4. Follow the prompts to install the cloud token into your cluster.

5. When the token installation completes, refresh the Service Catalog page.

<Alert severity="success"><b>Victory!</b> All the Services running in your cluster are now listed in Service Catalog!
You should now be able to see all running services in your cluster at http://app.getambassador.io/cloud/services </Alert>

## 2. Fork the demo repository

Fork the <a href="https://github.com/datawire/a8r-gitops-example" target="_blank">demo repository</a> and clone your fork into your local environment. This repository contains a Kubernetes service that you will add to the service catalog.

## 3. Apply the manifests in your cluster and see the service being reported in the service catalog

From the root of your local `a8r-gitops-example` demo repository, apply the Kubernetes manifests to your cluster.

```
kubectl create namespace gitops-demo && \
    kubectl apply -n gitops-demo -f ./manifests
```

This will create a `Deployment`, `Service` and `AmbassadorMapping` in the `gitops-demo` namespace in your cluster. All resources applied in this guide can be removed by running `kubectl delete namespace gitops-demo` when you're done with the quickstart.

<Alert severity="info">The <a href="https://app.getambassador.io/cloud/services" target="_blank">Service Catalog</a> should display information about the `quote` service!</Alert>

## 4. Configure GitHub integration

1. Navigate to the <a href="https://app.getambassador.io/cloud/settings/teams" target="_blank">Teams Settings page</a> in Ambassador Cloud.

2. Click the "Integrations" button to navigate to the github settings page.

    ![Integrations](../../images/gitops-quickstart-02.png)

3. Click "Manage Permissions". You will be taken to github.com and asked to choose which account you want to install Ambassador DCP.

    ![Manage Permissions](../../images/gitops-quickstart-03.png)

4. Select the account which contains the `a8r-gitops-example` demo repository.

    ![Git Account](../../images/gitops-quickstart-x1.png)

    Configure the installation for the demo repository:

    ![Git configure](../../images/gitops-quickstart-x2.png)

    After clicking "Install", you will be directed back to the Ambassador DCP.

4. Once back in the Ambassador DCP, find the `a8r-gitops-example` demo repository in the list of repositories, and click "Enable".

    ![Enable Repository](../../images/gitops-quickstart-04.png)

5. Configure the Ambassador DCP to access your cluster information and Kubernetes manifests from git.

    For the `manifest` text box, enter the relative path to your Kubernetes manifest files in your repository. For the demo repository, the [manifests](https://github.com/datawire/a8r-gitops-example/tree/main/manifests) live in the `manifest` directory.

    Select the cluster you initialized in step 1 from the `cluster` drop down.

    Hit `Submit`. This will trigger the Ambassador DCP to create a pull request with the information you just entered into your demo repository.

    ![GitOps Form](../../images/gitops-quickstart-05.png)

6. Click `Submit`. This will trigger the Ambassador DCP to create a pull request with the information you just entered into your demo repository. The pull request will add a file named `.a8r.yaml` to the root of your repository, and the contents will look something like this:

    ```
    k8s_config:
    - manifest_path: /manifests/
      cluster_info:
        cluster_id: 282b8880-24d4-530c-87aa-55f75132985b
        cluster_name: My Awesome Cluster
    ```

7. Merge the pull request to the main branch of your demo repo.

<Alert severity="success"><b>Congrats!</b> Your demo repository is now configured to receive pull request feedback from the Ambassador DCP. </Alert>

## 5. Create a Pull Request

Now that your repository is configured to recieve feedback from the Ambassador DCP, let's create a pull request modifying Kubernetes resources to the demo repository.

Write the following yaml to the `manifests/echo-service.yaml` directory.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-server
  namespace: gitops-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo-server
  template:
    metadata:
      labels:
        app: echo-server
    spec:
      containers:
        - name: echo-server
          image: jmalloc/echo-server
          ports:
            - name: http-port
              containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: echo-server
  namespace: gitops-demo
spec:
  ports:
    - name: http-port
      port: 80
      targetPort: http-port
      protocol: TCP
  selector:
    app: echo-server
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorMapping
metadata:
  name: echo-server
  namespace: gitops-demo
spec:
  hostname: '*'
  prefix: /backend/
  service: echo-server
```

Commit and push changes to Git:

```
git checkout -b new-echo-service
git add manifests/echo-service.yaml
git commit -am 'Add new echo service' && git push -u origin new-echo-service
```

Navigate to github to create a pull request for your change. Make sure the target repository is in your git organization, not `datawire`.

## 6. Review & merge Pull Request

When you create a new pull request that changes your configuration, Ambassador DCP will be automaticall notified. The configuration change that you make is compared to your existing (runtime) configuration change, and the implications of this change are analyzed. Ambassador DCP will post a comment analyzing the consequences of merging the pull request into your main branch.

In this example, Ambassador DCP detects a conflict on the `/backend` route, as multiple `AmbassadorMappings` point to the same route.

![Conflicting Routes](../../images/gitops-quickstart-warning.png)

In this case, we didn't intend to create a route conflict, so we'll make a change to fix this particular problem.

In `manifests/echo-service.yaml`, edit the prefix for the `echo-service` `AmbassadorMapping`, so that it doesn't conflict with the `quote` service:

```diff
 metadata:
   name: echo-server
   namespace: gitops-demo
 spec:
   hostname: '*'
-  prefix: /backend/
+  prefix: /echo-backend/
   service: echo-server
```

Commit and push changes:
```
git add manifests/echo-service.yaml
git commit -am 'Update echo service to avoid route conflicts' && git push -u origin HEAD
```

The Ambassador DCP will update the Pull Request, noting that the new route is entirely new and does not conflict with other routes!
![Fixed Routes](../../images/gitops-quickstart-newroutes.png)

<Alert severity="success"><b>Congratulations!</b> You've just avoided utter disaster in production. </Alert>

## What's Next?

See the [reference](../reference) for more information on automated configuration analysis with the Ambassador DCP.