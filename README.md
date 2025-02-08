# Passbolt with Postgresql Helm Chart

Passbolt is an open source credential platform for modern teams. A versatile, battle-tested solution to manage and collaborate on passwords, accesses, and secrets. All in one.

![Version: 1.0.0](https://img.shields.io/badge/Version-1.0.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 4.11.0](https://img.shields.io/badge/AppVersion-4.11.0-informational?style=flat-square)

**Homepage:** <https://github.com/DeathByMisadventure/passbolt-helm>

## About Passbolt

This helm chart deployment for Kubernetes is designed for an easy to configure and maintain deployment into
your corporate infrastructure.

[Passbolt](https://www.passbolt.com/) is an open-source, self-hosted password manager designed for teams and organizations. It allows users to securely store, share, and manage passwords, while also providing features for password generation, authentication, and access control.

Key Features:

1. **Password Storage**: Passbolt allows users to store passwords in a secure, encrypted vault.
2. **Password Sharing**: Users can share passwords with others, either individually or through groups, while maintaining control over access and permissions.
3. **Password Generation**: Passbolt includes a built-in password generator that creates strong, unique passwords for users.
4. **Authentication**: Passbolt supports various authentication methods, including username/password, two-factor authentication (2FA), and single sign-on (SSO) through LDAP, Active Directory, or SAML.
5. **Access Control**: Passbolt enables administrators to manage user access and permissions, including role-based access control (RBAC) and attribute-based access control (ABAC).
6. **Encryption**: Passbolt uses end-to-end encryption, ensuring that only authorized users can access password data.
7. **Mobile and Desktop Apps**: Passbolt offers mobile apps for Android and iOS, as well as desktop apps for Windows, macOS, and Linux.

**Benefits:**

1. **Security**: Passbolt provides a secure way to store and share sensitive password information, reducing the risk of password-related breaches.
2. **Convenience**: Passbolt simplifies password management, allowing users to access and share passwords from a single, intuitive interface.
3. **Collaboration**: Passbolt facilitates teamwork and collaboration by enabling secure password sharing and access control.
4. **Compliance**: Passbolt helps organizations meet regulatory requirements and industry standards for password management and security.
5. **Customization**: As an open-source tool, Passbolt can be customized and extended to meet specific organizational needs.

**Self-Hosted**: Passbolt is designed to be self-hosted, giving organizations full control over their password management infrastructure and data. This approach ensures that sensitive password information is not stored in the cloud or managed by a third-party provider.

**Community-Driven**: Passbolt is an open-source project with an active community of contributors, users, and supporters. This community-driven approach ensures that the tool is continuously improved, updated, and secured.

Overall, Passbolt is a robust, open-source password manager that provides a secure, convenient, and collaborative solution for teams and organizations to manage passwords and sensitive information.

## Prerequisites

A Postgresql server for Passbolt must be deployed into the environment, or configure
the values to enable the built in Postgresql instance.  For enterprise use, an external
Postgresql server is recommended.

## Configure values.yaml or valuesoverride.yaml

A few variables need to be created in the values.yaml or in a values override. Documentation
of the variables are below or in the values.yaml file.

[Additional configuration options](https://hub.docker.com/r/passbolt/passbolt/) for a containerized
deployment of Passbolt is available.

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

If using the internal Postgres deployment, create the PV and deploy it according to your Kubernetes
infrastructure requirements.

Customize any values by creating a local-values.yaml and including the overrides from the main values.yaml.

Add a secret for the ingress TLS certificate if needed, and configure TLS in the values.
[Documentation](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_secret_tls/).
If no certificate is provided, then a self signed certificate is generated automatically.

`kubectl create --namespace passbolt secret tls external-tls-cert --cert=tls.crt --key=tls.key`

Install the helm chart with:

`helm install --namespace passbolt -f ./local-values.yaml deployment-name .`

## Creating First Admin User

At the end of the installation, instructions will be provided on the command to copy/paste to create the
initial administrator account.  From there, you can log in with the URL provided to continue setting up
the administrator account, then configure the Passbolt application, and finally add users.

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://kubernetes.github.io/ingress-nginx | ingress-nginx | ~4.11.3 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| firstAdmin | object | `{"email":"admin@domain.com","firstName":"First","surname":"Admin"}` | Information on setting up the initial admin account |
| imagePullSecrets | list | `[]` | Used for private repositories |
| ingress-nginx.controller.config | object | `{"enable-modsecurity":"true","enable-owasp-modsecurity-crs":"true","modsecurity-snippet":"SecRuleEngine On\nSecStatusEngine Off\nSecAuditLog /dev/stdout\nSecAuditLogFormat JSON\nSecAuditLogParts ABCFHKZ\nSecAuditEngine RelevantOnly\nSecPcreMatchLimit 500000\nSecPcreMatchLimitRecursion 500000\nSecAction \"id:900200,phase:1,nolog,pass,t:none,setvar:tx.allowed_methods=GET HEAD POST OPTIONS PUT PATCH DELETE\"\nSecRuleRemoveById 920440\n","modsecurity-transaction-id":"$request_id"}` | Ingress-NGINX Configuration for MODSECURITY OWASP Protection Comment out this entire section to disable the WAF |
| ingress.annotations | object | `{"nginx.ingress.kubernetes.io/force-ssl-redirect":"true"}` | Ingress annotations |
| ingress.className | string | `"nginx"` | Ingress class type |
| ingress.enabled | bool | `true` | Enable Ingress |
| ingress.hostname | string | `"passbolt.localdev.me"` | ingress external hostname |
| ingress.tls | object | `{"secretName":null}` | Enable TLS with a specific preconfigured TLS certificate secret |
| passbolt | object | `{"config":null,"image":{"pullPolicy":"IfNotPresent","repository":"passbolt/passbolt","tag":"latest"},"name":"passbolt","pvc":{"storageRequest":"100Mi"},"replicas":1,"resources":{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"1000m","ephemeral-storage":"2Gi","memory":"1Gi"}},"securityContext":null,"service":{"name":"http","port":80,"type":"ClusterIP"}}` | Image pull secrets |
| passbolt.config | string | `nil` | Configuration data passed directly into passbolt pod environment variables as is Additional configuration settings are available at * https://github.com/passbolt/passbolt_api/blob/master/config/default.php * https://github.com/passbolt/passbolt_api/blob/master/config/app.default.php Note: It is recommended to set email options through the GUI once deployed |
| passbolt.image.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| passbolt.image.repository | string | `"passbolt/passbolt"` | Image repository |
| passbolt.image.tag | string | `"latest"` | Image tag defaults to Chart AppVersion |
| passbolt.name | string | `"passbolt"` | Container Name |
| passbolt.pvc.storageRequest | string | `"100Mi"` | PVC size for the pod data |
| passbolt.replicas | int | `1` | Number of pod replicas |
| passbolt.resources | object | `{"limits":{"cpu":"1000m","memory":"1Gi"},"requests":{"cpu":"1000m","ephemeral-storage":"2Gi","memory":"1Gi"}}` | Pod assigned resources |
| passbolt.securityContext | string | `nil` | Pod security context |
| passbolt.service | object | `{"name":"http","port":80,"type":"ClusterIP"}` | Passbolt container service port information |
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
