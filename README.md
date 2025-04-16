# Traefik Homelab

This is a companion repository for my blog post [**Migrating To Traefik**](https://blog.foxxmd.dev/posts/migrating-to-traefik/).

The goal of this repo is to provide real, working*-ish* examples of full, docker compose stacks to replicate the end state of Traefik and it's associated serivces described in the blog post. 

**Please read all the documentation here and comments in each `compose.yaml` and config file as there are likely settings that need to be modified before they can be used.**

The stacks here produce:

* Two Traefik (external and instance) instances, [isolated](https://blog.foxxmd.dev/posts/migrating-to-traefik/##by-isolated-docker-network) using (overlay) docker networks
* Internal Traefik instance
  * Provisions [wildcard](https://blog.foxxmd.dev/posts/migrating-to-traefik/#wildcards) ACME SSL certs for two domains
* External Traefik instance
  * Uses [Cloudflare Tunnels for all ingress](https://blog.foxxmd.dev/posts/migrating-to-traefik/#cloudflare-tunnels-integration)
  * Has [Crowdsec integrated as a bouncer](http://blog.foxxmd.dev/posts/migrating-to-traefik/#crowdsec-integration)
  * Uses [Authentik](https://blog.foxxmd.dev/posts/migrating-to-traefik/#authentik-integration) for forward auth
* Both Instances...
  * Are configured to discovery docker services on [multiple standalone hosts/swarm nodes](http://localhost:4000/posts/migrating-to-traefik/#multi-host-docker-discovery) using [traefik-kop](https://github.com/jittering/traefik-kop)
  * Use opinionated, best-practices for [storing static and dynamic config](https://blog.foxxmd.dev/posts/migrating-to-traefik/#staticdynamic-config-best-practices)
  * Utilize [Logdy](https://logdy.dev) to [view Traefik access logs in realtime](https://blog.foxxmd.dev/posts/migrating-to-traefik/#viewing-realtime-logs)

# Prerequisties

## Networks

You will need to create two, [externally-managed](https://docs.docker.com/reference/cli/docker/network/create/) docker networks. One for each Traefik instance in order to [separate internal and external services.](https://blog.foxxmd.dev/posts/migrating-to-traefik/#separating-internalexternal-services)

If you have multiple machines running Docker and want to route traffic to all of them I would **highly recommend** setting up [Docker Swarm and using Overlay networks](https://blog.foxxmd.dev/posts/migrating-to-traefik/#swarm-and-overlay) for this (it's easy and zero cost to your existing setup!)

```shell
docker network create --driver=overlay --attachable internal_overlay
```
```shell
docker network create --driver=overlay --attachable --subnet=10.99.0.0/24 public_overlay
```

If you are not using overlay networks then replace `overlay` with `bridge`.

While not *necessary* you should also create two more [externally-managed](https://docs.docker.com/reference/cli/docker/network/create/) docker networks for use with traefik-kop and crowdsec. These make hostname resolution easier and are used in the example stacks. If you do not want to use them then environmental variables referencing hostnames using these networks can be replaced with HOST:IP and the network can be commented out where found in stacks.

```shell
docker network create --driver=overlay --internal --attachable kop_overlay
docker network create --driver=overlay --internal --attachable crowdsec_overlay
```

# Setup

If you do not plan on using a certain feature (authentik, crowdsec) make sure to comment out or remove all mentions of it.

### Placeholders

There are two placeholder sites used in the examples stacks:

* `CHANGEME.casa` represents the internal-only (LAN-accessible) domain used with [traefik_internal](/traefik_internal/)
* `CHANGEME.com` represents the public-facing domain used with [traefik_external](/traefik_external/) through Cloudflare Tunnel, Authentik, and Crowdsec

You need to Find-And-Replace all instances of these sites with your own domains.

Additionally:

* Find-And-Replace all *other* instances of `CHANGEME` with your own values
* Find-And-Replace instances of `192.168.` with your own IP:HOST

### Required Stacks

To run any of the end-user examples in [example_services](/example_services/) you will, at a minimum, need to setup

* [traefik_internal](/traefik_internal/) and/or [traefik_external](/traefik_external/)
* and an instance of [traefik_kop](/traefik_kop/) running on this host

# Usage

* View **internal** network Traefik dashboard at `https://traefik-internal.CHANGEME.casa`
* View **internal** network access logs at `https://traefik-log-internal.CHANGEME.casa`
* View **external** network Traefik dashboard at `https://traefik-external.CHANGEME.casa`
* View **external** network access logs at `https://traefik-log-external.CHANGEME.casa`

Create a service on the internal network viewable at `http://echo1.CHANGEME.casa`

```shell
docker compose -f examples_services/compose-internal-service.yaml up
```

Create a service on the external network viewable at `http://echo1.CHANGEME.com`

```shell
docker compose -f examples_services/compose-external-service.yaml up
```

Create a service on the external network, behind Authentik, viewable at `http://echo2.CHANGEME.com`

```shell
docker compose -f examples_services/compose-ext-auth-service.yaml up
```
