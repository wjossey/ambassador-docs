import Alert from '@material-ui/lab/Alert';

# Upgrade $productName$ 2.1.0 or 2.1.1 to $productName$ $version$ (Helm)

<Alert severity="info">
  This guide covers migrating from $productName$ 2.1.0 or 2.1.1 to $productName$ $version$. If
  this is not your <b>exact</b> situation, see the <a href="../../../../migration-matrix">migration
  matrix</a>.
</Alert>

<Alert severity="warning">
  This guide is written for upgrading an installation originally made using Helm.
  If you did not install with Helm, see the <a href="../../../yaml/emissary-2.1/emissary-2.1">YAML-based
  upgrade instructions</a>.
</Alert>

Since $productName$'s configuration is entirely stored in Kubernetes resources, upgrading between minor
versions is straightforward.

Migration is a two-step process:

1. **Install new CRDs.**

   Before installing $productName$ $version$ itself, you need to update the CRDs in
   your cluster; Helm will not do this for you. This is mandatory during any upgrade of $productName$.

   ```
   kubectl apply -f https://app.getambassador.io/yaml/emissary/$version$/emissary-crds.yaml
   kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system
   ```

   <Alert severity="info">
     $productName$ $version$ includes a Deployment in the `emissary-system` namespace
     called <code>$productDeploymentName$-apiext</code>. This is the APIserver extension
     that supports converting $productName$ CRDs between <code>getambassador.io/v2</code>
     and <code>getambassador.io/v3alpha1</code>. This Deployment needs to be running at
     all times.
   </Alert>

   <Alert severity="warning">
     If the <code>$productDeploymentName$-apiext</code> Deployment's Pods all stop running,
     you will not be able to use <code>getambassador.io/v3alpha1</code> CRDs until restarting
     the <code>$productDeploymentName$-apiext</code> Deployment.
   </Alert>

2. **Install $productName$ $version$.**

   After installing the new CRDs, use Helm to upgrade $productName$ $version$:

      ```bash
      helm upgrade emissary-ingress && \
      kubectl rollout status  -n emissary deployment/emissary-ingress -w
      ```

   <Alert severity="warning">
    You must use the <a href="https://github.com/emissary-ingress/emissary/tree/release/v2.1/charts/emissary-ingress"><code>$productHelmName$</code> Helm chart</a> for $productName$ 2.X.
    Do not use the <a href="https://github.com/emissary-ingress/emissary/tree/release/v1.14/charts/ambassador"><code>ambassador</code> Helm chart</a>.
   </Alert>
