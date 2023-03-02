I don't like this app for my needs. You can only edit files thru it so if you edit outside of it you'll have to update the database to see those changes.

It'll work for Anna and the boys but not my workflow.




```d
version: '3'

volumes:

  nextcloud-db:

services:

  nextcloud-app:
    image: nextcloud:latest
    restart: unless-stopped
    volumes:
      - /docker/appdata/nextcloud:/var/www/html
	  - /docker/appdata/nextcloud/custom_apps:/var/www/html/custom_apps
      - /docker/appdata/nextcloud/config:/var/www/html/config
      - /docker/appdata/nextcloud/data:/var/www/html/data
#     - /docker/appdata/nextcloud/themes/<YOUR_CUSTOM_THEME>:/var/www/html/themes/<YOUR_CUSTOM_THEME>
    environment:
      - MYSQL_PASSWORD=$MYSQL_PASSWORD
      - MYSQL_DATABASE=$MYSQL_DATABASE
      - MYSQL_USER=$MYSQL_USER
      - MYSQL_HOST=nextcloud-db
    ports:
      - 80:80

  nextcloud-db:
    image: mariadb:latest
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - nextcloud-db:/var/lib/mysql
    environment:
      - MYSQL_RANDOM_ROOT_PASSWORD=true
      - MYSQL_PASSWORD=$MYSQL_PASSWORD
      - MYSQL_DATABASE=$MYSQL_DATABASE
      - MYSQL_USER=$MYSQL_USER
```





```d

networks:
  frontend:
    # add this if the network is already existing!
    # external: true
  backend:

services:

  nextcloud-app:
    image: nextcloud
    restart: always
    volumes:
      - nextcloud-data:/var/www/html
    environment:
      - MYSQL_PASSWORD=omhxRqQ7wzhH_3r69kHy
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=nextcloud-db
    networks:
      - frontend
      - backend

  nextcloud-db:
    image: mariadb
    restart: always
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - nextcloud-db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=Wa.RYc2KpEzc-UXJRtw_
      - MYSQL_PASSWORD=omhxRqQ7wzhH_3r69kHy
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
    networks:
      - backend
```