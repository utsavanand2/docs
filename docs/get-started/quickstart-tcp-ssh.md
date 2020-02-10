# Get SSH access from anywhere

In this tutorial we will use inlets-pro to access your computer behind NAT or a firewall using SSH.

Scenario: You want to allow SSH access to a computer that doesn't have a public IP, is inside a private network or behind a firewall. A common scenario is connecting to a Raspberry Pi on a home network or a home-lab.

## Setup your exit node with `inletsctl`

You will need to have an account and API key with one of the [supported providers](https://github.com/inlets/inletsctl#featuresbacklog), 
we are using Digital Ocean in this example. So download your access key to a file called `do-access-token` and put it in
your home directory.

We need to know the IP of the machine you want to SSH onto, in my case this was 192.168.0.99

On each of the computers you work with in this tutorial, you can use `inletsctl` to download `inlets-pro`.

```sh
curl -sLSf https://inletsctl.inlets.dev | sudo sh
sudo inlets-pro --download
```

# Create an exit node

## A) Automate your exit node

```
inletsctl create --access-token-file ~/do-access-token --remote-tcp 192.168.0.99
```

This should then create an exit node, which will forward all TCP traffic to through the client to the `remote-tcp` address
that we set, in my case `192.168.0.99`

After the machine has been created, inletsctl will output the following:

```sh 
export TCP_PORTS="8000"
export LICENSE=""
inlets-pro client --connect "wss://206.189.114.179:8123/connect" \
    --token "4NXIRZeqsiYdbZPuFeVYLLlYTpzY7ilqSdqhA0HjDld1QjG8wgfKk04JwX4i6c6F" \
    --license "$LICENSE" \
    --tcp-ports $TCP_PORTS
```

## B) Manual setup of exit node

Use B) if you want to provision your virtual machine manually, or if you already have a host from another provider.

Log in with ssh and obtain the binary:

```sh
curl -SLsf https://github.com/inlets/inlets-pro/releases/download/0.5.1/inlets-pro > inlets-pro
chmod +x ./inlets-pro
mv ./inlets-pro /usr/bin/inlets-pro
```

Find your public IP:

```
export IP=$(curl -s ifconfig.co)
```

Confirm the IP with `echo $IP` and save it, you need it for the client

Get an auth token and save it for later to use with the client

```sh
export TOKEN="$(head -c 16 /dev/urandom |shasum|cut -d'-' -f1)"

echo $TOKEN
```

Start the server:

```sh
export PROXY_TO_HERE="localhost"

sudo inlets-pro server \
  --auto-tls \
  --common-name $IP \
  --remote-tcp $PROXY_TO_HERE \
  --token $TOKEN
```

If running the inlets client on the same host as SSH, you can simply set `PROXY_TO_HERE` to `localhost`. Or if you are running SSH on a different computer to the inlets client, then you can specify a DNS entry or an IP address like `192.168.0.15`.

### Configure the ssh agent

Our exit-server is already using port 22 for SSH access. This means we need to configure our machine to listen for ssh connections on another port, let's use 2222 which is a standard alternative:

If we find the line in  `/etc/ssh/sshd_config` near the top that is `Port 22`, make sure it's un-commented
and add `Port 2222`

```
Port 22
Port 2222
```

It is recommend to disable password authentication, and instead rely on SSH Keys only.

> WARNING: make sure you have validated that SSH works using your SSH Key before disabling password authentication.
> If you don't check you may be unable to access your machine. You can use something like [ssh-copy-id](http://manpages.ubuntu.com/manpages/precise/man1/ssh-copy-id.1.html) 
> to help

set this line to `no`, it may also be commented out
```
PasswordAuthentication no
```

We now need to reload the service so these changes take effect

```
sudo systemctl reload sshd
```

### Start the inlets-pro client

First we need to download the inlets-pro client onto our machine running ssh

```sh
curl -SLsf https://github.com/inlets/inlets-pro/releases/download/0.5.1/inlets-pro > inlets-pro
chmod +x ./inlets-pro
mv ./inlets-pro /usr/bin/inlets-pro
```

Use the command we saw earlier to start the client on our server,

```sh 
  export TCP_PORTS="2222"
  export LICENSE="<your lisence here>"
  inlets-pro client --connect "wss://206.189.114.179:8123/connect" \
        --token "4NXIRZeqsiYdbZPuFeVYLLlYTpzY7ilqSdqhA0HjDld1QjG8wgfKk04JwX4i6c6F" \
        --license "$LICENSE" \
        --tcp-ports $TCP_PORTS
```

### Try it out

Verify the installation by trying to SSH to the public IP, using port `2222`. You can run this from anywhere, using your phone or iPad could provide a good test.

```sh 
ssh -p 2222 user@206.189.114.179
```

You should now have access to your server via SSH for remote access, `scp` and additional port-forwarding via `ssh -L`.

## Wrapping up

The principles in this tutorial can be adapted for other protocols that run over TCP such as MongoDB or PostgreSQL, just adapt the port number as required.

> This content was adapted from [inlets-pro documentation](https://github.com/inlets/inlets-pro/blob/master/docs/ssh-tutorial.md)
