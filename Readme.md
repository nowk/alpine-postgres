# alpine-postgres

Postgres + Alpine for Rocket ACI conversion.

    FROM nowk/alpine-bare:3.2


| Stats             |         |
| ----------------- | ------- |
| Docker Image Size | ~23 MB  |
| Rocket ACI Size   | ~9 MB   |

---

`ENV` variables

    PG_MAJOR 9.4
    PG_VERSION 9.4.4
    LANG en_US.utf8
    PGDATA /var/lib/postgresql/data
    PATH /usr/lib/postgresql/$PG_MAJOR/bin:$PATH

---

__Converting:__

    docker2aci docker://nowk/alpine-postgres:9.4.4

*Latest version of the actool will properly export the LABEL directives defined 
in the Dockerfile, else please read below.*

Because the `arch` label is not exported, we will need to add that in manually 
by extracting, modifying the manifest then rebuilding the ACI before adding to
our image store.

    tar xvf nowk-alpine-postgres-9.4.4.aci -C alpine-postgres

Add the `arch` label.

    ...
    "labels": [
        ...
        {
            "name": "arch",
            "value": "amd64"
        },
        ...
    ],
    ...

Rebuild the ACI.

    actool build --overwrite alpine-postgres alpine-postgres-9.4.4-linux-amd64.aci

Add to the image store via `rkt fetch`.

    sudo rkt --insecure-skip-verify fetch alpine-postgres-9.4.4-linux-amd64.aci

__Add as a dependency:__

In your app's ACI `manifest`.

    ...
    "dependencies": [
        {
            "imageName": "nowk/alpine-postgres",
            "labels": [
                {
                    "name": "version",
                    "value": "9.4.4",
                },
                {
                    "name": "os",
                    "value": "linux",
                },
                {
                    "name": "arch",
                    "value": "amd64",
                }
            ]
        }
    ],
    ...
