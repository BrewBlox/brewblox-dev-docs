# Connecting multiple devices

BrewBlox is designed to let you control multiple devices with a single application.

To make this both reliable and easy, devices are connected to a central hub (the **server**).

<PlantUml src="server_devices.puml" title="Linked devices"/>

On the server, we need some software to talk to individual devices. To make it easy to add new devices, we split the software into **services**.

Some services are used for shared functionality: The **history service** collects data from device services, and stores it for later use in graphs. Others are used to control individual devices.

Some examples of supported devices:
- The BrewPi Spark
- The [Tilt hydrometer](https://github.com/j616/brewblox-tilt)
- The [iSpindel hydrometer ](https://github.com/bdelbosc/brewblox-ispindel)
- The [Plaato digital airlock](https://github.com/BrewBlox/brewblox-plaato)

<PlantUml src="server_services.puml" title="Server services"/>

## Service configuration

Services are configured using the `docker-compose.yml` file. [YAML](https://learnxinyminutes.com/docs/yaml/) is a markup language that uses indentation to show nested values.

A shortened configuration file:

```yaml
version: "3"

services:
  history:
    image: brewblox/brewblox-history:edge

  spark-one:
    image: brewblox/brewblox-devcon-spark:edge

  spark-two:
    image: brewblox/brewblox-devcon-spark:edge

```

There are three services here:
* history
* spark-one
* spark-two

All of them have a unique name, but `spark-one` and `spark-two` share the same type. That's ok: as long as the name is unique, services can have the same settings.

GOOD:
```yaml
services:
  service-one:
    image: brewblox-image-one

  service-two:
    image: brewblox-image-one
```

BAD: 
```yaml
services:
  service-one:
    image: brewblox-image-one

  service-one:
    image: brewblox-image-two
```

## Adding devices

If we want to add a new device, we need a new service to manage it. Once again: the name must be unique, but the type can be the same.

Each type of service may have a slightly different configuration. We'll take a detailed look at a Spark service here, but other services will have configuration that is very much like it.

## Service configuration: settings

When you install BrewBlox, it generates a `docker-compose.yml` file for you. This includes the default `spark-one` service.

```yaml
  spark-one:
    image: brewblox/brewblox-devcon-spark:rpi-${BREWBLOX_RELEASE:-stable}
    privileged: true
    restart: unless-stopped
    labels:
      - "traefik.port=5000"
      - "traefik.frontend.rule=PathPrefix: /spark-one"
    command: >
      --name=spark-one
      --mdns-port=${BREWBLOX_PORT_MDNS:-5000}
```

This configuration is more advanced than what we've seen so far. To make sense of it, we'll look at the individual settings.


---
```yaml
spark-one:
  image: brewblox/brewblox-devcon-spark:rpi-${BREWBLOX_RELEASE:-stable}
  ...
```

This is like the short service configurations we saw earlier. The two most important settings in a service's configuration are its name (spark-one), and its type (everything after `image:`).

The basic principles still apply. When you want to control multiple Spark devices, you'll need one service per device. Every service will have the same `image`, but a different name.

---
```yaml
  ...
  privileged: true
  restart: unless-stopped
```

These settings are the same for every Spark service (and many other services).

`privileged` services can use USB connections. 

`restart: unless-stopped` does what it says: when your service crashes, it will automatically restart.

---
```yaml
  ...
  labels:
    - "traefik.port=5000"
    - "traefik.frontend.rule=PathPrefix: /spark-one"
```

Labels are used by the Traefik gateway service. The gateway makes it possible for the UI to communicate with the service.

Not all services need this. For example: Tilt, iSpindel, and Plaato services don't have their own widgets in the UI. They send data to the history service, and the history service sends that data to the UI.

**The `PathPrefix` setting in the label must match the service name.** If you add the `spark-two` service, the second label must be:

```yaml
"traefik.frontend.rule=PathPrefix: /spark-two"
```

---
```yaml
  ...
  command: >
  --name=spark-one
  --mdns-port=${BREWBLOX_PORT_MDNS:-5000}
```

The `command` setting contains settings for the software running *inside* the service.

The `--name` argument must (again) be the same as the service name.

::: tip
The service name is mentioned three times in the YAMl for a Spark service. It must be the same each time.
- at the top (`spark-one:`)
- in the labels (`PathPrefix: /spark-one`)
- in the command (`--name=spark-one`)
:::

For a Spark service, the command is where you add the settings for [how it connects to a Spark controller](./connect_settings.html)