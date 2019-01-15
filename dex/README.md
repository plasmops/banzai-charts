# Dexidp dex chart

## Installing the Chart

```shell
helm repo add banzaicloud-stable http://kubernetes-charts.banzaicloud.com/branch/master
helm install --name dex banzaicloud-stable/dex
```

## Configuration

The following tables lists configurable parameters of the dex chart and their default values.

|               Parameter             |                Description                  |                  Default                 |
| ----------------------------------- | ------------------------------------------- | -----------------------------------------|
|image                                |dex image                                    |quay.io/dexidp/dex                        |
|imageTag                             |image tag                                    |master                                    |
|imagePullPolicy                      |image pull policy                            |IfNotPresent                              |
|replicas                             |number of replicas                           |1                                         |
|postgresql.enabled                   |deploy postgresql                            |true                                      |
|postgresql.postgresqlUsername        |postgresql username                          |dex                                       |
|postgresql.postgresqlPassword        |postgresql password                          |foo                                       |
|postgresql.postgresqlDatabase        |postgresql database                          |dex_db                                    |
|ports[0].name                        |port 0 name                                  |http                                      |
|ports[0].containerPort               |port 0 port                                  |5556                                      |
|posts[0].protocol                    |port 0 protocol                              |TCP                                       |
|service.type                         |service type                                 |ClusterIP                                 |
|service.annotations                  |service annotattions                         |{}                                        |
|certs.tlsName                        |tls secret name                              |dex-web-server-tls                        |
|certs.caName                         |CA secret name                               |dex-web-server-ca                         |
|env                                  |environment variables                        |[]                                        |
|rbac.create                          |RBAC resources should be created             |true                                      |
|serviceAccount.create                |serviceAccount should be created             |true                                      |
|serviceAccount.name                  |name of the ServiceAccount to use            |default                                   |
|config.issuer                        |issuer url                                   |http://127.0.0.1:5556/dex                 |
|config.logger.level                  |logger level                                 |debug                                     |
|config.web.http                      |dex host and port                            |0.0.0.0:5556                              |
|config.storage.type                  |storage type                                 |postgresql                                |
|config.storage.config.database       |storage db                                   |{{ postgresql.postgresqlDatabase }}       |
|config.storage.config.user           |storage user                                 |{{ postgresql.postgresqlUsername }}       |
|config.storage.config.password       |storage password                             |{{ postgresql.postgresqlPassword }}
|config.storage.config.host           |storage host                                 |Release.Name-postgresql.Release.Namespace..svc.cluster.local:5432|
|config.storage.config.ssl.mode       |storage ssl mode                             |disable                                   |
|config.staticClients                 |client config (use config file see below)    |[]                                      |
|config.connectors                    |connectors config (use config file see below)|[]                                        |
|nodeSelector                         |nodeselector                                 |{}                                        |

An example of config/connectors-config.yaml:
```yaml
- type: github
  id: github
  name: GitHub
  config:
    loadAllGroups: true
    clientID: <github-client-id>
    clientSecret: <fithub-client-secret>
    redirectURI: http://127.0.0.1:5556/dex/callback

- type: ldap
  name: OpenLDAP
  id: ldap
  config:
    host: <ladp-ip>:<ldap-port>
    # No TLS for this setup.
    insecureNoSSL: true
    # This would normally be a read-only user.
    bindDN: cn=admin,dc=example,dc=org
    bindPW: admin
    usernamePrompt: Email Address
    userSearch:
      baseDN: ou=People,dc=example,dc=org
      filter: "(objectClass=person)"
      username: mail
      # "DN" (case sensitive) is a special attribute name. It indicates that
      # this value should be taken from the entity's DN not an attribute on
      # the entity.
      idAttr: DN
      emailAttr: mail
      nameAttr: cn
    groupSearch:
      baseDN: ou=Groups,dc=example,dc=org
      filter: "(objectClass=groupOfNames)"
      # A user is a member of a group when their DN matches
      # the value of a "member" attribute on the group entity.
      userAttr: DN
      groupAttr: member
      # The group name should be the "cn" value.
      nameAttr: cn
```

An example of config/client-config.yaml:
```yaml
- id: example-app
  redirectURIs:
  - 'http://127.0.0.1:9000/auth/dex/callback'
  - 'http://localhost:9000/auth/dex/callback'
  name: 'Example App'
  secret: example-secret