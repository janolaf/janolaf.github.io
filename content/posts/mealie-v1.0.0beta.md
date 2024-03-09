---
title: "Setting up Mealie using Podman"
date: 2022-11-19
draft: false
toc: false
images:
categories:
  - podman
tags:
  - mealie
  - podman
---
[Mealie](https://mealie.io/) is a self hosting recipe manager and meal planner. You can import recipes from your favorite cooking blogs or websites.

*WARNING* The instructions I have posted are probably not best practice. 
 
## Why Mealie
 
I chose Mealie over the competition, because it was quite simple to setup. Originally, setting up Mealie required pulling and configuring one container.
 
```bash
podman run --name mealie 
    -p 9925:80
    -v /srv/mealie/data:/app/data:Z
    hkotel/mealie:latest
```
It was that simple. With version 1.0 beta, that is no longer true.
 
## Version 1.0.0 Beta
 
Version 1.0.0 beta has split Mealie into two containers, a frontend and an api container.
 
You can use Mealie's preferred method, docker-compose. I prefer using a shell script or podman pods yaml file over docker-compose. Just a personal preference, and with just a little work, it is easy to get Mealie 1.0.0 Beta to play nice with podman.
 
I must confess, initially I had a problem getting Mealie 1.0 working in podman. I never figured out exactly what all the issues were. One issue was `PUID` and `PGID`, do not use environmental variables `PUID` or `PGID`, podman runs in userspace anyways. I set the timezone, using podmans `--tz=local` flag. If you have problems, you can use environmental variable `TZ=` instead.
 
If you have SELinux enabled, Use `z` and not `Z`, both containers need read/write permission to the same volume. My example below assumes SELinux is enabled on your system.
 
## Creating the Mealie Pod
 
These instructions should work with Podman 3.4.7 and up. There have been some changes to Podman 4.0 that I will not go into here.
 
Rather than run the containers like most examples, I create the containers first. I see less issues this way.
 
First create the pod
 
```bash
podman pod create --name mealie -p 9925:3000
```
 
The mealie-api container. Change BASE_URL to your dns address if you have one. You can change ALLOW_SIGNUP to false, if you like.
 
``` bash
podman container create --name mealie-api --pod mealie
  --replace
    -v /srv/mealie/data:/app/data:z
    -e ALLOW_SIGNUP=true
    -e MAX_WORKERS=1
    -e WEB_CONCURRENCY=1
    --tz=local
    -e BASE_URL=http://mealie.foo.com
    hkotel/mealie:api-v1.0.0beta-5
```
 
```bash 
podman container create --name mealie-frontend --pod mealie
  --replace
    -v /srv/mealie/data:/app/data:z
    -e API_URL=http://mealie-api:9000
    hkotel/mealie:frontend-v1.0.0beta-5
```
 
start the pod:
 
```bash
podman pod start mealie
```
 
If everything is working, you should be able to access Mealier from localhost:9925.
 
## Using systemd to manage Mealie pod
 
If everything is working, we can generate the systemd service file to manage the startup and shutdown of Mealie.
 
If you have not done so already, create:
 
```bash
mkdir -p ~/.config/systemd/user
```
 
and stop the Mealie Pod
 
```bash
podman pod stop mealie
```
 
Next, generate the systemd service file:
 
```bash
podman generate systemd --name mealie --file ~/.config/systemd/user/
```
 
This should create `pod-mealie.service`, `container-mealie-api.service` and `container-mealie-frontend.service` in `~/.config/systemd/user`.
 
We need to reload systemd so it can see the new service files.
 
```bash
systemctl --user daemon-reload
```
 
Then start Mealie pod through systemd. `pod-mealie.service` will start the `container-mealie-api.service` and `container-mealie-frontend.service`
 
```bash
systemctl --user start pod-mealie
```
 
Go to `localhost:9925`. If everything is working we can enable Mealie to start up on boot.
 
```bash
systemctl --user enable pod-mealie
```
 
## Updating Mealie Beta
 
Right now, there is no :latest tag in the 1.0.0. beta release. Each time Mealie makes a new release available, you are going to have to manually change the containers your pod uses. So far, all it has required is changing `hkotel/mealie:api-v1.0.0beta-4` to `hkotel/mealie:api-v1.0.0beta-5`.
 
Which is why I maintain a bash shell script. I know I can generate a kubernetes yaml file, but I am more familiar with bash. And there is always the possibility of major, breaking changes, coming down the line.

## Conclusion

I hope this helps podman users with setting up Mealie. Other than a few hiccups at the beginning, Mealie is easy to setup and works will under podman.

I look forward to Mealies continued development. 
