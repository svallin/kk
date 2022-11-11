It is now time to create your first application. The application is structured according to: application-service-container. A service is the entity that the scheduler places on hosts. Along with the above structure you can define, mounts, configuration and many other configurations. See [Application](https://docs.avassa.io/api.html#tag/Applications/operation/v1_config_applications_post) for details.

We will now perform the steps in the ["Deploy your first application"](https://docs.avassa.io/docs/tutorials/first-application) tutorial.

Use the editor and create the following application. Save it to popcorn.yml

```yaml
name: popcorn-controller
services:
  - name: popcorn-service
    mode: replicated
    replicas: 1
    containers:
      - name: kettle-popper-manager
        image: registry.gitlab.com/avassa-public/movie-theaters-demo/kettle-popper-manager
```{{copy}}
<br>

`popcorn.yml`{{open}}

The above application specification says:

1. name the application popcorn-controller
2. it contains a single service with a single container
3. the container is fetched from the public repository registry.gitlab.com/avassa-public/
4. the scheduler will start one replica of that service

Load that application:

```plain
./supctl create applications < popcorn.yml 
```{{exec}}

<br>

You can validate that it is registered in the Control Tower

```plain
./supctl show applications popcorn-controller
```{{exec}}

Note well that the status tells that the images has been fetched and store in the Control Tower registry.
You can also use the Control Tower to see the application. (You may need to refresh the window)

<br>
Now it is time to deploy the application. Use the editor to create the following deployment. Save it in popcorn-deployment.yml

```yaml
name: popcorn-deployment
application: popcorn-controller
placement:
  match-site-labels: system/type = edge
```{{copy}}


The interesting piece above is the label matching placement. Avassa will find all sites with a matching label and request the scheduler to run it there.
Now, create that deployment:

```plain
./supctl create application-deployments < popcorn-deployment.yml
```{{exec}}

<br>

Check the state of the deployment:

```plain
./supctl show application-deployments popcorn-deployment
```{{exec}}

The first time you perform the command above it might say `deploying-to`, perform the command again and you will most likely see the `deployed-to` state.

You can also get a summary status of the *application health* across all sites

```plain
./supctl show application-status applications popcorn-controller
```{{exec}}

In order to drill down and see details about an application on a site you can do

```plain
./supctl show --site gothenburg-bergakungen applications popcorn-controller 
```{{exec}}

An important use case is to be able to see the container logs on the sites. The command below shows the container logs on the gothenburg-bergakungen since 10 minutes. Note well that you can use tab completion to build the command

```plain
./supctl do --site gothenburg-bergakungen volga topics system:container-logs:popcorn-controller.popcorn-service-1.kettle-popper-manager consume --position-since 10m
```{{exec}}

