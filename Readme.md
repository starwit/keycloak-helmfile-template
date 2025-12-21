# Helmfile example for Keycloak deployments

This is an example Helmfile configuration showing, how to use Keylcoak in conjunction with Starwit's [PostreSQL](https://github.com/starwit/timescaledb-chart) chart.

## Goal & Motivation
Authentication is an essential aspect to run applications. If running an identity provider  (IdP) is too complicated, especially smaller projects may be tempted to include that username/password hash table in their application - don't do this! So main motivation for this template is to make it as easy as possible, to run Keycloak as IdP in your project. 

There is so far no desire to support larger installations, so don't expect replication or fail overs. If somethings goes wrong, components will be simply restarted.

## Components
Keycloak needs a database to store user identities. Following diagram shows
how Keycloak is supposed to be deployed to Kubernetes. Next to the necessary 
database it is worth noting, that user traffic and admin console can be 
exposed under different hostnames. This way you can decide, whether admin 
console is reachable by users.

![Components](./doc/ComponentOverview.drawio.svg)

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

See [Helmfile docs](https://helmfile.readthedocs.io/en/latest/) if you need help, on how to use Helmfile

### Realm Import
A standard use case is, that upon first installation a Keycloak realm is imported.
This way one can create all necessary config, to enable other parts of your application
to use Keycloak as IdP right away. You can also create initial user identities or 
functional users.

Realm is provided as a [config-map](helmfile/config-data/templates/realm.configmap.yaml) in
which realmname and client secret are inserted as values from your env-file. This file 
probably needs adaption to your needs.

It is then mounted into Keycloak image and is going to be imported upon first start.

## Docker Compose
TODO

# License and Contribution
TODO