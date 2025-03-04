import Alert from '@material-ui/lab/Alert';

# Upgrade $productName$ 2.0.5 to $productName$ $versionTwoX$ (Helm)

<Alert severity="info">
  This guide covers migrating from $productName$ 2.0.5 to $productName$ $versionTwoX$. If
  this is not your <b>exact</b> situation, see the <a href="../../../../migration-matrix">migration
  matrix</a>.
</Alert>

<Alert severity="warning">
  This guide is written for upgrading an installation originally made using Helm.
  If you did not install with Helm, see the <a href="../../../yaml/edge-stack-2.0/edge-stack-2.X">YAML-based
  upgrade instructions</a>.
</Alert>

<Alert severity="warning">
  <b>Upgrading from $productName$ 2.0.5 to $productName$ $versionTwoX$ typically requires downtime.</b>
  In some situations, Ambassador Labs Support may be able to assist with a zero-downtime migration;
  contact support with questions.
</Alert>

Migrating from $productName$ 2.0.5 to $productName$ $versionTwoX$ is a four-step process:

1. **Install new CRDs.**

   Before installing $productName$ $versionTwoX$ itself, you need to update the CRDs in
   your cluster; Helm will not do this for you. This is mandatory during any upgrade of $productName$.

   ```
   kubectl apply -f https://app.getambassador.io/yaml/edge-stack/$versionTwoX$/aes-crds.yaml
   kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system
   ```

   <Alert severity="info">
     $productName$ $versionTwoX$ includes a Deployment in the `emissary-system` namespace
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

2. **Delete $productName$ 2.0.5 Deployment.**

   <Alert severity="warning">
     Delete <b>only</b> the Deployment for $productName$ 2.0.5 in order to preserve all of
     your existing configuration.
   </Alert>

   Use `kubectl` to delete the Deployment for $productName$ 2.0.5. Typically, this will be found
   in the `ambassador` namespace.

   ```
   kubectl delete -n ambassador deployment edge-stack
   ```

3. **Install $productName$ $versionTwoX$.**

   After installing the new CRDs, use Helm to install $productName$ $versionTwoX$. Start by
   making sure that your `datawire` Helm repo is set correctly:

   ```bash
   helm repo remove datawire
   helm repo add datawire https://app.getambassador.io
   helm repo update
   ```

   Then, install $productName$ in the `$productNamespace$` namespace. If necessary for
   your installation (e.g. if you were running with `AMBASSADOR_SINGLE_NAMESPACE` set),
   you can choose a different namespace.

   ```bash
   helm install -n $productNamespace$ \
        $productHelmName$ datawire/$productHelmName$ && \
   kubectl rollout status  -n $productNamespace$ deployment/$productDeploymentName$ -w
   ```

   <Alert severity="warning">
     You must use the <a href="https://artifacthub.io/packages/helm/datawire/edge-stack/$aesChartVersion$"><code>$productHelmName$</code> Helm chart</a> to install $productName$ 2.X.
     Do not use the <a href="https://artifacthub.io/packages/helm/datawire/ambassador/6.9.3"><code>ambassador</code> Helm chart</a>.
   </Alert>
