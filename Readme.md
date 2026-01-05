# Helmfile example for Keycloak deployments

This is an example Helmfile configuration showing, how to use Keylcoak in conjunction 
with Starwit's [PostreSQL](https://github.com/starwit/timescaledb-chart) chart.

## Goal & Motivation
Authentication is an essential aspect to run applications. If running an identity 
provider  (IdP) is too complicated, especially smaller projects may be tempted to 
include that username/password hash table in their application - don't do this! So 
main motivation for this template is to make it as easy as possible, to run Keycloak 
as IdP in your project. 

There is so far no desire to support larger installations, so don't expect replication 
or fail overs. If somethings goes wrong, components will be simply restarted.

## Components
Keycloak needs a database to store user identities. Following diagram shows
how Keycloak is supposed to be deployed to Kubernetes. Next to the necessary 
database it is worth noting, that user traffic and admin console can be 
exposed under different hostnames. 

![Components](./doc/ComponentOverview.drawio.svg)

Liveness/Readyness/Started probes are all checked using Keycloak's internal interface. 

## Helmfile
Example Helmfile can be found [here](helmfile/keycloak-helmfile.yaml.gotmpl).
It contains one environment with sample values. For better integration with
your applications, all values, that needs to be globally available, can be 
placed in config-data helper chart. All config for an imported realm on startup
is a good example.

Secrets like Keycloak admin password are pulled from a environment variables. See
[template](helmfile/env.sh-template) how to set values. There are plans to support
password managing services.

Once you have configured access to target Kubernetes cluster, sourced env-file for
necessary variables, Keycloak can be deployed like so:
```bash
helmfile -e default apply -f keycloak-helmfile.yaml.gotmpl
```

See [Helmfile docs](https://helmfile.readthedocs.io/en/latest/) if you need help, on 
how to use Helmfile

### Configuration
```yaml
    values:
      - namespace: auth-test # namespace to install app
        hostname: yourdomain.com # base hostname
        tls: true # if true all external URLs are https
        auth:
          keycloak_admin_pw: {{ requiredEnv "KEYCLOAK_ADMIN" }} # admin password for console
          keycloak_db_password: {{ requiredEnv "KEYCLOAK_DB_PW" }} # password for database user
          importFileLocation: {{ requiredEnv "REALM_IMPORT_LOCATION" }} # add realm export to conf folder, disables 
          realmname: realmname # name if default realm is imported
          clientId: testClientId # clientid if default realm is imported
          clientSecret: testClientSecret # secret for client if default realm is imported
          realmAdminPw: adminUserPw # realm admin password if default realm is imported
```

Liveness/Readyness/Started probes timeout can be set like so:
```yaml
        startupProbe: |
          httpGet:
            path: '/admin/health/started'
            port: 'http-internal'
          periodSeconds: 25
```
Sometimes Keycloak takes very long to start, so make sure, this values is not preventing a 
successful start. Also make sure, that Keycloak has enough CPU power to run fast enough. Field 
cpu is configuring this:
```yaml
        resources:
          requests:
            memory: "768Mi"
            cpu: "2000m"
```

### Realm Import
A standard use case is, that upon first installation a Keycloak realm is imported.
This way one can create all necessary config, to enable other parts of your application
to use Keycloak as IdP right away. You can also create initial user identities or 
functional users.

Realm is provided as a [config-map](helmfile/config-data/templates/realm.configmap.yaml) in
which realmname and client secret are inserted as values from your env-file. This file 
probably needs adaption to your needs. 

It is then mounted into Keycloak image and is going to be imported upon first start.

If you have an exported realm, you can set this via REALM_IMPORT_LOCATION in [environment file](helmfile/env.sh-template).

### Realm Export
Exporting realms including users can not be done via admin console. Please note that exporting 
large realms, may slow your Keycloak instance down. In any case refer to 
[Keycloak's documenation](https://www.keycloak.org/documentation).

To export a realm including users, you can use Keycloak's command line interface. 
See [Keycloak's manual](https://www.keycloak.org/server/importExport) for more details on 
importing/exporting realm data.

Here is an example on how commands can look like for extracting a realm from a Keycloak instance 
running on Kubernetes.

```bash
# open shell in a Keycloak instace
cd /path/to/keycloak/bin # e.g. /opt/bitnami/keycloak/bin
KC_CACHE=local ./kc.sh export --dir /tmp --users same_file  --realm your-realm
```
Copying exported data to your computer can be done like so:

```bash
kubectl -n your-namespace cp keycloak-0:/tmp/export.json export.json
```

## Docker Compose
TODO

# License and Contribution
TODO