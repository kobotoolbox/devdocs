---
title: KPI Frontend Development Troubleshooting
---

_Note: we assume you used [kobo-install](https://github.com/kobotoolbox/kobo-install) to setup your development environment._

## Things to do when things don't work

If your http://kf.kobo.local/ is broken, try these things:

1. Check out `python run.py --logs` output.
2. Check http://kf.kobo.local/service_health/ status.
3. You should probably be on `kobo-install` on `master` on latest commit -- see [Branching Reference](https://github.com/kobotoolbox/kpi/wiki/Branching-Reference) and if you're not behind.
4. Restart `npm run watch`, as sometimes hot reloading chokes on itself.
5. Restart `kpi` softly: `./run.py -cf restart kpi`
6. Restart `kobo-install` instance: `python run.py --stop && python run.py`.
7. When there are `kpi` Backend changes on your branch, you may need to rebuild it: `./run.py -cf build kpi`.
8. If you're on `kpi`'s non-`master` branch, try switching to `master` and see if problem still occurs.
9. Rerun migrations: `./run.py -cf exec kpi bash` and `./manage.py makemigrations` plus `./manage.py migrate`
10. If you get this error for Docker on MacOS, try restarting docker: `ERROR: An HTTP request took too long to complete. Retry with --verbose to obtain debug information.`. You can also try increasing timeout by running `COMPOSE_HTTP_TIMEOUT=200 ./run.py`.


## Hardcore options

![fire](/images/fire.gif "Fire!")

1. Restart Docker.
2. Wipe out database by removing `kobo-docker/.vols` directory.
3. Stop all docker containers `docker stop $(docker ps -aq)`.
4. Remove all stopped docker containers and unused data: `docker system prune`. A longer better version:
   ```
   docker rm $(docker ps -a -q); docker rmi $(docker images -q); docker volume rm $(docker volume ls -qf dangling=true)
   ```
5. If you can't stop or kill containers (e.g. `(…) tried to kill container, but did not receive an exit event`) use `docker rm -f $(docker ps -aq)`.
6. Remove `kobo-docker` and `kobo-deployments` directories, and hard restart `kobo-install`: `python run.py --stop && python run.py --setup`.
7. Sometimes old `.pyc` files causes errors while starting `kobo-install` (e.g. `ImportError: cannot import name KpiUidField`), use `find . -name "*.pyc" -type f -delete` in your kpi/kobocat local repository to fix it
8. Restart your machine (srsly)

## Nginx bind issue

This error:

```
nginx_1 | nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx_1 | nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
…
nginx_1 | nginx: [emerg] still could not bind()
```

1. Enter nginx container: `docker exec -it kobo-docker_nginx_1 bash` (your container name may be a bit different).
2. Run `sv stop nginx`.

## Developing on production build

1. Enter your KPI container: `docker exec -it kpi_1 bash`
2. Build: `npm run build`
3. Gather new static files: `python manage.py collectstatic`
4. Exit container
5. Restart KPI: `./run.py -cf restart kpi`

## Debugging

Checking out Enketo logs: `./run.py -cf logs --tail=10 -f enketo_express` at `kobo-install`

Checking out kobocat uwsgi logs: `tail -f uwsgi.log` at `kobo-docker/log/kobocat`

Checking out main logs: `./run.py --logs` at `kobo-install`

Check out kobocat logs `./run.py -cf logs -f --tail=10 kobocat` at `kobo-install`

## Misc problems

### JS out of memory

`ts-loader FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory`

This happens because Node doesn't have enough memory. To fix that, either try upgrading Node to newer version or this in terminal:

```
export NODE_OPTIONS="--max-old-space-size=8192"
```
