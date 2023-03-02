

- Go to Cloudflare -> eurekadaycare.com -> Access -> Launch Zero Trust -> Access -> Tunnels and click on "Create a Tunnel"

```
Tunnel name: crow-eurekadaycare

- click "Save tunnel"

- click Next (you'll do this part in a minute)

Subdomain: enter if needed
Domain: eurekadaycare.com
Type: HTTP
URL: 192.168.70.170:8000

- save [NAME] tunnel
```

- The DNS records should be auto-updated




## DNS Settings ##

Should like something like this to begin with:

```
-A       thecanpart.com    public IP (what's my IP)

-CNAME       www               public IP (what's my IP)

-CNAME   start             thecanpart.com
```

* All of the orange proxied sliders can be on

* Until you get it working, put the domain into developer mode.
	* At the time of this writing, it's under thecanpart.com-->Overview


After setting up the tunnel it should look like this:

```
-CNAME    thecanpart.com    297d2cn0-a2fd-418d-ab07-2b764z616de6.cfargotunnel.com

-CNAME    www               thecanpart.com

-CNAME    start             thecanpart.com
```

* the main tunnel cname will say something about flattening and that's fine



## Certificates ##

You need to create a certificate under SSL/TLS-->Origin Server-->Create Certificate. Just follow the instructions and save the files to your computer.


-Upload .pem and .key certificates in the SSL tab of Nginx Proxy Manager




## Authentication ##

Done thru Cloudflare Access

* Login to Cloudflare
* Select domain
* Click Access on the sidebar
* Click “Create Access Policy” for any new apps I need access to from other locations
* I left it off for Overseerr because that app uses Plex as an auth and adding another would deter users
* No need to set up with Plex as there is a plex.tv address with auth already supplied



## Cloudflare Tunnel ##

```
sudo mkdir -p /docker/appdata/cloudflared-mjc

sudo chmod -R 777 /docker/appdata/cloudflared-mjc

sudo docker run -it --rm -v /docker/appdata/cloudflared-mjc:/home/nonroot/.cloudflared/ cloudflare/cloudflared:latest tunnel login

sudo docker run -it --rm -v /docker/appdata/cloudflared-mjc:/home/nonroot/.cloudflared/ cloudflare/cloudflared:latest tunnel create crow-mjc
```

The tunnel key will look something like this:

	8bc2c2a4-1e35-4325-a8ba-86a8379b8187

Create the config file:

```
sudo vim /docker/appdata/cloudflared-mjc/config.yaml
```

Add this and change as necessary:
	* The IP should point to the NPM installation
	* Change the tunnel UUID to your new one
	* Change the domain name as needed

```
tunnel: 8bc2c2a4-1e35-4325-a8ba-86a8379b8187
credentials-file: /home/nonroot/.cloudflared/8bc2c2a4-1e35-4325-a8ba-86a8379b8187.json

# NOTE: You should only have one ingress tag, so if you uncomment one block comment the others

# forward all traffic to Reverse Proxy w/ SSL
ingress:
  - service: https://192.168.40.101:4443
    originRequest:
      originServerName: thecanpart.com

#forward all traffic to Reverse Proxy w/ SSL and no TLS Verify
#ingress:
#  - service: https://REVERSEPROXYIP:PORT
#    originRequest:
#      noTLSVerify: true
```

Install the Docker container
	* Change the tunnel number at the end to match yours

```
sudo docker create --name='cloudflared-mjc' --net='bridge' -e TZ="America/Los_Angeles" -v '/docker/appdata/cloudflared-mjc':'/home/nonroot/.cloudflared/':'rw' --restart unless-stopped 'cloudflare/cloudflared:latest' tunnel run e8589644-e62b-44da-81c3-eb4cd4a6246
```


```
docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token eyJhIjoiMjdmNmYyNDFmM2M2YTYxMDU0NTM1ZmY4Nzc2NWEwNmYiLCJ0IjoiYjk4OWMxOGEtNmI3Zi00MDkwLTk2ZmEtMTczOTRmNmExM2E4IiwicyI6IlptUTVZV0ZsTnpjdE5qYzBPUzAwTkRBekxXSmhaR0V0TWpRME0yWmhZbU5tTlROaiJ9
```


```
sudo docker create --name='cloudflared-mjc' --net='bridge' -e TZ="America/Los_Angeles" -v '/docker/appdata/cloudflared-mjc':'/home/nonroot/.cloudflared/':'rw' --restart unless-stopped 'cloudflare/cloudflared:latest' tunnel run e8589644-e62b-44da-81c3-eb4cd4a6246
```







Start the container in Portainer