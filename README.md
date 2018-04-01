# wp.box – A Lightweight WordPress Box

![](https://cdn.rawgit.com/kadimi/wp.box/master/banner.png "wp.box banner")

## TLDR;

```shell
git clone https://github.com/kadimi/wp.box.git && cd wp.box && vagrant up
```
Then visit http://wp.box/.

## Create a wp.box Box

1. **Clone this repository**

  ```shell
  git clone https://github.com/kadimi/wp.box.git awesome.box
  ```
2. **Change directory**

  ```shell
  cd awesome.box
  ```
3. **Bring machine up**

  ```shell
  # # For http://wp.box/
  # vagrant up

  # For http://awesome.box/, provide a hostname with the first `vagrant up`
  hostname='awesome.box' vagrant up
  ```
4. **Visit website**

  URL is http://wp.box/ unless you provided a hostname, e.g. http://awesome.box/.

## Included out of the box

A lot of tools are included, but more importantly:

- Adminer at /adminer.php
- Composer
- PHPUnit
- WP-CLI, `wp` from guest or `./wp` from host
- Xdebug

## WP-CLI

You can run WP-CLI commands directly from the host machine, no need for `vagrant ssh`, here are some examples:

<pre>
<b>$ ./wp plugin list</b>
+---------+----------+--------+---------+
| name    | status   | update | version |
+---------+----------+--------+---------+
| jetpack | inactive | none   | 5.1     |
| lessphp | active   | none   | 2.0.1   |
+---------+----------+--------+---------+
Connection to 127.0.0.1 closed.

<b>$ ./wp plugin search git</b>
Success: Showing 10 of 1325 plugins.
+------------------------+-----------------------+--------+
| name                   | slug                  | rating |
+------------------------+-----------------------+--------+
| WordPress  GitHub Sync | wp-github-sync        | 100    |
| Event List             | event-list            | 92     |
| ...                    | ...                   | ...    |
+------------------------+-----------------------+--------+
Connection to 127.0.0.1 closed.
</pre>

## Administrators

- **WordPress**
  - Username is `admin`
  - Password is `password`

- **Database**
  - Username is `root`
  - Password is `root`

## Miscellaneous

- A random port is chosen on the first boot from `8800` to `8899` and is stored in a file named `port`. The port is forwarded to port `80` on the guest machine. This comes in handly if you want to show your website over the Internet when you are behind a rooter (you will probably be using No-IP, Dyn or similar).

## Planned

### Automated backups

Something like this running periodically:

```bash
#!/usr/bin/env bash

D='/var/www/public/wp-content/backup';
mkdir -p $D;
wp db export $D/"`date +"db-%s-%F-%H-%M-%S.sql"`" --add-drop-table;
```
