# What is DeskPRO?

DeskPRO is the modern helpdesk platform. It offers a ticket system, user help portal, real-time chat, CRM, crowdsourced feedback, file-hosting and more. The extensible platform integrates easily with your existing email accounts and website, your SSO infrastructure and third-party services. Unparalleled usability and flexible configuration mean that your agents can concentrate on helping users.

[DeskPRO.com](https://www.deskpro.com/)

![logo](https://www.deskpro.com/assets/build/img/deskpro/logo.png)

# How to use this image

You can choose from the two available variants: `apache` or `fpm`. The first comes with Apache preinstalled, while the second one runs `php-fpm`, which means you'll need something that speaks FCGI (such as [nginx](https://hub.docker.com/_/nginx/))  in front of it.

In order to run DeskPRO, you'll need to run two containers from this image: one for the web interface and one for the periodic tasks. They have to share a volume containing the DeskPRO codebase. If running manually, it can look like the following:

```console
$ docker volume create deskpro
$ docker run [...] --volume deskpro:/var/www/html deskpro/deskpro:apache
$ docker run [...] --volume deskpro:/var/www/html deskpro/deskpro:apache deskpro-docker-cron
```

It's also required to set a few environment variables:

* `MYSQL_HOST`: the hostname/IP address of the desired MySQL database host
* `MYSQL_USER`: the username to connect to MySQL as
* `MYSQL_PASSWORD`: the password for the username described above
* `MYSQL_DATABASE`: the database name to connect to

Please ensure that the target database is empty, otherwise DeskPRO won't perform the installation to avoid overwriting any data.

The easiest way to try this image is with a `docker-compose.yml` file such as below:

```yaml
---
version: '2'
services:
  deskpro:
    image: deskpro/deskpro:apache
    depends_on:
      - mariadb
    environment:
      MYSQL_HOST: mariadb
      MYSQL_USER: deskprouser
      MYSQL_PASSWORD: deskpropassword
      MYSQL_DATABASE: deskprodb
    ports:
      - "80:80"
    volumes:
      - deskpro:/var/www/html
  cron:
    image: deskpro/deskpro:apache
    command: deskpro-docker-cron
    depends_on:
      - deskpro
      - mariadb
    volumes:
      - deskpro:/var/www/html
  mariadb:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: mysqlrootpassword
      MYSQL_DATABASE: deskprodb
      MYSQL_USER: deskprouser
      MYSQL_PASSWORD: deskpropassword
volumes:
  deskpro:
```

## Testing

If you wish to work on the image itself, there's two files that'll allow you to run Deskpro with `docker-compose`:

- `docker-compose-test-fpm.yml`: uses php-fpm and nginx
- `docker-compose-test-apache.yml`: uses Apache

Use them with `docker-compose -f filename.yml`.

For actual usage of the image, check the previous section.
