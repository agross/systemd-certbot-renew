# systemd-certbot-renew

Renew certbot certificates and after new certificates have been obtained,
optionally

* update [cockpit](http://cockpit-project.org/) certificate,
* commit new certificate within [etckeeper](https://github.com/joeyh/etckeeper)
  repository,
* restart [nginx](http://nginx.org/) or any other web server run as a systemd
  unit.

## Installation

1. Decide where you want to install things.

   ```sh
   INSTALL_DIR=/usr/local/src/systemd-certbot-renew
   ```

1. Clone this repository.

   ```sh
   $ git clone https://github.com/agross/systemd-certbot-renew.git "$INSTALL_DIR"
   Cloning into 'systemd-certbot-renew'...
   ```

1. Install `certbot-renew` systemd service and timer unit.

   ```sh
   $ "$INSTALL_DIR/install"
   Linking /usr/local/src/data/systemd-certbot-renew/certbot.service
   Created symlink /etc/systemd/system/certbot.service → /usr/local/src/data/systemd-certbot-renew/certbot.service.
   Linking /usr/local/src/data/systemd-certbot-renew/certbot.timer
   Created symlink /etc/systemd/system/certbot.timer → /usr/local/src/data/systemd-certbot-renew/certbot.timer.
   Enabling /usr/local/src/data/systemd-certbot-renew/certbot.timer
   Created symlink /etc/systemd/system/timers.target.wants/certbot.timer → /usr/local/src/data/systemd-certbot-renew/certbot.timer.
   ```

1. Tell `certbot-renew` about your domain's certificate to be used as the
   cockpit certificate and web server systemd unit.

   ```sh
   $ systemctl edit certbot.service

   # Editor opens. Add these lines and save the file.

   [Service]
   # This causes /etc/letsencrypt/live/example.com/fullchain.pem + privkey
   # to be copied to /etc/cockpit/ws-cert.d/letsencrypt.cert.
   # After that, changes are committed if /etc/cockpit/ws-cert.d is under git.
   Environment='COCKPIT_DOMAIN=-d example.com'

   # Optional, nginx.service by default.
   Environment='WEBSERVER=-w nginx.service'
   # Do not reload web server.
   Environment=WEBSERVER=
   ```

1. Start `certbot.timer`.
   ```sh
   $ systemctl start certbot.timer
   ```

## Optional configuration items to think about

### Run Let's Encrypt renewal manually

```sh
$ "$INSTALL_DIR/certbot-renew" -h
Usage: certbot-renew [-d DOMAIN] [-w WEBSERVER] [-k]

Renew Let's Encrypt certificates and optionally update cockpit certificate and restart web server.

  -d DOMAIN     domain name of the Let's Encrypt certificate for cockpit
  -w WEBSERVER  name of the web server systemd unit to be restarted
  -k            run certbot post-update actions (committing, cockpit, web server)
```

### Get notified when certificate update fails

You can tell systemd to start another unit whenever a unit fails. See
[this blog post](http://northernlightlabs.se/systemd.status.mail.on.unit.failure)
for how it is done. Also see my
[systemd-unit-status-mail](https://github.com/agross/systemd-unit-status-mail)
repository that contains everything you need.

After `unit-status-mail` is in place, add it to your `certbot.service` using a
systemd override:

```sh
$ systemctl edit certbot.service

# Editor opens. Add these lines and save the file.

[Unit]
OnFailure=unit-status-mail@%n.service
```

### Getting the whole picture

To get all `certbot.service` and `certbot.timer` settings use

* `systemctl cat certbot.service` or `systemctl cat certbot.timer` and
* `systemctl show certbot.service` or `systemctl show certbot.timer`.
