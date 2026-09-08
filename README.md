# Lab (Sep 8, 2026): Docker Compose + secrets

This lab is a short intro to Docker Compose: the same `docker run` flags you already know, written down as YAML. The second half is how to pass a real secret without putting it in that YAML.

## Setup

Make sure you have Docker installed. You can download it from the [Docker website](https://www.docker.com/products/docker-desktop). You may also use [OrbStack](https://orbstack.dev/) as an alternative Docker runtime.

## A useful container (the long way)

[Vaultwarden](https://github.com/dani-garcia/vaultwarden) is a lightweight Bitwarden-compatible password manager. This is very similar to the command from lecture.

```bash
docker run -d \
  --name vaultwarden \
  --restart unless-stopped \
  --publish 8080:80 \
  --volume vw-data:/data \
  -e ADMIN_TOKEN=youwillnevergetthis \
  -e LOG_LEVEL=info \
  -e TZ=UTC \
  --memory="512m" \
  --cpus="1.0" \
  vaultwarden/server:latest
```

Go ahead and run it. Then open [http://localhost:8080](http://localhost:8080) and create an account.

The admin page is [http://localhost:8080/admin](http://localhost:8080/admin). Unlock it with the token from `-e ADMIN_TOKEN`.

That's a lot of flags. Each one is doing something:

- `-d` — run in the background
- `--name` — so you can `docker stop vaultwarden` later
- `--restart unless-stopped` — come back after a reboot
- `--publish 8080:80` — without this, the app is listening inside the container and your browser cannot see it
- `--volume vw-data:/data` — vaults live in `/data`. Without a volume they disappear when the container is removed
- `-e` — environment variables. `LOG_LEVEL` and `TZ` are settings. `ADMIN_TOKEN` is a secret
- `--memory` / `--cpus` — don't let one container eat the machine

### Some things to notice

Stop and remove the container, then start it again with the same command:

```bash
docker rm -f vaultwarden
```

Run the long `docker run` again. Your account should still be there. That is the volume.

Now imagine you have to remember that command next week, or send it to a teammate. Super easy to get wrong.

Also look at this:

```bash
docker inspect vaultwarden --format '{{range .Config.Env}}{{println .}}{{end}}'
```

`ADMIN_TOKEN=youwillnevergetthis` is sitting in the container environment. Anyone who can inspect the container can read it.

## Docker Compose

Let's use docker compose to make the same container run with simply `docker compose up -d`.

You can use lectures or docker documentation to help!

## Exercise 1: Write a Compose file

Your job is to replace the `docker run` above with `docker-compose.yml` in this directory.

You should not need a Dockerfile. Use the image `vaultwarden/server:latest` and map every flag from the command. The [Compose file reference](https://docs.docker.com/reference/compose-file/) can help.

When you think you have it:

```bash
docker rm -f vaultwarden
docker compose up -d
```

[http://localhost:8080](http://localhost:8080) should work.

`docker compose config` will print the rendered file. If `ADMIN_TOKEN` is written directly in the YAML, you would be committing it. This is not good, lets fix it!

## Exercise 2: Compose secrets

`-e ADMIN_TOKEN=...` and `environment: ADMIN_TOKEN: ...` both land in `Env`. There is a better approach and we mentioned it in lecture!

HINT: Vaultwarden can load the token from a file. Set the **environment variable** `ADMIN_TOKEN_FILE` to the path of that file (Compose mounts secrets under `/run/secrets/`). This is better because we can add that file to the gitignore.

When you are done, [http://localhost:8080/admin](http://localhost:8080/admin) should still unlock with the token, and `docker inspect` should no longer print it.
