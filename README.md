# ![](assets/web/proxyinjector-round-100px.png) Proxy Injector
A Kubernetes controller to inject an authentication proxy container to relevant pods

[![Get started with Stakater](https://stakater.github.io/README/stakater-github-banner.png)](http://stakater.com/?utm_source=ProxyInjector&utm_medium=github)

## Problem Statement

We want to automatically inject an authentication proxy container in a pod, for any deployment that requires to connect
 to our SSO provider, instead of manually adding a sidecar container with each deployment 

## Solution

This controller will continuously watch deployments in specific or all namespaces, and automatically add a sidecar container
 for the authentication proxy. Configuration for the proxy is managed through annotations of the respective deployment
 or with ConfigMap of the ProxyInjector.

### Supported proxies

For now the ProxyInjector only supports [Keycloak Gatekeeper](https://github.com/keycloak/keycloak-gatekeeper)
 as the authentication proxy, to work with [Keycloak Server](https://github.com/keycloak/keycloak)


## Usage

The following quickstart let's you set up ProxyInjector:

1. Add configuration to the ProxyInjector
    The following arguments can either be added to the proxy injector `config.yaml` in the ConfigMap/Secret for centralized configuration,
     or as annotations on the individual target deployments with a `authproxy.stakater.com/` prefix. In case of both,
     the deployment annotation values will override the central configuration. 

    | Key              | Description                                                               |
    |------------------|---------------------------------------------------------------------------|
    | listen           | the interface address and port the proxy should be listening on           |
    | upstream-url     | url for the upstream endpoint you wish to proxy                           |
    | resources        | list of resources to proxy uri, methods, roles                            |
    | client-id        | client id used to authenticate to the oauth service                       |
    | client-secret    | client secret used to authenticate to the oauth service                   |
    | gatekeeper-image | Keycloak Gatekeeper image e.g. `keycloak/keycloak-gatekeeper:6.0.1` |

The rest of the available options can be found at the [Keycloak Gatekeeper documentation](https://www.keycloak.org/docs/latest/securing_apps/index.html#configuration-options)

Note 1: See the section `Using Secrets` below if you do not want to use ConfigMap (because `client-id` and `client-secret` in plain text) and want to use Secrets to hide them.

2. Deploy the controller by running the following command:

    For Kubernetes Cluster using kubectl
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/stakater/ProxyInjector/master/deployments/kubernetes/proxyinjector.yaml -n default

3. When deploying any application that needs Keycloak authentication, add the following annotations to the `deployment`. The `service` will not need changes as such, all configuration can be provided as annotations in the deployment for the app. And proxy injector automatically modifies the service when injecting the sidecar container.
  
    | Key                                        | Description                                                                                                                                       |
    |--------------------------------------------|--------------------------------------------------------|
    | authproxy.stakater.com/enabled             | (**REQUIRED** true/false, default=false) Enables Keycloak gatekeeper configuration |
    | authproxy.stakater.com/secret-name         | (**REQUIRED**) name for the secret to inject as a `envFrom` object                       |
    | authproxy.stakater.com/source-service-name | (**REQUIRED**) Name of service that needs to be reconfigured to connect to the proxy. instead of the service directly routing to the app container, it will now route to the proxy sidecar instead. |
    | authproxy.stakater.com/target-port         | (default=80) the port on the pod where the proxy sidecar (keycloak gatekeeper) will be listening. If not specified, the default value of 80 is used. This port should match the `listen` configuration |
    | authproxy.stakater.com/resources           | String of resources separated by `&` e.g. (`uri=/*|white-listed=true&uri=/css/*|white-listed=false|methods=GET,POST`)

    The `authproxy.stakater.com/listen` annotation or the `listen` property in the ProxyInjector ConfigMap should
    specify where the proxy sidecar will listen for incoming requests, e.g. "0.0.0.0:80" i.e. local port 80
 

### Using Secrets

To use secrets:
    
  1. Open [values.yaml](https://github.com/stakater/ProxyInjector/blob/master/deployments/kubernetes/chart/proxyinjector/values.yaml) file by navigating to `deployments/kubernetes/chart/proxyinjector/`
  
  2. Set `proxyinjector.mount` equals to `"secret"` and pass the data in the data section at the bottom.
  
  3. Deploy the secret with `helm install`
  

To use existing Secrets:

  1. Set `proxyinjector.mount` equals to `"secret"`
  2. set `proxyinjector.existingSecret` equals to `EXISTING_SECRET_NAME`

#### Passing Client id and Secret
  1. Create a Secret in the namespace in which the targets of ProxyInjector are deployed. The secret **MUST** be set this way:

  ```yaml
  apiVersion: v1
  data:
  CLIENT_ID: <base64 of client id>
  CLIENT_SECRET: <base64 of client secret>
  kind: Secret
  metadata:
    name: <name of the secret>
    namespace: <target namespace>
  type: Opaque
  ```

  2. Set in the annotations of the target deployment `authproxy.stakater.com/secret-name` as the name of the secret you just made. The proxyInjector will add an `envFrom` section in the injected deployment.
      (Be aware that this will work only if you also edit the Keycloak image in order to accept CLIENT_ID and CLIENT_SECRET as env vars.)

### Using ConfigMap

To pass user credentials/ API keys in secrets:
     
  1. Open [values.yaml](https://github.com/stakater/ProxyInjector/blob/master/deployments/kubernetes/chart/proxyinjector/values.yaml) file by navigating to `deployments/kubernetes/chart/proxyinjector/`
  
  2. Set `proxyinjector.mount` equals to `"configmap"` and pass the data in the data section at the bottom.
  
  3. Run `helm template . > proxyinjector.yaml`
  
  4. Deploy using the `Deploying` section below.

### Deploying

You can deploy the controller in the namespace you want to monitor by running the following kubectl command:

```bash
kubectl apply -f proxyinjector.yaml -n <namespace>
```

*Note*: Before applying `proxyinjector.yaml`, You need to modify the namespace in the `RoleBinding` subjects section to the namespace you want to apply RBAC to.

## Help

### Documentation
You can find more documentation [here](docs/)

### Have a question?
File a GitHub [issue](https://github.com/stakater/ProxyInjector/issues), or send us an [email](mailto:hello@stakater.com).

### Talk to us on Slack
Join and talk to us on the #tools-proxyinjector channel for discussing the ProxyInjector

[![Join Slack](https://stakater.github.io/README/stakater-join-slack-btn.png)](https://stakater-slack.herokuapp.com/)
[![Chat](https://stakater.github.io/README/stakater-chat-btn.png)](https://stakater.slack.com/messages/CFCP3MUR4/)

## License

Apache2 © [Stakater](http://stakater.com)

## About

The `ProxyInjector` is maintained by [Stakater][website]. Like it? Please let us know at <hello@stakater.com>

See [our other projects][community]
or contact us in case of professional services and queries on <hello@stakater.com>

  [website]: http://stakater.com/
  [community]: https://www.stakater.com/projects-overview.html

## Contributers

Stakater Team and the Open Source community! :trophy:
