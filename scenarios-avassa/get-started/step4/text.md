We will continue with a more advanced application and an application upgrade scenario.

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

Open in the editor:

`theater-room-manager.yml`{{open}}


<br>
And the deployment:

```yaml
name: theater-room-manager-deployment
application: theater-room-manager
application-version: "*"
placement:
  match-site-labels: >
    system/type = edge
```{{copy}}

You can open that in the editor:

`theater-room-manager-deployment.yml`{{open}}




