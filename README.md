# Keelson Connector ANavS - Binary Solution Output Format (up to 125Hz)

ANAVS connector to there own defined binary message format, as ANAVS can also output NMEA but lilited aroud 10Hz due to NMEA convetions. The Binary format is targeting more automation and integratoom of robotic and therefor higher data output range. 

### Docker setup
The `Dockerfile` builds a container containing the connector. `docker-compose.nmea.yml` provides examples for running the connector with SOCAT using either an UDP or USB source.

### Experimental notebooks
The `experimental` folder contains Jupyter notebooks and recorded JSON examples that demonstrate how to work with the produced data.

## SOCAT install
```sh
sudo apt install socat
```

## Quick start (SOCAT pipe)
```bash

# TCP
socat TCP:192.168.1.124:6001 STDOUT | ./bin/main --log-level 10 -r rise -e ssrs18 -s sealog --publish all

```



Setup for development environment on your own computer:

1. Install [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)
   - Docker desktop provides an UI for monitoring and controlling docker containers
   - If you want to learn more about docker and its building blocks checkout [Docker quick hands-on guide](https://docs.docker.com/guides/get-started/)
2. Start up of **Zenoh router** either on your computer or another machine within your local network

   ```bash
   # Navigate to folder containing docker-compose.zenoh-router.yml

   # Start router with log output
  docker-compose -f docker-compose.zenoh-router.yml up

   # If no obvious errors, stop container with "ctrl-c"

   # Start container and let it run in the background/detached (append -d)
  docker-compose -f docker-compose.zenoh-router.yml up -d
   ```

  [docker-compose.zenoh-router.yml](docker-compose.zenoh-router.yml)

3. Now the Zenoh router should be running and available on localhost:8000. This can be tested with the [Zenoh Rest API](https://zenoh.io/docs/apis/rest/) or by continuing to the next step using the Python API
4. Set up a python virtual environment (`python >= 3.11`)
   1. Install packages with `pip install -r requirements.txt`
5. Explore example scripts in the [experimental folder](./experimental/)
   1. Samples are based on the [Zenoh Python API](https://zenoh-python.readthedocs.io/en/0.10.1-rc/#quick-start-examples)

[Zenoh CLI for debugging and problem solving](https://github.com/RISE-Maritime/zenoh-cli)
