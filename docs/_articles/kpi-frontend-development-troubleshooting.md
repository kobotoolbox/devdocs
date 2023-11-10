---
title: KPI Front-end Development Troubleshooting
---

_Note: we assume you used [kobo-install](https://github.com/kobotoolbox/kobo-install) to setup your development environment._

## Things to do when things don't work

If your http://kf.kobo.local/ is broken, try these things:

1. Check out `python run.py --logs` output.
  - Hint: you can use `./run.py -cf logs -f --tail 10 kpi` to only see latest logs. Useful if your instance is running for long time.
2. Check http://kf.kobo.local/service_health/ status.
3. You should probably be on `kobo-install` on `master` on latest commit -- see [Branching Reference](https://github.com/kobotoolbox/kpi/wiki/Branching-Reference) and if you're not behind.
4. Restart `npm run watch`, as sometimes hot reloading chokes on itself.
5. Restart `kpi` softly: `./run.py -cf restart kpi`
6. Restart `kobo-install` instance: `python run.py --stop && python run.py`.
7. When there are `kpi` Backend changes on your branch, you may need to rebuild it: `./run.py -cf build kpi`.
8. If you're on `kpi`'s non-`master` branch, try switching to `master` and see if problem still occurs.
9. Rerun migrations: `./run.py -cf exec kpi bash` and `./manage.py migrate`
10. If you get this error for Docker on MacOS, try restarting docker: `ERROR: An HTTP request took too long to complete. Retry with --verbose to obtain debug information.`. You can also try increasing timeout by running `COMPOSE_HTTP_TIMEOUT=200 ./run.py`.
11. If you need to refresh static files (e.g. favicon), there is [a neat script for that](/files/refresh-frontend-files.sh).
12. If `npm install` is failing, check your `npm --version` and compare with one that is known to work. Sometimes when you install node you get a different npm version than everyone else has. Apparently the dependency resolution algorithm can behave differently even between minor versions, like `8.5.5` vs `8.11`. You can use your npm to install a different version, for example, `npm install -g npm@8.5.5`.
13. If you get `502` error on `kf.kobo.local`, please use `./run.py -cf restart nginx`

## Hardcore options

![fire](/images/fire.gif "Fire!")

1. Force restart `kpi`:
   ```
   ./run.py -cf stop kpi && ./run.py -cf rm kpi && ./run.py -cf build --no-cache --force-rm kpi
   ```
2. Restart Docker.
3. Wipe out database by removing `kobo-docker/.vols` directory.
4. Stop all docker containers `docker stop $(docker ps -aq)`.
5. Remove all stopped docker containers and unused data: `docker system prune`. A longer better version:
   ```
   docker rm $(docker ps -a -q); docker rmi $(docker images -q); docker volume rm $(docker volume ls -qf dangling=true)
   ```
6. If you can't stop or kill containers (e.g. `(…) tried to kill container, but did not receive an exit event`) use `docker rm -f $(docker ps -aq)`.
7. Remove `kobo-docker` and `kobo-deployments` directories, and hard restart `kobo-install`: `python run.py --stop && python run.py --setup`. Note: before restarting you might want to wipe out static files: `rm -rf kpi/staticfiles`
8. Sometimes old `.pyc` files causes errors while starting `kobo-install` (e.g. `ImportError: cannot import name KpiUidField`), use `find . -name "*.pyc" -type f -delete` in your kpi or kobocat local repository to fix it
9. Restart your machine (srsly)

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

Note: If you happen to get this error inside Docker container (e.g. while running `kobo-install`), you might need to allocate more resources to Docker app.

### No styles on admin page

When you use `kobo-install` with option to run `npm` locally, sometimes you can end up with `/api/` and `/admin/` pages missing CSS files. To fix that you need to:

1. Enter KPI container: `./run.py -cf exec kpi bash`.
2. Run `python manage.py collectstatic --noinput`.
3. Run `rsync -aq --chown=www-data "${KPI_SRC_DIR}/staticfiles/" "${NGINX_STATIC_DIR}/"`
4. Exit container and restart Nginx: `./run.py -cf restart nginx`

### kobo-install self restarting death loop

Sometimes `./run.py` is stuck in a loop of containters restaring endlessly. This can happen because of missing database migrations (e.g. you pulled some commits that contains back-end changes). When this happens you would see something like `django.db.utils.ProgrammingError: relation "settings__country_codes_idx" already exists` (or other error just after `Running migrations:` line). To fix that try:

1. Enter the kpi container: `./run.py -cf exec kpi bash`
2. Run migrations: `./manage.py migrate kpi`

Sometimes the error is a bit different, if so try some of these tricks:
- tell database that all migrations are applied: `./manage.py migrate kpi --fake` and then go back few migrations and re-run migrations
- go back few migrations: `./manage.py migrate kpi 0001` (you can put any migration identifier number here)
- if you get kicked out of kpi container, try `./run.py -cf run --rm kpi bash` (this one will not get ejected, and you can fix issues from within)

### kobo-install re-run "cannot stop container"

Sometimes when re-running `./run.py` the script will have a problem stopping some containers. E.g.:

```
(…)
Stopping kobobe_redis_main_1  ... error
(…)
ERROR: for kobobe_redis_main_1  cannot stop container: 59672659abeec9aafc7f979dfd920a855cf3a36d798fa1fe92f42e5ec117a1a4: tried to kill container, but did not receive an exit event
(…)
An error has occurred
```

If you see that, and you are on Mac, try restarting your Docker app.

### kobo-install fails to build because of "invalid signature"

If you encounter similar error to the one below while running `./run.py --build`, the way to fix it is to cleanup your docker, as it possibly ran out of space. Try running prune or the more complex command from "Hardcore options" above.

```
 => ERROR [stage-1  3/18] RUN apt-get -qq update &&     apt-get -qq -y install curl &&     curl -sL https://deb.nodesource.com/setup_16.x | bash - &&     apt-ge  0.6s
------
 > [stage-1  3/18] RUN apt-get -qq update &&     apt-get -qq -y install curl &&     curl -sL https://deb.nodesource.com/setup_16.x | bash - &&     apt-get -qq -y install --no-install-recommends         ffmpeg         gdal-bin         gettext         git         gosu         less         libproj-dev         locales         nodejs         postgresql-client         rsync         runit-init         vim-tiny         wait-for-it &&     apt-get clean &&         rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*:
#7 0.586 W: GPG error: http://deb.debian.org/debian bullseye InRelease: At least one invalid signature was encountered.
#7 0.586 E: The repository 'http://deb.debian.org/debian bullseye InRelease' is not signed.
#7 0.586 W: GPG error: http://deb.debian.org/debian-security bullseye-security InRelease: At least one invalid signature was encountered.
#7 0.586 E: The repository 'http://deb.debian.org/debian-security bullseye-security InRelease' is not signed.
#7 0.586 W: GPG error: http://deb.debian.org/debian bullseye-updates InRelease: At least one invalid signature was encountered.
#7 0.586 E: The repository 'http://deb.debian.org/debian bullseye-updates InRelease' is not signed.
------
executor failed running [/bin/sh -c apt-get -qq update &&     apt-get -qq -y install curl &&     curl -sL https://deb.nodesource.com/setup_16.x | bash - &&     apt-get -qq -y install --no-install-recommends         ffmpeg         gdal-bin         gettext         git         gosu         less         libproj-dev         locales         nodejs         postgresql-client         rsync         runit-init         vim-tiny         wait-for-it &&     apt-get clean &&         rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*]: exit code: 100
ERROR: Service 'kpi' failed to build : Build failed
An error has occurred
```
