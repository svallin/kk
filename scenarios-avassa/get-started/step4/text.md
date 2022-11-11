We will continue with a more advanced application and an application upgrade scenario.

The application below has several services (theater-operations, curtain-controller) and it also picks up secrets (user-name, password).
Note also that we have explicit versions on the containers as well as a version for the application itself.

```yaml
name: theater-room-manager
version: "1.0"
services:
  - name: theater-operations
    share-pid-namespace: false
    variables:
      - name: OPERATIONS_USERNAME
        value-from-vault-secret:
          vault: operations
          secret: credentials
          key: username
    containers:
      - name: projector-operations
        image: registry.gitlab.com/avassa-public/movie-theaters-demo/projector-operations:v1.0
        on-mounted-file-change:
          restart: true
      - name: digital-assets-manager
        image: registry.gitlab.com/avassa-public/movie-theaters-demo/digital-assets-manager:v1.0
        mounts:
          - volume-name: credentials
            mount-path: /credentials
        env:
          USERNAME: ${OPERATIONS_USERNAME}
        on-mounted-file-change:
          restart: true
    mode: replicated
    replicas: 1
    volumes:
      - name: credentials
        vault-secret:
          vault: operations
          secret: credentials
  - name: curtain-controller
    share-pid-namespace: false
    containers:
      - name: curtain-controller
        image: registry.gitlab.com/avassa-public/movie-theaters-demo/curtain-controller:v1.0
        on-mounted-file-change:
          restart: true
    mode: replicated
    replicas: 1
```{{copy}}

<br>

When the application is scheduled on sites it will mount the secrets as files. Secrets are configure in the Control Tower and distributed encrypted to the sites. Create a strongbox vault with secrets as shown below.

Create the vault with secrets. For simplicity we are just distributing the secrets to all sites. There are fine-grained distribution policies as well. 

1. Create the vault

```plain
./supctl create strongbox vaults <<EOF               
name: credentials
distribute:
  to: all
EOF
```{{exec}}

2. Create the secret with values:

```plain
./supctl create strongbox vaults credentials secrets <<EOF
name: operations
data:
  password: the-password
  username: the-user
allow-image-access:
  - "*"
EOF
```{{exec}}

<br>

Now, we can deploy the application:

```plain
./supctl create application-deployments <<EOF
name: theater-room-manager-deployment
application: theater-room-manager
application-version: "*"
placement:
  match-site-labels: >
    system/type = edge
EOF
```{{exec}}

Check the status of the deployment

```plain
./supctl show application-deployments theater-room-manager-deployment
```{{exec}}

<br>

Now, time for a version upgrade. In this example the deployemt specification referred to version '*' of the application.
This means that whenever the application version is upgraded it will be immediately upgraded on the sites. A more controlled model would have been to refer to a specific application version in the deployment specification. Anyhow, lets upgrade:

We will use the merge operation which replaces matching config lines, we are modifying the application version from 1.0 to 1.1 and picking v2.0 of the `projector-operations` container

```plain
./supctl merge applications theater-room-manager <<EOF
name: theater-room-manager
version: "1.1"
services:
  - name: theater-operations
    containers:
      - name: projector-operations
        image: registry.gitlab.com/avassa-public/movie-theaters-demo/projector-operations:v2.0
EOF
```{{exec}}


Now, check the status of the application to see that the upgrade made it:
```plain
./supctl show application-deployments theater-room-manager-deployment
```{{exec}}

