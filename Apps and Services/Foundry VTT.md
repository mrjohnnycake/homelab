
- Items Directory? (what's it for?)
- Rollable Tables?


Dungeon map dimensions in a note
670x467


https://tools.2minutetabletop.com/token-editor/



# Usage #

## Combat ##

- right click




Jess- Clifton707
Emmett- Pokemon707
Calvin- Carson95501
Miles- Carson95501
Charlie- Carson95501




# Install #

```
sudo useradd -u 421 -M -s /usr/sbin/nologin foundry

sudo usermod -aG docker foundry

sudo usermod -aG foundry dave
```


```yml
version: "3.8"

services:
  foundry:
    image: felddy/foundryvtt:release
    container_name: foundry-vtt
    hostname: foundryvtt
    volumes:
      - /docker/appdata/foundry-vtt:/data
      - /mnt/FoundryVTT:/data/Data
    environment:
      - FOUNDRY_PASSWORD=gwh_dwv8mvw6NPJ*dxc
      - FOUNDRY_USERNAME=mrjohnnycake
      - FOUNDRY_ADMIN_KEY=NasaogXm*MxVRzjj7.jd
      - CONTAINER_PRESERVE_CONFIG=true
      - PUID=1010
      - PGID=1000
      - TZ=America/Los_Angeles
    ports:
      - 30000:30000/tcp
    restart: unless-stopped
```
