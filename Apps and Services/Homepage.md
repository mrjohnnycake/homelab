# To Do #

- Need to figure out how to monitor VMs
- Need to figure out how to hide REFRESH icon and VERSION NUMBER
- I need something to monitor ZFS pool(s). Maybe Glances?
- Get Paperless stats working
- Gluetun?
- Figure out Docker integration
	- Needs to be secure
	- Needs to support multiple VMs
- AFTER EVERYTHING IS SETUP, decide on a working layout



# Wants / Requests #

- CSS support
- Hide certain labels
- Additional pages like Homer
- Click on Informational Widgets



# Installation #

```yaml
version: "3.3"
services:
  homepage:
    image: ghcr.io/benphelps/homepage:latest
    container_name: homepage
    ports:
      - 3005:3000
    volumes:
      - /docker/appdata/homepage/config:/app/config
      - /docker/appdata/homepage/public/images:/app/public/images
      - /docker/appdata/homepage/public/icons:/app/public/icons
      - /var/run/docker.sock:/var/run/docker.sock # (optional) For docker integrations
```



# Configuration #

#### widgets.yaml ####

```
# For configuration options and examples, please see:
# https://gethomepage.dev/en/configs/widgets

# - resources:
#     cpu: true
#     memory: true
#     disk: /

- unifi_console:
    url: https://192.168.1.1
    username: [username]
    password: [password]

# - glances:
#   url: http://ip-address:61208
#     username: [username] # optional if auth enabled in Glances
#     password: [password] # optional if auth enabled in Glances
#     label: MyMachine # optional

- openmeteo:
  # label: Eureka # optional
    latitude: 40.784350
    longitude: -124.148840
    timezone: America/Los_Angeles # optional
    units: imperial
    cache: 5 # Time in minutes to cache API responses, to stay within limits

# https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat/DateTimeFormat
- datetime:
    locale: en
    format:
      weekday: long
      month: long
      day: 2-digit
      year: numeric
```

