## Quick Start

[简体中文](./docs/README_zh-Hans.md) | [繁體中文](./docs/README_zh-Hant.md) | [日本語](./docs/README_ja.md)

Using one command to start the [Syncplay](https://syncplay.pl/) service. Yes, that's it.

```bash
$ docker run --rm --net=host dnomd343/syncplay
Welcome to Syncplay server, ver. 1.7.4
```

> Pressing `Ctrl+C` will exit the service.

<details>

<summary><b>Unable to access Docker Hub?</b></summary>

<br/>

If you cannot access the Internet, you need to obtain the OCI image and copy it into storage medium. For details, see [offline usage](#Registry).

If you are located in China Mainland that cannot access the Docker Hub normally, you can replace the `dnomd343/syncplay` as `ccr.ccs.tencentyun.com/dnomd343/syncplay` , which will access the TCR service at Guangzhou.

</details>

If there are no accidents, you can fill in the server IP or domain name on the client for verification, the default port is `tcp/8999` . If you can't connect, please check your firewall settings.

In order to run the service better, we can use the following command to make Syncplay running in the background.

```bash
$ docker run -d --net=host \
    --restart=always --name=syncplay dnomd343/syncplay
```

> You can use `docker ps -a` to see the running service, and using `docker rm -f syncplay` to stop the service.

You can add more arguments to achieve customization. For example, we require a password when connecting to the server, prohibit chat, and display a welcome message after entering the room. Use the following commands.

> Note that before pressing Enter, you must execute `docker rm -f syncplay` to remove the original services, otherwise they will conflict.

```bash
$ docker run -d --net=host \
    --restart=always --name=syncplay dnomd343/syncplay \
    --disable-chat --motd='Hello' --password='PASSWD'
```

Sometimes we need to restart the server, it is necessary to persist Syncplay at this time, which means that the room data will be saved to disk. You need to choose a working directory to save them, such as `/etc/syncplay/` , execute the following command, the data will be saved to the `rooms.db` file.

```bash
$ docker run -d --net=host         \
    --volume /etc/syncplay/:/data/ \
    --restart=always --name=syncplay dnomd343/syncplay \
    --persistent --motd='Persistent Server'
```

This directory has more uses. For example, adding the `--enable-stats` option will enable the statistics function, and the data will be saved to the file `stats.db` in the directory. You can also create a `config.yml` file in the directory and write the configuration options in it, Syncplay will automatically read it when it starts, avoiding the need to type a large number of arguments in the command line.

```yaml
# /etc/syncplay/config.yml
password: 'My Password'
persistent: true
enable-stats: true
disable-chat: false
motd: |
  Hello, here is a syncplay server.
  More information...
```

When deploying, it's always a good idea to turn on TLS (of course it's not necessary and this step can be skipped), and luckily Syncplay makes it easy to do this. Before starting, you need to prepare a domain name and resolve its DNS to the current server. At the same time, we must have its private key and certificate file.

Application for a certificate can be made through [`acme.sh`](https://acme.sh/) , [`certbot`](https://certbot.eff.org/) or other reasonable methods. Anyway, you will end up with a private key and certificate file, and Syncplay requires you to provide the following three files.

+ `cert.pem` : The certificate issued by the CA.
+ `chain.pem` : The certificate chain of CA service.
+ `privkey.pem` : The private key of the certificate.

For example, in `acme.sh` , you can execute the command like this to save the certificate configuration of the domain name `343.re` to the `/etc/ssl/certs/343.re/` directory.

```bash
$ acme.sh --install-cert -d 343.re               \
    --cert-file  /etc/ssl/certs/343.re/cert.pem  \
    --ca-file    /etc/ssl/certs/343.re/chain.pem \
    --key-file   /etc/ssl/certs/343.re/privkey.pem
```

Now that we are ready, we just need to execute the following command and a more secure and private Syncplay service will be started.

```bash
$ docker run -d --net=host                  \
    --volume /etc/syncplay/:/data/          \
    --volume /etc/ssl/certs/343.re/:/certs/ \
    --restart=always --name=syncplay dnomd343/syncplay \
    --enable-tls --motd='Secure Server'
```

> Note that the client's server address must match the certificate, otherwise the connection will fail.

It should be noted that unlike some services, Syncplay does not need to be manually restarted when the certificate is updated. It will automatically detect certificate changes and use the latest version. In addition, TLS on the Syncplay server is adaptive, which means that even older versions of clients that do not support TLS can still communicate normally, but note that security encryption will no longer take effect at this time.

## Command-line Arguments

You can customize the Syncplay server by specifying the following command line arguments.

> The following parameters are adjusted for docker and are not exactly the same as [official documentation](https://man.archlinux.org/man/extra/syncplay/syncplay-server.1). Please refer to this when using.

+ `--config [FILE]` : Specify the configuration file, the default is `config.yml` .

+ `--port [PORT]` : Listening port of Syncplay service, the default is `8999` .

+ `--password [PASSWD]` : Authentication when connecting to the server, not enabled by default.

+ `--motd [MESSAGE]` : The welcome text after the user enters the room, not enabled by default.

+ `--salt [TEXT]` : Specify a random string as the [salt value](https://en.wikipedia.org/wiki/Salt_(cryptography)) used to secure password, defaults to empty.

+ `--random-salt` : Using randomly generated salt value, valid when `--salt` is not specified, not enabled by default.

+ `--isolate-rooms` : Enable room isolation, users cannot see information from anyone outside their room, not enabled by default.

+ `--disable-chat` : Disable the chat feature, not enabled by default.

+ `--disable-ready` : Disable the readiness indicator feature, not enabled by default.

+ `--enable-stats` : Enable the server statistics feature, the data will be saved in the `stats.db` file, not enabled by default.

+ `--enable-tls` : Enable TLS support, the files need to be mounted in the `/certs/` directory, including `cert.pem` , `chain.pem` and `privkey.pem` , not enabled by default.

+ `--persistent` : Enable room data persistence, the information will be saved to the `rooms.db` file, only valid when `--isolate-rooms` is not specified, not enabled by default.

+ `--max-username [NUM]` : Maximum length of usernames, default is `16` .

+ `--max-chat-message [NUM]` : Maximum length of chat messages, default is `150` .

+ `--permanent-rooms [ROOM ...]` : Specifies a list of rooms that will still be listed even if their playlist is empty, only valid when `--persistent` is specified, defaults to empty.

+ `--listen-ipv4 [ADDR]` : Listening address of Syncplay service on IPv4 network, not enabled by default.

+ `--listen-ipv6 [ADDR]` : Listening address of Syncplay service on IPv6 network, not enabled by default.

> When you specify only `--listen-ipv4` , Syncplay will not listen on IPv6 and vice versa. When both are specified, Syncplay will work under dual-stack networking.

Add `--version` option to print Syncplay and Python versions, as well as CPU architecture.

```bash
$ docker run --rm dnomd343/syncplay --version
Syncplay Docker Bootstrap v1.7.4 (Yoitsu 115) [CPython 3.12.11 aarch64]
```

You can also use the following command to output help information.

<details>

<summary><b>Help message of command-line</b></summary>

<br/>

```bash
$ docker run --rm dnomd343/syncplay --help
usage: syncplay [-h] [-v] [-c FILE] [-p PORT] [-k PASSWD] [-m MESSAGE]
                [--salt TEXT] [--random-salt] [--isolate-rooms]
                [--disable-chat] [--disable-ready] [--enable-stats]
                [--enable-tls] [--persistent] [--max-username NUM]
                [--max-chat-message NUM] [--permanent-rooms [ROOM ...]]
                [--listen-ipv4 ADDR] [--listen-ipv6 ADDR]

Syncplay Docker Bootstrap

options:
  -h, --help            Show this help message and exit.
  -v, --version         Show version information and exit.
  -c FILE, --config FILE
                        Specify the configuration file path, the default is
                        `config.yml`.
  -p PORT, --port PORT  Listening port of Syncplay service, the default is
                        8999.
  -k PASSWD, --password PASSWD
                        Authentication when connecting to the server.
  -m MESSAGE, --motd MESSAGE
                        The welcome text after the user enters the room.
  --salt TEXT           A string used to secure passwords, defaults to empty.
  --random-salt         Use a randomly generated salt value, valid when
                        `--salt` is not specified.
  --isolate-rooms       Enable room isolation, users cannot see information
                        from anyone outside their room.
  --disable-chat        Disables the chat feature.
  --disable-ready       Disables the readiness indicator feature.
  --enable-stats        Enable the server statistics feature, the data will be
                        saved in the `stats.db` file.
  --enable-tls          Enable TLS support, the private key and certificate
                        needs to be mounted in the `/certs/` directory.
  --persistent          Enable room data persistence, the information will be
                        saved to the `rooms.db` file, only valid when
                        `--isolate-rooms` is not specified.
  --max-username NUM    Maximum length of usernames, default is 16.
  --max-chat-message NUM
                        Maximum length of chat messages, default is 150.
  --permanent-rooms [ROOM ...]
                        Specifies a list of rooms that will still be listed
                        even if their playlist is empty, only valid when
                        `--persistent` is specified, defaults to empty.
  --listen-ipv4 ADDR    Listening address of Syncplay service on IPv4.
  --listen-ipv6 ADDR    Listening address of Syncplay service on IPv6.
```

</details>

## Configure File

If you configure a lot of options, it will be quite troublesome and error-prone to enter a large number of command line arguments every time you start. At this time, you can write them into the configuration file.

Creating `config.yml` file in the working directory, it uses YAML format and supports all arguments in the command line. Syncplay will automatically read and load it when starting, but it should be noted that if the same arguments are specified on the command line, will override the configuration file's options.

```yaml
port: 7999
salt: 'SALT'
random-salt: true
password: 'My Password'
persistent: true
enable-tls: true
enable-stats: true
isolate-rooms: true
disable-chat: true
disable-ready: true
max-username: 256
max-chat-message: 2048
listen-ipv4: 127.0.0.1
listen-ipv6: ::1
permanent-rooms:
  - ROOM_1
  - ROOM_2
  - ROOM_3
motd: |
  Hello, here is a syncplay server.
  More information...
```

You can also use JSON or TOML formats, relying on the suffix to identify them. The default file name `config.yml` can be obtained by adding the `--config` parameter or passing the `CONFIG` environment variable.

## Environment Variables

The Syncplay container also supports configuration through environment variables. It supports three types of fields: numbers, strings, and boolean, this means that `permanent-rooms` is not supported. Environment variables are named in uppercase letters, and `-` is replaced by `_` , boolean values are represented by `ON` or `TRUE`. The following is an example of using environment variables.

```bash
$ docker run -d --net=host \
    --env PORT=7999        \
    --env MOTD=Hello       \
    --env DISABLE_CHAT=ON  \
    --restart=always --name=syncplay dnomd343/syncplay
```

You may have noticed that we support three configuration methods: command line arguments, configuration file and environment variables. Their priority is from high to low, that is, the command line arguments will override the options of the configuration file, and the configuration file will override the environment variables. You can use them together.

## Docker Compose

Using `docker compose` to deploy Syncplay is a more elegant way. You need to create a `docker-compose.yml` configuration file and write the following example.

```yaml
# /etc/syncplay/docker-compose.yml
services:
  syncplay:
    container_name: syncplay
    image: dnomd343/syncplay
    network_mode: host
    restart: always
    volumes:
      - ./:/data/
```

We save this file in the `/etc/syncplay/` directory. Since a relative path is used, it is also in the working directory. Execute the command in this directory to start the Syncplay service.

```bash
$ docker compose up -d
[+] Running 1/1
✔ Container syncplay Started
```

> Adding the `-d` option allows the service to run in the background.

Similarly, you can map the certificate directory to enable TLS functionality, and edit the `config.yml` file to configure more options.

## Security

In the above commands, we use `--net=host` to expose external services, which means that the container can directly access the host network. From a security perspective, it is recommended to use the bridge network to map the `tcp/8999` port, although it may result in a slight performance loss.

```bash
$ docker run -d -p 8999:8999 \
    --restart=always --name=syncplay dnomd343/syncplay
```

By default, Docker runs containers as the root user, which can pose a security risk. The images built by this project complies with the OCI standard, so [Podman](https://podman.io/) can be used completely instead of Docker, which runs in non-root mode by default.

```bash
$ podman run -d -p 8999:8999 \
    --restart=always --name=syncplay dnomd343/syncplay
```

Of course, we can also use Docker [rootless mode](https://docs.docker.com/engine/security/rootless/), but it is quite cumbersome to configure. If you only want to use Docker, you can specify the `UID` and `GID` when building the image, and the container will not have root permissions.

```bash
# You can view the current non-root UID and GID value.
$ id
uid=1000(dnomd343) gid=1000(dnomd343) ...

# Use the obtained UID and GID values as build arguments.
$ docker build -t my-syncplay \
    --build-arg USER_UID=1000 \
    --build-arg USER_GID=1000 \
    https://github.com/dnomd343/syncplay-docker.git

$ docker run -d -p 8999:8999 \
    --restart=always --name=syncplay my-syncplay
```

## Registry

The images released by this project comply with the [OCI Image Format Specification](https://github.com/opencontainers/image-spec) and can be distributed on any registry that complies with the [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec). In the current workflow, Github Action will be automatically distribute the same images to the following registries:

- Docker Hub: `dnomd343/syncplay`
- Github Package: `ghcr.io/dnomd343/syncplay`
- Tencent Cloud: `ccr.ccs.tencentyun.com/dnomd343/syncplay`

There are four CPU architectures supported, namely `amd64` , `arm64` , `i386` and `arm/v7` . When pulling the image, the container tool will automatically select the appropriate image based on the host architecture.

You can pull the original OCI image and store it as tar file, which can be used offline. It is recommended to use the [skopeo](https://github.com/containers/skopeo) tool to achieve this.

> You can use the `docker save` command to export, but only supports a single architecture.

```bash
$ skopeo copy --all                             \
    docker://docker.io/dnomd343/syncplay:v1.7.4 \
    oci-archive:syncplay-v1.7.4.tar

$ skopeo copy --override-os=linux --override-arch=arm64 \
    docker://docker.io/dnomd343/syncplay:v1.7.4         \
    oci-archive:syncplay-v1.7.4-arm64.tar

$ docker load < syncplay-v1.7.4.tar
```

## Troubleshooting

If you encounter any errors, please first use the `docker logs syncplay` command to print the process output. It may contain useful error information. You can also output more detailed logs by specifying the environment variable `DEBUG=ON` .

```bash
$ docker run --rm --env DEBUG=ON dnomd343/syncplay
ENV_OPTS -> ...
CFG_OPTS -> ...
ARG_OPTS -> ...
Environment variables -> ...
Configure content -> ...
Environment options -> ...
Command line options -> ...
Configure file options -> ...
Bootstrap final options -> {}
Syncplay startup arguments -> ['syncplay', '--port', '8999', '--salt', '']
Welcome to Syncplay server, ver. 1.7.4
```

## Advanced

For some reason, you may need to change the path of the configuration files or working directory. This is possible in the Syncplay container, which requires you to specify it using environment variables.

+ `TEMP_DIR` ：Temporary directory, it does not need to be persisted, defaults to `/tmp/` .

+ `WORK_DIR` ：The working directory, which stores data related to Syncplay, defaults to `/data/` .

+ `CERT_DIR` ：Certificate directory, which is used to store TLS certificates and private key files, defaults to `/certs/` .

## Build Image

> This project uses several [BuildKit](https://github.com/moby/buildkit) features (bundled after Docker 23.0), and other builders may have compatibility issues.

You can build an image directly from the source code using the following command.

```bash
$ docker build -t syncplay https://github.com/dnomd343/syncplay-docker.git
```

You can also change the source code to implement your own customizations.

```bash
$ git clone https://github.com/dnomd343/syncplay-docker.git
$ cd syncplay-docker/
# some edit...
$ docker build -t syncplay .
```

If you need images for multiple architectures, please use the `buildx` command to build.

```bash
$ docker buildx build -t dnomd343/syncplay                    \
    --platform=linux/amd64,linux/386,linux/arm64,linux/arm/v7 \
    https://github.com/dnomd343/syncplay-docker.git --push
```

## License

MIT ©2023 [@dnomd343](https://github.com/dnomd343)
