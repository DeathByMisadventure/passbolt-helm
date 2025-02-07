# Passbolt with Postgresql Helm Chart

A Helm chart for Kubernetes

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 0.1.0](https://img.shields.io/badge/AppVersion-0.1.0-informational?style=flat-square)

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://kubernetes.github.io/ingress-nginx | ingress-nginx | ~4.11.3 |

## Prerequisites

A Postgresql server for Passbolt must be deployed into the environment, or configure
the values to enable the built in Postgresql instance.  For enterprise use, an external
Postgresql server is recommended.

## Configure values.yaml or valuesoverride.yaml

A few variables need to be created in the values.yaml or in a values override. Documentation
of the variables are below or in the values.yaml file.

Optional configuration for Guacamole can be configured under guacamole.settings: such as
OpenID authentication.

All environmental variable names will automatically be uppercased and properly underscored.

## Use Internal Postgresql

If you utilize the instance of Postgresql included in this chart, a Persistent Volume (PV) is recommended.
Create a PV according to the requirements of your environment.

This demo kubernetes PV will create with storage in your kubernetes host's /tmp/postgres-pv directory.
Remember to deploy this in the same namespace as the helm deployment.

The two keys are the `storageClassName:` and the `metadata.name:` which will then need to match
`postgres.pvc.storageClassName:` and the `postgres.pvc.selector:` in values.yaml for the Persistent Volume Claim
used in the helm chart.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /tmp/postgres-pv
```

## NGINX Modsecurity OWASP Web Application Firewall (WAF)

`ingress-nginx.controller.config` in the values.yaml configuration enables the [Web Application Firewall (WAF)](https://kubernetes.github.io/ingress-nginx/user-guide/third-party-addons/modsecurity/).  The Ingress WAF comes with the [OWASP Core Rule Set (CRS)](https://owasp.org/www-project-modsecurity-core-rule-set/).

The OWASP CRS provides the rules for the NGINX ModSecurity WAF to block SQL Injection (SQLi), Remote Code Execution (RCE), Local File Include (LFI), cross‑site scripting (XSS), and many other attacks.

The necessary rule changes to work with Guacamole have been applied.

## Deployment

Import the dependencies for the helm chart:

`helm dependency update`

Create a namespace if needed:

`kubectl create ns passbolt`

If using the internal Postgres deployment, create the PV and deploy it.

Customize any values by creating a local-values.yaml and including the overrides from the main values.yaml.

Add a secret for the ingress TLS certificate if needed, and configure TLS in the values.
[Documentation](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_secret_tls/)

`kubectl create --namespace passbolt secret tls external-tls-cert --cert=tls.crt --key=tls.key`

Install the helm chart with:

`helm install --namespace passbolt -f ./local-values.yaml deployment-name .`

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| imagePullSecrets | list | `[]` | Used for private repositories |
| ingress-nginx.controller.config | object | `{"enable-modsecurity":"true","enable-owasp-modsecurity-crs":"true","modsecurity-snippet":"SecRuleEngine On\nSecStatusEngine Off\nSecAuditLog /dev/stdout\nSecAuditLogFormat JSON\nSecAuditLogParts ABCFHKZ\nSecAuditEngine RelevantOnly\nSecPcreMatchLimit 500000\nSecPcreMatchLimitRecursion 500000\nSecAction \"id:900200,phase:1,nolog,pass,t:none,setvar:tx.allowed_methods=GET HEAD POST OPTIONS PUT PATCH DELETE\"\nSecRuleRemoveById 920440\n","modsecurity-transaction-id":"$request_id"}` | Ingress-NGINX Configuration for MODSECURITY OWASP Protection Comment out this entire section to disable the WAF |
| ingress.annotations | object | `{"nginx.ingress.kubernetes.io/force-ssl-redirect":"true"}` | Ingress annotations |
| ingress.className | string | `"nginx"` | Ingress class type |
| ingress.enabled | bool | `true` | Enable Ingress |
| ingress.hostname | string | `"passbolt.localdev.me"` |  |
| ingress.hosts[0] | object | `{"host":"passbolt.localdev.me","paths":["/"]}` | Ingress hostname to bind |
| ingress.hosts[0].paths | list | `["/"]` | Ingress host paths |
| ingress.tls | string | `nil` | Enable TLS with a specific preconfigured TLS certificate |
| passbolt | object | `{"config":{"emailTransportDefaultHost":"smtp.domain.tld","emailTransportDefaultPort":"587"},"image":{"pullPolicy":"IfNotPresent","repository":"passbolt/passbolt","tag":"latest"},"name":"passbolt","pvc":{"storageRequest":"100Mi"},"replicas":1,"resources":{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"1000m","ephemeral-storage":"2Gi","memory":"1Gi"}},"securityContext":null,"service":{"name":"http","port":80,"type":"ClusterIP"}}` | Image pull secrets |
| passbolt.config | object | `{"emailTransportDefaultHost":"smtp.domain.tld","emailTransportDefaultPort":"587"}` | Configuration data passed directly into passbolt pod environment variables as is |
| passbolt.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| passbolt.image.repository | string | `"passbolt/passbolt"` | Image repository |
| passbolt.image.tag | string | `"latest"` | Image tag defaults to Chart AppVersion |
| passbolt.name | string | `"passbolt"` | Container Name |
| passbolt.pvc.storageRequest | string | `"100Mi"` | PVC size for the pod data |
| passbolt.replicas | int | `1` | Number of pod replicas |
| passbolt.resources | object | `{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"1000m","ephemeral-storage":"2Gi","memory":"1Gi"}}` | Pod assigned resources |
| passbolt.securityContext | string | `nil` | Pod security context |
| passbolt.service | object | `{"name":"http","port":80,"type":"ClusterIP"}` | Passbolt container service information |
| postgres.database | string | `"passbolt"` | Database name |
| postgres.enabled | bool | `true` | Enable internal postgres database |
| postgres.hostname | string | `"postgres.localdev.me"` | If internal postgres is disabled, the external database hostname |
| postgres.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| postgres.image.repository | string | `"postgres"` | Image repository |
| postgres.image.tag | string | `"16-alpine"` | Image tag |
| postgres.name | string | `"postgres"` | Container Name |
| postgres.password | string | `"p@ssb0lt"` | Postgres password |
| postgres.pvc.selector | string | `nil` | Selector to match pre-provisioned PV |
| postgres.pvc.storageClassName | string | `nil` | Storage Class Name for a pre-provisioned PV |
| postgres.pvc.storageRequest | string | `"100Mi"` | Postgres PVC storage request size |
| postgres.replicas | int | `1` | Number of pod replicas |
| postgres.resources | object | `{"limits":{"cpu":"100m","ephemeral-storage":"2Gi","memory":"1Gi"},"requests":{"cpu":"100m","ephemeral-storage":"2Gi","memory":"1Gi"}}` | Pod assigned resources |
| postgres.securityContext | string | `nil` | Pod security context |
| postgres.service.port | string | `"5432"` | Service port number |
| postgres.user | string | `"passbolt"` | Postgres username |
