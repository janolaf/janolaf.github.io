---
title: 'Quadlet and Mealie'
date: 2024-03-08T23:00:22-06:00
draft: false
toc: false
categories: [ "podman" ]
tags: [ "podman", "quadlet", ]
---

Podman has introduced quadlets which are a new way to manage containers using systemd. In simple terms, you use systemd syntax to managed containers by systemd.

Quadlet can manage a container with `foo.container`, volumes with `foo.volumes`, podman networks with `foo.network`, a kubernetes yaml file with `foo.kube`.

Since Mealie has moved everything back to a single container, it is a perfect time to switch to quadlet to manage my container. 

## The Old Way

In the past, you would create the container and then generate the systemd service file.

``` bash
podman container create \
    --name mealie\
    --label “io.containers.autoupdate=registry” \
    -p 10000:9000 \
    -v mealie-data:/app/data:z \
    -e ALLOW_SIGNUP=true \
    -e WEB_CONCURRENCY=1 \
    -e BASE_URL=https://food.yourdomain.local \
    -tz local \
    ghcr.io/mealie-recipes/mealie:nightly

```
You would then generate the systemd service file:

``` bash
podman generate systemd --name mealie --file ~/.config/systemd/user/mealie.service
```

Reload systemd:

``` bash
systemctl --user deamon-reload
```

Start the service:

``` bash
systemctl --user enable --now mealie.service
```
 Any time you made changes to the container, you would also have to regenerating the systemd service file.


## Quadlet Way

Quadlet replaces the old method of generating systemd.

For a single container, you create a `mealie.container` file in `~/.config/container/systemd`.

``` systemd
[Container]
ContainerName=mealie
Image=ghcr.io/mealie-recipes/mealie:nightly
AutoUpdate=registry

PublishPort=10000:9000

Volume=mealie-data:/app/data:z

Environment=ALLOW_SIGNUP=true
Environment=MAX_WORKERS=1
Environment=WEB_CONCURRENCY=1
Environment=BASE_URL=https://food.yourdomain.local

Timezone=local

[Install]
WantedBy=multi-user.target default.target

```
{{< alert >}}
Note: Your container will not start on boot if it does not have `[Installed]` and a `.target` like `default.target`

You can see more options at [podman-systemd](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)
{{< /alert >}}

After run:
``` bash
systemctl --user daemon-reload
```

This will generate the systemd service file for you. Every time you make a change to mealie.container, you can regenerate the service file.

You can then start the service file like normal

``` bash
systemctl --user start mealie.service
```
 Quadlets makes it easier to automate container management with systemd. 

## Quadelt Resources 

* [podman-systemd](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)
* [Quadlets or: How I learned to stop worrying and love dot containers](https://www.linkedin.com/pulse/quadlets-how-i-learned-stop-worrying-love-dot-adam-clater/)
* [Make systemd better for Podman with Quadlet](https://www.redhat.com/sysadmin/quadlet-podman)
* [Deploying a multi-container application using Podman and Quadlet](https://www.redhat.com/sysadmin/multi-container-application-podman-quadlet)
