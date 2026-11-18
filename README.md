# Visit counter

A small web app that counts how many times the page was opened. I made it for a Docker
course: it runs in three containers with Docker Compose, and on the server Caddy is added
in front of them for HTTPS.

## How to run it

https://student12.wt.thomasjuldo.com

To put an update online, I change the code on my laptop, commit and push it to GitHub.
Then on the server, in the project folder:

```
git pull
docker compose -f compose.yaml -f compose.server.yaml up -d --build
```

`git pull` gets the new commits from GitHub. `up -d --build` rebuilds the images that
changed and only replaces the containers that need it. The visit count is not lost,
because the database is stored in a volume.

## What's inside

- `site/index.html`: the page. It asks the API for the number and shows it.
- `api/`: the API, a small Python program made with Flask, and its Dockerfile. Each call
  to `/visits` adds one to the counter in the database.
- `compose.yaml`: the three services (`site` with nginx, `api`, and `db` with Redis) and
  the `counter-data` volume that keeps the count.
- `compose.server.yaml`: the changes for the server. It adds Caddy, removes the ports of
  `site` and `api` so that only Caddy is open to the internet (ports 80 and 443), and
  makes every service restart by itself after a crash or a reboot.
- `Caddyfile`: the Caddy settings. Requests starting with `/api/` go to the API, everything
  else goes to the site. Caddy gets the HTTPS certificate by itself. Requests starting with
  `/lastfm/` go to my other project, [lastfm-map](https://github.com/AlexHAGT/lastfm-map),
  which runs on the same server and joins this project's Docker network.

## Run it on your own computer

With Docker running:

```
docker compose up -d --build
```

Then open http://localhost:8081 and reload the page a few times, the number goes up each
time. To stop everything, run `docker compose down` (add `-v` only if you also want to
delete the count).

## Note about the database

The database has no password. It can't be reached from outside: `db` doesn't publish any
port, and only the API talks to it, on the private network that Compose creates.
