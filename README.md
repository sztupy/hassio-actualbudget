# hassio-actualbudget

Home Assistant repository to run Actual Budget as an addon

## Frequently Asked Questions / FAQ

### Do you support Nabu Casa or similar?

No, the add-on currently only supports the method described below using NGINX Proxy Manager. Nabu Casa and similar ingress based deployments are not currently supported. This is due to limitations in Actual Budget itself. For more info see [Issue #5](https://github.com/sztupy/hassio-actualbudget/issues/5#issuecomment-2039383654)

### Actual Budget got updated recently, when can I expect it to be available through Home Assistant?

Both the edge and stable versions are tracked from the official sources automatically and the latest releases should generally be available through Home Assistant within 24 hours. If it has been more than that time, feel free to create a GitHub Issue detailing the problem.

# Installation

## Add-Ons

The following are based on obtaining a free DuckDNS domain with a Let's Encrypt certificate. If you want to use your own domain the steps around DuckDNS won't apply.

First to install use the following link to add the repository:

[![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fsztupy%2Fhassio-actualbudget)

Alternatively go to the Add-On Store, Click the three dots, then Repositories, and then add the following:

```
https://github.com/sztupy/hassio-actualbudget
```

Once present you will need to install the following Add-Ons:

* DuckDNS
* MariaDB
* Nginx Proxy Manager (note: this is not the Nginx Home Assistant SSL Proxy)
* Actual Budget (either the Stable or the Edge version)

Note: the Stable and Edge versions can be installed in parallel but they will use a different datastore. To sync between the two you would need to download a backup and restore that in the other instance manually.

## DuckDNS

Make sure you have an account set up with DuckDNS http://www.duckdns.org/ Once logged in note the `Token` value at the top as it'll be needed. You should also set up your DuckDNS domain. In the examples below we will use `hassio-actualbudget.duckdns.org`

Once registered go to the Add-On config in Home Assistant for DuckDNS and add your domain into the `Domains` section, and your token into the `Token` section. Change the `accept_terms` section to `true` to accept the Let's Encrypt terms and conditions.

Once the changes are done go to `Info` and `START` the service. After a minute or so it should start up, if there are issues you can check the `Log` page for notes

## MariaDB

Before you start MariaDB you should go to it's Add-On configuration and change the Login password to something more secure. Once that's done keep the rest of the config and `START` the Add-On

## Actual Budget

There is no configuration for Actual Budget. If you plan on running both the Stable and Edge versions, make sure you change the Port for one of them to something else than the default `5006`. You can then `START` the service.

To check it has started you should navigate to `http://homeassistant.local:5006`. There will be a warning message about the connection not being secure, which we will sort out in the next step.

## Nginx Proxy Manager

Finally `START` the Nginx Proxy Manager. Once started go to `http://homeassistant.local:81` to set it up

### User management

1. The default username/password is `admin@example.com` / `changeme`
2. You will need to change both the username and password above after the first login

### SSL setup

1. After this go to the `SSL Certificates` tab, and click `Add SSL Certificate`. Use `Let's Encrypt`
2. In the box that opens enter your domain name you set up in DuckDNS. Note, you can alternatively also set up an SSL Certificate for the wildcard version, e.g `*.hassio-actualbudget.duckdns.org` if you wish to use the certificate for multiple proxies, for example the stable and the edge versions separately.
3. Enter your email address, and click on `Use a DNS Challenge`
4. From the DNS Provider box select `DuckDNS`, and in the configuration section below replace the `your-duckdns-token` part with your DuckDNS token. Keep the rest of the config
5. Agree once again to the Let's Encrypt T&C, and click `Save`

After a while the certificate will be ready, and you are allowed to continue

### Proxy setup

1. Go to `Hosts` -> `Proxy hosts`
2. Click `Add Proxy Host`
3. Set up the domain name you have. If you have set up a wildcard certificate in the previous section then make sure you use a subdomain, e.g. `stable.hassio-actualbudget.duckdns.org`
4. Scheme should be set to `http`, forward hostname to `homeassistant.local` and port to `5006` (or the port you have configured for Actual Budget)
5. Go to the `SSL` tab and select the certificate you created earlier
6. Click `Save`

## Router and DNS config

The following steps are based on where you would like to access your Actual Budget instance.
### Local-only

If you only wish to access Actual Budget in your local network then no changes are needed on the router. However you will need to set up a local DNS update to make sure that your domain points to your local instant of homeassistant. The easiest way to do it is by changing your hosts file:

1. First obtain your homeassistant IP address. You can do that from a terminal by typing `ping homeassistant.local`. You should get an IP address that looks something like `192.168.1.181`
2. Next edit your hosts file. On Linux and OSX this will be at `/etc/hosts` on Windows this will be at `c:\Windows\System32\Drivers\etc\hosts`. Note you will need admin privileges to edit these files.
3. Add the following line to the end of the hosts file (replacing both the domain and the IP address with the ones you obtained):
```
hassio-actualbudget.duckdns.org 192.168.1.181
```
4. Save this file

Once you have done this you should be able to access Actual Budget on `https://hassio-actualbudget.duckdns.org` with a fully verified certificate and no security errors. Note that you will need to do the `hosts` file change for every computer you wish to access your budget from, and your budget will only be accessible from your home network, and not from the internet.

On mobile phones where the hosts file cannot be edited you can use some "DNS filtering" applications, adding your domain above to the filter list (making sure it points to the IP address obtained above)

### Local & Remote

If you wish to access your Actual Budget instance anywhere in the world, you'll need to update your router settings to forward port `443` to your Home Assistant box. This step is different for each router, and you should consult the manual on how to do this step.

Once this has been done you should be able to access `https://hassio-actualbudget.duckdns.org` from anywhere where you have internet connection.

Do note that enabling port forwarding on your home router can expose you to additional security risks, so thread carefully when using this option. Your router's manual might have additional suggestion on how to keep your system as safe as possible after port forwarding is set up.
