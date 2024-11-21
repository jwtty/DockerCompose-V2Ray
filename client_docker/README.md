# Use Docker to host Clash

1. Install docker

```bash
# https://github.com/docker/docker-install
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

> NOTE: better setup as [rootless mode](https://docs.docker.com/engine/security/rootless/)

2. Setup docker mirror (rootless mode paths)

`~/.config/docker/daemon.json`

```json
{
    "registry-mirrors": ["https://docker.m.daocloud.io", "https://dockerhub.azk8s.cn", "https://docker.mirrors.ustc.edu.cn", "https://dockerproxy.com", "https://mirror.baidubce.com", "https://docker.nju.edu.cn", "https://mirror.iscas.ac.cn"]
}
```

```bash
systemctl --user daemon-reload
systemctl --user restart docker
```

```bash
docker info
```

```
...

 Registry Mirrors:
  https://docker.m.daocloud.io/
  https://dockerhub.azk8s.cn/
  https://docker.mirrors.ustc.edu.cn/
  https://dockerproxy.com/
  https://mirror.baidubce.com/
  https://docker.nju.edu.cn/
  https://mirror.iscas.ac.cn/

...
```

3. Start Clash

NOTE: you should modify `config.yaml` with proper `proxies` settings

```bash
docker compose up --build

docker compose -f docker-compose-host-network.yaml up --build
```

```
clash-1  | time="2024-11-21T11:56:12Z" level=info msg="Start initial compatible provider Auto Select"
clash-1  | time="2024-11-21T11:56:12Z" level=info msg="Start initial compatible provider PROXY"
clash-1  | time="2024-11-21T11:56:12Z" level=info msg="Start initial compatible provider Final"
clash-1  | time="2024-11-21T11:56:12Z" level=info msg="inbound http://:7890 create success."
clash-1  | time="2024-11-21T11:56:12Z" level=info msg="inbound socks://:7891 create success."
clash-1  | time="2024-11-21T11:56:12Z" level=info msg="RESTful API listening at: 127.0.0.1:9090"
```

```bash
# Test with Google
sudo apt install w3m
w3m google.com
```

---

```bash
$ clash -h
Usage of clash:
  -d string
        set configuration directory
  -ext-ctl string
        override external controller address
  -ext-ui string
        override external ui directory
  -f string
        specify configuration file
  -secret string
        override secret for RESTful API
  -t    test configuration and exit
  -v    show current version of clash
```

- [What is Clash? | Clash Knowledge](https://en.clash.wiki/)
- [快速入手 | Clash 知识库](https://clash.wiki/configuration/getting-started.html)
