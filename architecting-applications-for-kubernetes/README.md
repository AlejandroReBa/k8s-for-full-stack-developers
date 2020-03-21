# Architecting Applications for Kubernetes

Notes for the guide [Architecting Application for Kubernetes][1] which explains some Software Design and Architecture for cloud-native applications, useful for Kubernetes deployments.

Your application must be created to be scalable, horizontally scalable (which is supported in cloud environments like Kubernetes). The *microservices* architecture is very suitable for this scalability demands: easy to scale your application and support the demands of the users properly divided by sections. In addition, each (*micro*) service must be observable, resilient and environment-adaptable (depending on your cloud provider).

One methology that gained attraction amongs cloud-native development and operations is the [Twelve-Factor App][2]. Includes twelve "tips" to help teams to create a great software for a such complex enviroment that is Kubernetes:

1. [Codebase][12-codebase]: Use version control (like git) to develop the code.
2. [Depdendencies][12-dependencies]: Explicitly define which are the dependencies of your app. Use package managers (from your language - like `pip`, `npm`/`yarn`, `go modules`...) or put the dependency code in the repository either using [git submodules][3] or providing the files in there. Never rely on a dependency to be installed in the system (provided by some magical elf).
3. [Config][12-config]: Separe configuration from your application. Use environment-provided configuration (like environment variables, secrets, bind mounts to files or folders...).
4. [Backing services][12-backing-services]: Other services used to support the execution of one of your services should be treated as attached resources, that uses network for comunication (can be datastores, caching systems, SMTP servers...).
5. [Build, release, run][12-build-release-run]: Explicitly separe the build step from the release step and run step. The build step creates an artifact, the release grabs the artifact and a configuration and run step runs the app from the release.
6. [Processes][12-processes]: Services runs in stateless processes. They must not store any state, but they can rely on datastores to implement the state, which can be accessible for each instance of the service (shared state).
7. [Port binding][12-port-binding]: Export services via port binding. Routing and request forwarding must be done externally.
8. [Concurrency][12-concurrency]: Applications must support multiple running instances without any code modifications (stateless helps to accomplish that). Services will scale up and down and may run in the same server on in different servers, and should not affect service functionallity.
9. [Disposability][12-disposability]: Fast startup and shutdown, without side-effects.
10. [Dev/prod parity][12-dev-prod-parity]: Keep development, staging and production environments as similar as possible. This helps find out bugs, or replicate issues easily, and improves testings from QA teams.
11. [Logs][12-logs]: Logs are events that must be streamed through standard outputs (`stdout` and `stderr`) and should not try to manage logs indepently.
12. [Admin processes][12-admin-processes]: One-off administration processes should be run against specific releases and shipped with the main process code.

There are other tips that should be taken in consideration to make good software and better operations. The first is to create optimized containers by making optimized images. They must be as small as possible and include only the necessary code to run. Images based on [`alpine`][4] are probably the best tradeoff between small distro with enough tools to extend the image. Its base image is ~5MB! Also images based on `scratch` (nothing on the image) are suitable for this but you are in charge to provide any runtime needs for your app. Not only the base image is important, your image build process is too. [Multi-stage][5] builds allows images to split the build between different stages to avoid to fill the image with build garbage. The new [BuildKit][6]-based image building from Docker, can improve build times considerably.

The basic build unit of Kubernetes are **Pods**. The structure of your pods should be small as possible. In general terms, they will consist only of one type of containers, but some containers require other containers that supports the first: supporting containers. Imagine that you have a service that connects to other service but the second can be changed from one provider to another, and this will make change the API contract. A supporting container can be bundled so the first container calls to the supporting container and it will contact the other service acting as a "bridge". From [this paper][7], there are three identified types of supporting containers:

- Sidecar: Extends and enhances the functionallity of the container.
- Ambassador: Abstracts remote resources.
- Adaptor: Translates data, contracts or interfaces from the container to another resource to standarize them.

One of the points mentioned before is *config*. Kubernetes **ConfigMaps** and **Secrets** helps to provide the configuration from the environment. **ConfigMaps** provides Pods with environment variables and/or mounted files to configure the app for the release. **Secrets** do the same but are stored securely in the cluster because contains sensible data that must not be exposed easily.

**Readiness** and **Liveness** probes helps the cluster to understand when the app is ready to serve requests (readiness) and whether the app is alive and can handle requests (liveness). Those probes can help the cluster and other services how to manage the containers and manage its live-cycle.

The recommended abstraction to manage the deployment of your app is **Deployments**. They provide probe-aware life-cycle container management, support for A/B version deployment, easy version rollback, service scaling and other cool stuff :)

To access services from other services, **ClusterIp** is your man. This services provides stable IP addresses accessible from the entire cluster (from within containers or from outside). To help with accessibility, a DNS service can help identify services by domain names instead of IP adresses.

To access services from outside, there are several ways to make it. The first is to use **load balancers** provided by the cloud provider, but they can be expensive if you have lot of them. So a new type of object comes to help you: **Ingress**. Services have **ingress rules** that helps the **ingress controller** (a nginx or traefik) send the requests to the right service.

Finally: **DO USE DECLARATIVE `yaml` FILES**.

  [1]: https://www.digitalocean.com/community/tutorials/architecting-applications-for-kubernetes
  [2]: https://12factor.net
  [12-codebase]: https://12factor.net/codebase
  [12-dependencies]: https://12factor.net/dependencies
  [3]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
  [12-config]: https://12factor.net/config
  [12-backing-services]: https://12factor.net/backing-services
  [12-build-release-run]: https://12factor.net/build-release-run
  [12-processes]: https://12factor.net/processes
  [12-port-binding]: https://12factor.net/port-binding
  [12-concurrency]: https://12factor.net/concurrency
  [12-disposability]: https://12factor.net/disposability
  [12-dev-prod-parity]: https://12factor.net/dev-prod-parity
  [12-logs]: https://12factor.net/logs
  [12-admin-processes]: https://12factor.net/admin-processes
  [4]: https://hub.docker.com/_/alpine
  [5]: https://docs.docker.com/develop/develop-images/multistage-build/
  [6]: https://docs.docker.com/develop/develop-images/build_enhancements/
  [7]: https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45406.pdf
