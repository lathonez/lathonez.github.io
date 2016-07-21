---
title:  "Securing a docker registry behind Apache"
date:   2016-07-20 12:34:23
categories: [dev]
tags: [docker, registry, apache, letsencrypt]
---

We recently went to setup a docker registry on our production build server. We wanted the quickest / easiest way to get the registry going but didn't know where to start. This information is all out there but not in one place (that I found).

Basically if you've already got apache running on the target server, getting everything going is incredibly simple. If you don't run apache this guide will not useful for you.

TLS
---

TLS [is mandatory][docker-docs-remote] when running a remote docker registry. We used [letsencrypt][letsencrypt-hiw] to obtain our certificates.

If you don't already have one, create an apache vhost with a subdomain for your registry. This is just so [certbot][certbot] will prompt you to generate the certificate for it.

```conf
<VirtualHost *:80>
    ServerAdmin dev@example.io
    ServerName  docker.example.io
</VirtualHost>
```

Install and run [certbot][certbot]. It'll prompt you for the necessary info and configure apache with the certifcates it receives from letsencrypt.

`./path/to/certbot-auto --apache`

VHOST
-----

Thanks to [R.I.Pienaar][rip-git] for this. I was banging my head against the `ProxyPreserveHost` line until I found it on his [blog][rip-blog].

Now you've got the certs, update your registry's vhost. Main things to note:

* using the SSL Cert we've just got from letsencrypt
* Proxying any requests to the registry through to `http://127.0.0.1:5000` where the registry will be running
* Basic auth using `htpasswd`

If you don't want auth (e.g. username:password) for your registry, remove the `<Location>` entries.

```xml
<VirtualHost *:443>

        ServerName docker.example.io
        ServerAdmin dev@example.io

        SSLEngine On
        SSLCertificateFile /etc/letsencrypt/live/client.example.io/cert.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/client.example.io/privkey.pem
        SSLCertificateChainFile /etc/letsencrypt/live/client.example.io/chain.pem

        ProxyPreserveHost on
        ProxyPass / http://127.0.0.1:5000/
        ProxyPassReverse / http://127.0.0.1:5000/

        <Location />
                Order deny,allow
                Allow from all

                AuthName "Registry Authentication"
                AuthType basic
                AuthUserFile "/opt/registry-htpasswd/.htpasswd"
                Require valid-user
        </Location>

        # Allow ping and users to run unauthenticated.
        <Location /v1/_ping>
                Satisfy any
                Allow from all
        </Location>

        # Allow ping and users to run unauthenticated.
        <Location /_ping>
               Satisfy any
               Allow from all
        </Location>

</VirtualHost>
```

htpasswd
---------

Skip if you don't want the auth.

<div class="highlighter-rouge">
<pre class="lowlight">
<code>mkdir /opt/registry-htpasswd
cd /opt/registry-htpasswd
htpasswd -bc .htpasswd username password</code>
</pre>
</div>

registry storage
----------------

It's [good practice][docker-docs-storage] to specify a storage folder for your registry.

`mkdir /opt/registry-data`

running the registry
--------------------

As we're using Apache for TLS and auth, running the registry is straightforward:

<div class="highlighter-rouge">
<pre class="lowlight">
<code>docker run -d -p 5000:5000 \
    --restart=always \
    --name registry \
    -v /opt/registry-data:/var/lib/registry \
    registry:2</code>
</pre>
</div>

using the registry
------------------

<div class="highlighter-rouge">
<pre class="lowlight">
<code>docker login docker.example.io
docker push docker.example.io/example_image:latest</code>
</pre>
</div>

[docker-docs-remote]:  https://docs.docker.com/registry/deploying/#/running-a-domain-registry
[docker-docs-storage]: https://docs.docker.com/registry/deploying/#/storage
[certbot]:             https://certbot.eff.org/
[letsencrypt-hiw]:     https://letsencrypt.org/how-it-works/
[rip-git]:             https://github.com/ripienaar/
[rip-blog]:            https://www.devco.net/archives/2015/01/21/running-a-secure-docker-registry-behind-apache.php