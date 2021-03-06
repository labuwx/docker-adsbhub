# docker-adsbhub

Docker container to send ADSB data to [ADSBHub](https://www.adsbhub.org). Designed to work in tandem with [mikenye/readsb](https://hub.docker.com/repository/docker/mikenye/readsb) or [mikenye/piaware](https://hub.docker.com/repository/docker/mikenye/piaware). Builds and runs on `x86_64`, `arm64` and `arm32v7` (see below).

This container pulls SBS/Basestation protocol data from a host or container, and sends the data to ADSBHub.

Thanks go to [fredclausen](https://github.com/fredclausen) for developing the original image.

## Supported tags and respective Dockerfiles

* `latest` (`master` branch, `Dockerfile`)
* Version and architecture specific tags available

## Multi Architecture Support

Currently, this image should pull and run on the following architectures:

* `amd64`: Linux x86-64
* `arm32v7`: ARMv7 32-bit (Odroid HC1/HC2/XU4, RPi 2/3)
* `arm32v6`: ARMv6 32-bit (Older RPis etc)
* `arm64`: ARMv8 64-bit (RPi 4 64-bit OSes)

## First-time users

### Obtaining ADSBHub Station Key

First-time users should obtain a ADSBHub Station dynamic IP key. Follow the directions for steps 1 and 2 at [ADSBHub how to feed](https://www.adsbhub.org/howtofeed.php), ensuring your station is set up as a client and the data protocol set as "SBS".

### Setting up your station

In your station preferences, you should set the following:

* Feeder type: `Linux`
* Data Protocol: `SBS`
* Station mode: `Client`

## Up-and-Running with `docker run`

```shell
docker run \
 -d \
 --rm \
 --name adsbhub \
 -e TZ="YOURTIMEZONE" \
 -e SBSHOST=YOURSBSHOST \
 -e CLIENTKEY=YOURCLIENTKEY \
 mikenye/adsbhub
```

You should obviously replace `YOURSBSHOST`, and `YOURCLIENTKEY` with appropriate values.

For example:

```shell
docker run \
 -d \
 --rm \
 --name adsbhub \
 -e TZ="Australia/Perth" \
 -e SBSHOST=readsb \
 -e LAT=-33.33333 \
 -e LONG=111.11111 \
 -e CLIENTKEY=zg84632abhf231 \
 mikenye/adsbhub
```

## Up-and-Running with Docker Compose

```yaml
version: '2.0'

services:
  adsbhub:
    image: mikenye/adsbhub:latest
    tty: true
    container_name: adsbhub
    restart: always
    environment:
      - TZ=Australia/Perth
      - SBSHOST=readsb
      - CLIENTKEY=zg84632abhf231
```

### Escaping special characters

The ADSBHub client key is full of special characters, that can be misinterpreted on multiple levels. To avoid that:
* Wrap the environment variable assignment (the whole assignment, not just the key) in single quotation marks so that YAML parses it correctly.
* Duplicate every `$` character. The single `$` sign is the start of a [variable substitution](https://docs.docker.com/compose/compose-file/#variable-substitution) in docker-compose. Use `$$` instead.

If your client key was `abc$123$$$ABC`, your `docker-compose.yml` should look like this:
```yaml
environment:
  - 'CLIENTKEY=abc$$123$$$$$$ABC'
```

## Up-and-Running with Docker Compose, including `mikenye/readsb`

```yaml
version: '2.0'

networks:
  adsbnet:

services:

  readsb:
    image: mikenye/readsb:latest
    tty: true
    container_name: readsb
    restart: always
    devices:
      - /dev/bus/usb/001/007:/dev/bus/usb/001/007
    networks:
      - adsbnet
    command:
      - --dcfilter
      - --device-type=rtlsdr
      - --fix
      - --json-location-accuracy=2
      - --lat=-33.33333
      - --lon=111.11111
      - --metric
      - --mlat
      - --modeac
      - --ppm=0
      - --net
      - --stats-every=3600
      - --quiet
      - --write-json=/var/run/readsb

  adsbhub:
    image: mikenye/adsbhub:latest
    tty: true
    container_name: adsbhub
    restart: always
    environment:
      - TZ=Australia/Perth
      - SBSHOST=readsb
      - CLIENTKEY=zg84632abhf231
    networks:
      - adsbnet
```

For an explanation of the `mikenye/readsb` image's configuration, see that image's readme.

## Ports

No ports are exposed in this container

## Environment variables

| Variable | Description | Required | Default |
|----------|-------------|---------|--------|
| `TZ` | Timezone for the container. | Optional | `UTC` |
| `SBSHOST` | Host for RAW ADSB packets.| Required | Unset |
| `SBSPORT` | Port on SBSHOST that provides beast formatted ADSB packets | Optional | `30003` |
| `CLIENTKEY` | ADSBHub Station Dynamic IP key. | Required | Unset |

## Logging

* All processes are logged to the container's stdout, and can be viewed with `docker logs [-f] container`.

## Getting Help

You can [log an issue](https://github.com/mikenye/docker-adsbhub/issues) on the project's GitHub.

I also have a [Discord channel](https://discord.gg/sTf9uYF), feel free to [join](https://discord.gg/sTf9uYF) and converse.
