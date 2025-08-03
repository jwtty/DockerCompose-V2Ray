# Deploy V2Ray with Docker Compose

Update from the original repo:

1. Add additional notes for trouble shooting
2. Update guidelines such as the latest usage of docker compose
3. Add more helper script to speed up and simplify deployment
4. (new) Add a simpler config of just V2Ray + WS (without DNS settings i.e. connect with IP directly)

## Steps

### 1. Setup VPS

> Using Azure as an example

1. Create a Virtual Machine
   1. Any kind of Size (schema) will work, but you can use exactly the same as mine
      1. Size: Standard B2s (2 vcpus, 4 GiB memory)
      2. Operating system: Linux (ubuntu 20.04)
      3. Location: Japan East
      4. Disk: Standard SSD
   2. Make sure you disable the auto-shutdown scheduling
2. Once the VM creation complete
   1. Open necessary ports, in this case you need to open 80, 443
   2. Create a DNS name: such as `your-dns-name` (optional: only required by TLS)
      1. You can use a dynamic IP address
      2. You will be able to connect to your machine by using `your-dns-name.japaneast.cloudapp.azure.com` depending on the machine's location

> For GCP you might facing these steps
>
> - [Add SSH keys to VMs  |  Compute Engine Documentation  |  Google Cloud](https://cloud.google.com/compute/docs/connect/add-ssh-keys)
> - [Create a VM instance with a custom hostname  |  Compute Engine Documentation  |  Google Cloud](https://cloud.google.com/compute/docs/instances/custom-hostname-vm)
> - [Add, modify, and delete records  |  Cloud DNS  |  Google Cloud](https://cloud.google.com/dns/docs/records)
> - [DNS Propagation Checker - Global DNS Testing Tool](https://www.whatsmydns.net/)

### 2. Setup Environment

> `ssh` into your machine

1. Install docker with `install_docker.sh`

> Legacy Steps
>
> 1. Install `docker`
>    1. Download the auto setup script and run: `curl -fsSL https://get.docker.com -o get-docker.sh` then `sh get-docker.sh`
>    2. Add user to the docker user group (so you don't need `sudo` to use `docker`): `sudo gpasswd -a $USER docker`
>    3. Make docker auto start on boot: `sudo systemctl start docker` then `sudo systemctl enable docker`
> 2. ~~Install `docker compose`~~ ([Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/)) (Currently docker compose is a built-in)
>    1. `DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}`
>    2. `mkdir -p $DOCKER_CONFIG/cli-plugins`
>    3. `curl -SL https://github.com/docker/compose/releases/download/v2.4.1/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose`
>    4. `chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose`

### 3. Clone Code and Config

Provide two ways of configuration

1. WS + TLS (will create certification for TLS connection)
2. WS Only

Common steps

1. Download some CLI tools
   1. `sudo apt update`
   2. `sudo apt install git vim tmux`
2. `git clone https://github.com/daviddwlee84/DockerCompose-V2Ray.git` and `cd DockerCompose-V2Ray`

#### 3-A. WS + TLS (will create certification for TLS connection)

> - Pros
>   - This will create certification for TLS connection
>   - Your V2Ray server looks like normal website if you directly access it
> - Cons
>   - You need to solve your DNS to make it reachable


1. Modify settings
   1. Modify `your_domain` and `your_email_address` in `initial_https.sh` and execute (if you forgot this step you will have to manually modify the files, or just `git reset --hard` to revert the changes and try again)
   2. `docker-compose.yml`
      1. No need to modify
   3. `data/v2ray/config.json`
      1. Change id to use your own `"id": "bae399d4-13a4-46a3-b144-4af2c0004c2e"` (or you can leave it as what it is)
      2. You can generate a new UUID using this online tool: [Online UUID Generator Tool](https://www.uuidgenerator.net/) (not sure what is the difference between different version UUIDs, seems not all UUID-like strings will work)
2. Start server
   1. `tmux`
   2. `docker compose up --build` (permission issue just add `sudo` in the front)
   3. Exit you can use `Ctrl + b` then `d` to detach tmux and type `exit` to close the terminal

> Legacy Steps
>
> 1. `init-letsencrypt.sh`
>    1. Modify `domains` and `email` to be your own
>    2. If you are using Azure, the `domains` is `your-dns-name.japaneast.cloudapp.azure.com` (DNS name).
>    3. Note that `domains` SHOULD BE AN ARRAY, that is you should keep the parenthesis there.
> 2. `data/nginx/conf.d/v2ray.conf`
>    1. Modify all `your_domain`
>    2. You can use vim `:%s/your_domain/your-dns-name.japaneast.cloudapp.azure.com/g`
> 3. Setup Nginx and HTTPS encryption stuff
>    1. `chmod +x ./init-letsencrypt.sh`
>    2. `./init-letsencrypt.sh`
>       * if you can't use this, might because docker need `sudo` permission
>       * must make sure the ports (firewall) are opened

#### 3-B. WS Only

> - Pros
>   - Simple, just start the docker and that's it
> - Cons
>   - Your IP might get banned by GFW more easily (in theory). But you can just switch to new public IP at anytime.

1. Directly start server with V2Ray only: `docker compose -f docker-compose-v2ray-only.yml up -d`

### 4. Config Your Client

#### 4-1. Shadowrocket (iOS client)

Type: Vmess

1. Address: `your-dns-name.japaneast.cloudapp.azure.com`
2. Port: `443`
3. UUID: `bae399d4-13a4-46a3-b144-4af2c0004c2e`
4. Alert ID: 64
5. Method: auto
6. TLS
   1. enable
   2. allow insecure
7. Transport: `websocket`
   1. Path: `/v2ray`

> NOTE
> 1. the `2. Port` should be `80` if you use `3-B`.
> 2. the `6. TLS` is needed only if you follow `3-A`; if you use `3-B` you can skip it.

#### 4-2. Clash for Windows (PC client)

> NOTE: different client might use different config format, even though it look similar at a glance (using yaml...)

**Clash for Windows**:

> NOTE: Clash for Windows can be used in Windows, MacOS, and Ubuntu...

- [Clash for Windows Download – Clash for Windows](https://www.clashforwindows.net/clash-for-windows-download/)
- [Releases · clashdownload/Clash_for_Windows](https://github.com/clashdownload/Clash_for_Windows/releases)
- [Releases · lantongxue/clash_for_windows_pkg](https://github.com/lantongxue/clash_for_windows_pkg/releases)
- ~~[Releases · Fndroid/clash_for_windows_pkg](https://github.com/Fndroid/clash_for_windows_pkg/releases)~~

```yaml
# ...

# 3-A.
proxies:
  - name: "Your Customized Name"
    type: vmess
    server: your-dns-name.japaneast.cloudapp.azure.com
    port: 443
    uuid: bae399d4-13a4-46a3-b144-4af2c0004c2e
    alterId: 64
    cipher: auto
    udp: false
    tls: true
    skip-cert-verify: true
    network: ws
    ws-opts:
      path: /v2ray

# ...
```

```yaml
# ...

# 3-B.
proxies:
  - name: "Your Customized Name"
    type: vmess
    server: your-ip-address
    port: 80
    uuid: bae399d4-13a4-46a3-b144-4af2c0004c2e
    alterId: 64
    cipher: auto
    udp: false
    network: ws
    ws-opts:
      path: /v2ray

# ...
```

[example yaml config](example/clash_for_windows.yml)

#### 4-3. Clash Core (CLI Client)

- [Releases · Kuingsmile/clash-core](https://github.com/Kuingsmile/clash-core/releases)

1. Simply download binary for your machine and run
2. Modify `~/.config/clash/config.yaml` => can consume Clash for Windows config!
3. Setup proxy `export http_proxy=http://127.0.0.1:7890 https_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890`
4. (Optional) Make this more "permanent":  [How to Configure Proxy Settings on Ubuntu](https://phoenixnap.com/kb/ubuntu-proxy-settings)
   - make clash a system service (`systemctl`)
   - put environment variable (3.) setting in your `~/.bashrc`

Setting up Proxy for wget

```ini
# ~/.wgetrc
use_proxy = on
http_proxy = "http://[proxy_address]:[port_number]/"
https_proxy = "https://[proxy_address]:[port_number]/"
ftp_proxy = "ftp://[proxy_address]:[port_number]/"
```

Setting up Proxy for git

```bash
# 1. Use the following commands in the terminal to configure the proxy server:
git config --global http.proxy http://[proxy_address]:[port_number]
git config --global https.proxy https://[proxy_address]:[port_number]

# 2. Check if the settings are applied:
git config --global --get http.proxy
git config --global --get https.proxy
```

Setting up Proxy for APT

```conf
# /etc/apt/apt.conf
Acquire::http::Proxy "http://[proxy_address]:[port_number]/";
Acquire::https::Proxy "https://[proxy_address]:[port_number]/";
```

How to Check Whether Ubuntu Proxy Works

```bash
# Environment variables
echo $http_proxy
echo $https_proxy

# Connectivity
curl -I http://example.com

# Wget
wget http://www.example.com

# Git
git ls-remote [remote_name_or_URL]

# Apt
sudo apt update
```

## Trouble Shooting

You can see `logs/` folder

Use `sudo tail -f ./path/to/log.log` to see the error message then debug

1. If you forgot to open `80,443` ports, you will fail at certbot step. Remove `data/certbot` folder and try again.
2. If your Nginx server successfully running, you can connect to `https://your-dns-name.japaneast.cloudapp.azure.com` using a browser and see "Congratulation!" which basically is this HTML ([`data/nginx/html/v2ray/index.html`](data/nginx/html/v2ray/index.html))
3. You can connect to `https://your-dns-name.japaneast.cloudapp.azure.com/v2ray`
   1. If you get a 502 error, that means your V2Ray server is not running correctly.
   2. If you get the text "bad request", that means it is successfully running.
4. If you changed UUID and failed to connect but every other thing is fine (v2ray log can see traffic income), maybe change UUID back to the default value.
5. If you are using rootless docker you might found issue of binding ports < 1024. `Error response from daemon: driver failed programming external connectivity on endpoint nginx (xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx): failed to bind port 0.0.0.0:80/tcp: Error starting userland proxy: error while calling PortManager.AddPort(): cannot expose privileged port 80, you can add 'net.ipv4.ip_unprivileged_port_start=80' to /etc/sysctl.conf (currently 1024), or set CAP_NET_BIND_SERVICE on rootlesskit binary, or choose a larger port number (>= 1024): listen tcp4 0.0.0.0:80: bind: permission denied`
   1. [Run the Docker daemon as a non-root user (Rootless mode) | Docker Docs](https://docs.docker.com/engine/security/rootless/#exposing-privileged-ports): `sudo setcap cap_net_bind_service=ep $(which rootlesskit)` then `systemctl --user restart docker`.
   2. Check your 80 ports is working `docker run -it -p 80:80 nginx` and open your IP in a browser.
6. To test you client you can use [What Is My IP Address - See Your Public Address - IPv4 & IPv6](https://whatismyipaddress.com/) to see if the IP is changed to the server IP
