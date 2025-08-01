# Nightscout LibreLink Up Uploader/Sidecar

![docker-image](https://github.com/timoschlueter/nightscout-librelink-up/actions/workflows/docker-image.yml/badge.svg)

Script written in TypeScript that uploads CGM readings from LibreLink Up to Nightscout. The upload should
work with at least Freestyle Libre 2 (FGM) and Libre 3 CGM sensors.

[![Deploy](https://www.herokucdn.com/deploy/button.svg)][heroku]

## Configuration

The script takes the following environment variables

| Variable                 | Description                                                                                                                                                                                                                                       | Example                                  | Required |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------|----------|
| LINK_UP_USERNAME         | LibreLink Up Login Email                                                                                                                                                                                                                          | mail@example.com                         | X        |
| LINK_UP_PASSWORD         | LibreLink Up Login Password                                                                                                                                                                                                                       | mypassword                               | X        |
| LINK_UP_CONNECTION       | LibreLink Up Patient-ID. Can be received from the console output if multiple connections are available.                                                                                                                                           | 123456abc-abcd-efgh-7891def              |          |
| LINK_UP_TIME_INTERVAL    | The time interval of requesting values from libre link up                                                                                                                                                                                         | 5                                        |          |
| LINK_UP_REGION           | Your region. Used to determine the correct LibreLinkUp service (Possible values: AE, AP, AU, CA, DE, EU, EU2, FR, JP, US, LA, RU, CN)                                                                                                             | EU                                       |          |
| NIGHTSCOUT_URL           | Hostname of the Nightscout instance (without https://)                                                                                                                                                                                            | nightscout.yourdomain.com                | X        |
| NIGHTSCOUT_API_TOKEN     | SHA1 Hash of Nightscout access token                                                                                                                                                                                                              | 162f14de46149447c3338a8286223de407e3b2fa | X        |
| NIGHTSCOUT_DISABLE_HTTPS | Disables the HTTPS requirement for Nightscout URLs                                                                                                                                                                                                | true                                     |          |
| NIGHTSCOUT_DEVICE_NAME   | Sets the device name used in Nightscout                                                                                                                                                                                                           | nightscout-librelink-up                  |          |
| LOG_LEVEL                | The setting of verbosity for logging, should be one of info or debug                                                                                                                                                                              | info                                     |          |
| SINGLE_SHOT              | Disables the scheduler and runs the script just once                                                                                                                                                                                              | true                                     |          |
| ALL_DATA                 | Upload all available data from LibreLink Up instead of just data newer than last upload. LibreLinkUp sometimes lags behind in reporting recent historical data, so it is advised to run the script with ALL_DATA set to true at least once a day. | true                                     |          |

## Usage

There are different options for using this script.

### Variant 1: On Heroku

- Click on [![Deploy](https://www.herokucdn.com/deploy/button.svg)][heroku]
- Login to Heroku if not already happened
- Provide proper values for the `environment variables`
- **Important: make sure that yor Nightscout API token is [hashed with SHA1](#hashing-api-token)**
- Click `Deploy` to deploy the app

### Variant 2: Local

The installation process can be started by running `npm install` in the root directory.

To start the process simply create a bash script with the set environment variables (`start.sh`):

```shell
#!/bin/bash
export LINK_UP_USERNAME="mail@example.com"
export LINK_UP_PASSWORD="mypassword"
export LINK_UP_TIME_INTERVAL="5"
export NIGHTSCOUT_URL="nightscout.yourdomain.com"
# use `shasum` instead of `sha1sum` on Mac
export NIGHTSCOUT_API_TOKEN=$(echo -n "librelinku-123456789abcde" | sha1sum | cut -d ' ' -f 1)
export LOG_LEVEL="info"

npm start
```

Execute the script and check the console output.

### Variant 3: Docker

The easiest way to use this is to use the latest docker image:

```shell
docker run -e LINK_UP_USERNAME="mail@example.com" \
           -e LINK_UP_PASSWORD="mypassword" \
           -e LINK_UP_TIME_INTERVAL="5" \
           -e LINK_UP_REGION="EU" \
           -e NIGHTSCOUT_URL="nightscout.yourdomain.com" \
           -e NIGHTSCOUT_API_TOKEN=$(echo -n "librelinku-123456789abcde" | sha1sum | cut -d ' ' -f 1) \
           -e LOG_LEVEL="info" \
           timoschlueter/nightscout-librelink-up
```

### Variant 4: Docker Compose

If you are already using a dockerized Nightscout instance, this image can be easily added to your existing
docker-compose file. In this example, the region is set for germany ("DE"):

```yaml
version: '3.7'

services:
  nightscout-libre-link:
    image: timoschlueter/nightscout-librelink-up
    container_name: nightscout-libre-link
    environment:
      LINK_UP_USERNAME: "mail@example.com"
      LINK_UP_PASSWORD: "mypassword"
      LINK_UP_TIME_INTERVAL: "5"
      LINK_UP_REGION: "DE"
      NIGHTSCOUT_URL: "nightscout.yourdomain.com"
      NIGHTSCOUT_API_TOKEN: "14c779d01a34ad1337ab59c2168e31b141eb2de6"
      LOG_LEVEL: "info"
```

### Hashing API token

`NIGHTSCOUT_API_TOKEN` must be a SHA1 hash of an Access Token from Nightscout (_Add new subject_ first in Nightscout's _Admin Tools_ if required), e.g. your
Access Token for a subject named _LibreLinkUp_ might be `librelinku-123456789abcde`.

Obtain your hash with

```shell
echo -n "librelinku-123456789abcde" | sha1sum | cut -d ' ' -f 1
```

(use `shasum` instead of `sha1sum` on Mac)

which will print the hash (40 characters in length):

```
14c779d01a34ad1337ab59c2168e31b141eb2de6
```

You might also use an online tool to generate your hash, e.g. https://codebeautify.org/sha1-hash-generator

## ToDo

- **Integration into Nightscout**: I have not yet looked into the plugin architecture of Nightscout. Maybe this should
  be converted into a plugin.

[heroku]: https://heroku.com/deploy?template=https://github.com/timoschlueter/nightscout-librelink-up

## Build and push

docker buildx build --load --tag partos/nightscout-librelink-up:latest .
docker push partos/nightscout-librelink-up:latest


