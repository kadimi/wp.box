# wp.dev – A Lightweight WordPress Box

![](https://cdn.rawgit.com/kadimi/wp.dev/master/banner.png "wp.dev banner")

## TLDR;

```shell
git clone https://github.com/kadimi/wp.dev.git && cd wp.dev && vagrant up
```
Then visit http://wp.dev/.

## Create a wp.dev Box

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
1. **Visit website**

  URL is http://wp.dev/ or whatever hostname you provided.

## Included out of the box

A lot of tools are included, but more importantly:

- Adminer (located at `.../adminer.php`)
- Composer
- WP-CLI

## Administrators

- **WordPress**
  - Username is `admin`
  - Password is `password`
  
- **Database**
  - Username is `root`
  - Password is `root`
