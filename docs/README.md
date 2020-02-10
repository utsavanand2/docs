# inlets

Inlets is a cloud native tunnel.

There are two options available for your tunnels:

* inlets - OSS L7 tunnel
* inlets-pro - commercial TCP L4 tunnel

You can use inlets/-pro to create a secure tunnel between two networks. The main use-case for inlets is to expose a private API or service within another network such as the Internet. The remote server is called an "exit-node" or "exit-server" because that is where traffic from the private network appears.

You can then configure inlets manually, or use one of the community-built tools to automate creating an exit-server for you.

* inletsctl - create exit-servers for inlets/-pro
* inlets-operator - Kubernetes automation to create exit-servers for inlets/-pro
