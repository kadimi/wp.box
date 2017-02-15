# `wp.dev` a Lightweight WordPress Box

### TLDR;

```shell
git clone https://github.com/kadimi/wp.dev.git && cd wp.dev && vagrant up
```
Then visit http://wp.dev/ or http://192.168.33.10/.

### Create a `wp.dev` Box

1. **Clone this repository**

  ```shell
  git clone https://github.com/kadimi/wp.dev.git awesome.box
  ```
1. **Change directory**

  ```shell
  cd awesome.box
  ```
1. **Bring machine up**

  ```shell
  # For http://wp.dev/
  vagrant up

  # # For http://awesome.box/, provide a hostname with the first `vagrant up`
  # hostname='awesome.box' vagrant up
  ```
1. **[Maybe](https://github.com/cogitatio/vagrant-hostsupdater "The Vagrant::Hostsupdater plugin adds an entry to your /etc/hosts file on the host system.") update `/etc/hosts`**

  ```
  192.168.33.10  wp.dev
  # 192.168.33.10  awesome.box # Or whatever hostname you provided.
  ```

1. **Visit website**

  URL is http://wp.dev/ or whatever hostname you provided.

### Included out of the box

A lot of tools are included, but more importantly:

- Adminer (located at `.../adminer.php`)
- Composer
- WP-CLI

### Administrators

- **WordPress**
  - Username is `admin`
  - Password is `password`
  
- **Database**
  - Username is `root`
  - Password is `root`
