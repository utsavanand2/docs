## Expose a HTTP server with a public IP

The easiest way to get started is with `inletsctl` which will create a public host for us on DigitalOcean and then configure the inlets-server automatically.

```bash
# Remove `sudo to install to the local folder
curl -sLSf https://raw.githubusercontent.com/inlets/inletsctl/master/get.sh | sudo sh

sudo inletsctl download [--pro]
```

Start a Python HTTP fileserver (on port 8000)

```bash
export STORE=/tmp/store/

mkdir -p $STORE
cd $STORE
echo "Hi" > hello-inlets.txt
python -m SimpleHTTPServer 8000
```

Now create an exit-server using inletsctl to automate the creation of the public host and the installation of the inlets server component:

```bash
inletsctl create --provider digitalocean \
  --region lon1 \
  --access-token-file ~/digitalocean-api
```

Run the command given for `inlets client` to connect to the server.

```bash
export UPSTREAM=http://127.0.0.1:8000
inlets client --remote "ws://142.93.33.12:8080" \
  --token "GhdoqT9NfZWbSlW0jHEQHcBPpC9Krv9xPxURdGrM9R3yXOwl7o6H2Pam3TnkMGNx" \
  --upstream $UPSTREAM
```

You can now access the Python HTTP server by going to the IP address given, i.e. `http://142.93.33.12`.

![Python example server](../images/python-example-server.png)

Finally feel free to run `inletsctl delete` with the command given to you.

```bash
inletsctl delete --provider digitalocean --id "179755668"
```

In this tutorial we automated the cloud host using inletsctl, it provisioned a server using an API token from DigitalOcean and then we started the tunnel manually.

If you would like to add TLS encryption to the tunnel then you can do this manually using a tutorial by Alex Ellis:

[HTTPS for your local endpoints with inlets and Caddy](https://blog.alexellis.io/https-inlets-local-endpoints/)

The easier alernative is to use [inlets-pro](https://github.com/inlets/inlets-pro) which has built-in TLS and can allow your computer to obtain a TLS certificate using a reverse proxy like Caddy, Nginx (with cert-bot) or Traefik.
