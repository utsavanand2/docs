# Cloud Native Tunnel

You can use inlets to connect HTTP and TCP services between networks securely. Through an encrypted websocket, inlets can penetrate firewalls, NAT, captive portals, and other restrictive networks lowering the barrier to entry.

VPNs traditionally require up-front configuration like subnet assignment and ports to be opened in firewalls. A tunnel with inlets can provide an easy-to-use, low-maintenance alternative to VPNs and other site-to-site networking solutions. 

You can run inlets as a stand-alone binary, in Docker, integrated into [Kubernetes](https://kubernetes.io) for [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/), or with cloud APIs. All services can be kept private in the target network, or exposed publicly.

> Inlets and [inlets PRO](https://inlets.dev/) are both available for MacOS, Linux (including ARM), Windows, FreeBSD, and as Docker containers.

## Use-cases

You can use inlets as a stand-alone networking tool, or integrate it into a platform to extend functionality for your users.

### For companies - hybrid cloud, multi-cluster and partner access

* Low-maintenance, secure, and quick alternative to a VPN
* To build a hybrid cloud between existing servers and public cloud for: CI, e2e testing, or for accessing legacy databases and IT systems
* To migrate on-premises databases and APIs to public cloud
* To connect to the private environments of your customers in a SaaS product
* For command and control of: services within private VPCs, IoT devices, and Point of Sale (PoS)
* Exposing private API endpoints to third-parties and partners
* As a cheaper, easier alternative to a data-center uplink or managed product like [AWS Direct Connect](https://aws.amazon.com/directconnect/) or [Azure Express Route](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-introduction)

### For teams and individuals - for public tunnels and Kubernetes connectivity

* As a zero-touch VPN - penetrating NAT, firewalls, captive portals and hotel WiFi
* When integrating with API that use webhooks such as Stripe, PayPal or GitHub, you can test live with your local machine
* To get a public IP address for your IngressController or services on a Kubernetes cluster
* To access your homelab, Owncloud instance, FreeNAS, or Raspberry Pi cluster using SSH and HTTPS
* As a freelancer, you can share a blog or website with a client or with your team
* As an alternative to SaaS and or proprietary offerings such as [Ngrok](https://ngrok.io) and [Argo Tunnel](https://www.cloudflare.com/en-gb/products/argo-tunnel/) where you can use your own DNS, and decide your own rate-limits

## Concept

There are two flavours of inlets tunnels depending on your needs:

* [inlets PRO](https://inlets.dev/) - L4 tunnel for any TCP traffic including built-in TLS encryption. Commercial license: support & enterprise solutions available.
* [inlets](https://github.com/inlets/inlets) - L7 tunnel for HTTP/HTTPS with OSS MIT license. TLS encryption needs to be added separately through a reverse proxy.

In the diagram we can see a developer has exposed a Node.js website on his or her laptop through the use of inlets and a server that has a public IPv4 address.

![Conceptual diagram for inlets](images/conceptual.png)

The remote server is called an "exit-node" or "exit-server" because that is where traffic from the private network appears. The user's laptop has gained a "VirtualIP" and users on the Internet can now connect to it using that IP.

## Exit-servers

An exit-server is a host that runs the `inlets server` or `inlets-pro server` command to expose its control-plane (websocket) to inlets clients. The inlets client can then connect to the exit-server and make services within its local network available through the exit-server.

Services are tunnelled through the exit-server, however unlike SaaS products such as Ngrok and Argo Tunnels, they do not need to be exposed on the public Internet. Exit-servers can also be run within Kubernetes Pods, and users who need to support many clients can use solutions like [inlets-cloud](https://inlets.dev/blog/2020/10/08/advanced-cloud-patterns.html).

Exit servers can be set up manually, or you can use tooling like [Terraform](https://www.terraform.io), bash, cloud-init or additional inlets community projects:

* [inletsctl](https://github.com/inlets/inletsctl)  - create individual exit-servers
* [inlets-operator](https://github.com/inlets/inlets-operator) - get an exit-server for each LoadBalancer service in your Kubernetes cluster

> These share the same provisioning code and create exit-servers on a range of cloud platforms like: DigitalOcean, Packet, Scaleway, Hetzner, AWS EC2, Azure, and GCP.

## Reference documentation

### inletsctl reference documentation

* [inletsctl documentation](/tools/inletsctl?id=inletsctl-reference-documentation)

### inlets-operator reference documentation

* [inlets-operator documentation](/tools/inlets-operator?id=inlets-operator-reference-documentation)

### inlets PRO reference documentation

* [inlets PRO CLI reference guide](https://github.com/inlets/inlets-pro/blob/master/docs/cli-reference.md)

## Tutorials

### inlets PRO

* [Quick-start: Tunnel a private SSH server over inlets PRO](/get-started/quickstart-tcp-ssh)
* [Quick-start: Expose a local website with HTTPS using Caddy](/get-started/quickstart-caddy)
* [Quick-start: Tunnel a private database over inlets PRO](/get-started/quickstart-tcp-database)

* [inlets PRO - reference architectures and configurations](https://github.com/inlets/inlets-pro)
* [Expose Apache Cassandra running on your local machine, out to another network with TLS encryption](https://github.com/inlets/inlets-pro/blob/master/docs/cassandra-tutorial.md)
* [Expose your private Grafana devops dashboards with Caddy and TLS](https://blog.alexellis.io/expose-grafana-dashboards/)

### inlets PRO with Kubernetes

* [Quick-start: Expose Your IngressController and get TLS from LetsEncrypt and cert-manager](/get-started/quickstart-ingresscontroller-cert-manager?id=expose-your-ingresscontroller-and-get-tls-from-letsencrypt)
* [Expose your local OpenFaaS functions to the Internet](https://inlets.dev/blog/2020/10/15/openfaas-public-endpoints.html)
* [Get kubectl access to your private cluster from anywhere](https://blog.alexellis.io/get-private-kubectl-access-anywhere/)
* [Get a private Docker registry with auth and TLS](https://blog.alexellis.io/get-a-tls-enabled-docker-registry-in-5-minutes/)

### inlets PRO Community blog posts

* [Inlets Pro Homelab Awesomeness by Matt Brewster](https://blog.brewsterops.dev/post/inlets-pro-homelab/)
* [Save Money by Connecting Your Local Database to the Public Cloud by Burton Rheutan](https://medium.com/@burtonr/local-database-for-the-cloud-with-inlets-pro-ac0488cc54e0)
* [Exploring NAT Traversal and Tunnels with Inlets and Inlets Pro by Alistair Hey](https://blog.heyal.co.uk/inlets-pro/)

### inlets OSS examples
* [Advanced Cloud Patterns with inlets and inlets-cloud](https://inlets.dev/blog/2020/10/08/advanced-cloud-patterns.html)
* [Quick-start: Expose a local HTTP server with a public IP](/get-started/quickstart-http)
* [Quick-start: Expose a Pod from your Kubernetes cluster with KinD](/get-started/quickstart-k8s-pod)
* [Video: Get public endpoints in seconds with inlets and create-react-app](https://www.youtube.com/watch?v=jrAqqe8N3q4&feature=youtu.be)

## Pricing

### inlets PRO

Buy an inlets PRO license for personal or commercial use:

* [inlets PRO pricing](https://inlets.dev/)

Try before you buy?

* [Start a free 14-day trial of inlets PRO](https://docs.google.com/forms/d/e/1FAIpQLScfNQr1o_Ctu_6vbMoTJ0xwZKZ3Hszu9C-8GJGWw1Fnebzz-g/viewform)

### inlets OSS

Perhaps the open source version of inlets is for you, feel free to reach out to us at [sales@openfaas.com](mailto:sales@openfaas.com) for a quote on commercial support, training, and custom solutions for any of the inlets projects.

Do you need to connect many clients to your inlets servers? Ask us about inlets-cloud, which can support thousands of tunnels and clients using inlets OSS or inlets PRO. inlets-cloud has its own REST API, metrics, authorization and CLI and is ideal for companies.

Alternatively, if you're a home user or a small company using inlets PRO, [please become a GitHub Sponsor](https://github.com/sponsors/alexellis).

### Buy SWAG

If you are using the project and want to support the community, then buy some SWAG to say thanks for our work.

![T-shirt and hoodie](images/inlets-swag.jpg)

**Buy your own [inlets t-shirt, hoodie or mug](https://store.openfaas.com/)**

## Connect with the community

Follow [@inletsdev](https://twitter.com/inletsdev) on Twitter.

Join the `#inlets` channel on [OpenFaaS Slack](https://slack.openfaas.io/)

[Contribute to this documentation on GitHub](https://github.com/inlets/docs/)

### Featured on

inlets is proud to be featured on the Cloud Native Landscape in the Service Proxy category.

<p><a href="https://landscape.cncf.io"><img width="200px" src="/images/cncf-landscape-left-logo.svg" alt="Cloud Native Landscape"></a></p>
