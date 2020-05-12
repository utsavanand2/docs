 # Get SSH access from anywhere

In this tutorial we will use inlets-pro to access your computer behind NAT or a firewall using SSH.

Scenario: You want to allow SSH access to a computer that doesn't have a public IP, is inside a private network or behind a firewall. A common scenario is connecting to a Raspberry Pi on a home network or a home-lab.

You will need: [a free trial license-key for inlets-pro](https://docs.google.com/forms/d/e/1FAIpQLScfNQr1o_Ctu_6vbMoTJ0xwZKZ3Hszu9C-8GJGWw1Fnebzz-g/viewform).

## Setup your exit node with `inletsctl`

You will need to have an account and API key with one of the [supported providers](https://github.com/inlets/inletsctl#featuresbacklog). For the tutorial DigitalOcean will be used.

So download your access key to a file called `do-access-token` and put it in your home directory.

You need to know the IP of the machine you want to SSH onto, in my case this was `192.168.0.35`

On each of the computers you work with in this tutorial, you can use `inletsctl` to download `inlets-pro`.

```bash
curl -sLSf https://inletsctl.inlets.dev | sudo sh
sudo inletsctl --download
```

# Create an exit node

## A) Automate your exit node

```bash
inletsctl create \
  --access-token-file ~/do-access-token \
  --remote-tcp 192.168.0.35
```

This should then create an exit node, which will forward all TCP traffic to through the client to the `remote-tcp` address that was set, in my case `192.168.0.35`.

After the machine has been created, `inletsctl` will output a sample command for the `inlets-pro client` command:

```bash 
export TCP_PORTS="8000"
export LICENSE=""
inlets-pro client --connect "wss://206.189.114.179:8123/connect" \
    --token "4NXIRZeqsiYdbZPuFeVYLLlYTpzY7ilqSdqhA0HjDld1QjG8wgfKk04JwX4i6c6F" \
    --license "$LICENSE" \
    --tcp-ports $TCP_PORTS
```

## B) Manual setup of your exit node

Use B) if you want to provision your virtual machine manually, or if you already have a host from another provider.

Log in to your remote exit node with `ssh` and obtain the binary using `inletsctl`:

```bash
curl -sLSf https://inletsctl.inlets.dev | sudo sh
sudo inletsctl --download
```

Find your public IP:

```bash
export IP=$(curl -s ifconfig.co)
```

Confirm the IP with `echo $IP` and save it, you need it for the client

Get an auth token and save it for later to use with the client

```bash
export TOKEN="$(head -c 16 /dev/urandom |shasum|cut -d'-' -f1)"

echo $TOKEN
```

Start the server:

```bash
export PROXY_TO_HERE="localhost"

sudo inlets-pro server \
  --auto-tls \
  --common-name $IP \
  --remote-tcp $PROXY_TO_HERE \
  --token $TOKEN
```

If running the inlets client on the same host as SSH, you can simply set `PROXY_TO_HERE` to `localhost`. Or if you are running SSH on a different computer to the inlets client, then you can specify a DNS entry or an IP address like `192.168.0.15`.

### Configure the internal SSH server's listening port

It's very likely (almost certain) that your exit server will already be listening for traffic on the standard ssh port `22`. Therefore you will need to configure your internal server to use an additional TCP port such as `2222`.

Once configured, you'll still be able to connect to the internal server on port 22, but to connect via the tunnel, you'll use port `2222`

Add the following to  `/etc/ssh/sshd_config`:

```bash
Port 22
Port 2222
```

For (optional) additional security, you could also disable password authentication, but make sure that you have inserted your SSH key to the internal server with `ssh-copy-id user@ip` before reloading the SSH service.

```bash
PasswordAuthentication no
```

Now need to reload the service so these changes take effect

```bash
sudo systemctl daemon-reload
sudo systemctl restart sshd
```

Check that you can still connect on the internal IP on port 22, and the new port 2222.

Use the `-p` flag to specify the SSH port:

```bash
export IP="192.168.0.35"

ssh -p 22 $IP "uptime"
ssh -p 2222 $IP "uptime"
```

### Start the inlets-pro client

First download the inlets-pro client onto the machine running ssh

```bash
curl -SLsf https://github.com/inlets/inlets-pro/releases/download/0.6.0/inlets-pro > inlets-pro
chmod +x ./inlets-pro
mv ./inlets-pro /usr/bin/inlets-pro
```

Use the command from earlier to start the client on the server:

```bash 
export TCP_PORTS="2222"
export LICENSE="LICENSE_KEY"
# export LICENSE="$(cat ~/LICENSE.txt)"

inlets-pro client --connect "wss://206.189.114.179:8123/connect" \
      --token "4NXIRZeqsiYdbZPuFeVYLLlYTpzY7ilqSdqhA0HjDld1QjG8wgfKk04JwX4i6c6F" \
      --license "$LICENSE" \
      --tcp-ports $TCP_PORTS
```

### Try it out

Verify the installation by trying to SSH to the public IP, using port `2222`.

```bash 
ssh -p 2222 user@206.189.114.179
```

You should now have access to your server via SSH over the internet with the IP of the exit server.

You can also use other compatible tools like `sftp`, `scp` and `rsync`, just make sure that you set the appropriate port flag. The port flag for sftp is `-P` rather than `-p`.

## Wrapping up

The principles in this tutorial can be adapted for other protocols that run over TCP such as MongoDB or PostgreSQL, just adapt the port number as required.

> This content was adapted from [inlets-pro documentation](https://github.com/inlets/inlets-pro/blob/master/docs/ssh-tutorial.md)
