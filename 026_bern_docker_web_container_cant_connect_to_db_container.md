# "Bern": Docker web container can't connect to db container.

**Description:** There are two Docker containers running, a web application (Wordpress or WP) and a database (MariaDB) as back-end, but if we look at the web page, we see that it cannot connect to the database. `curl -s localhost:80 |tail -4` returns:  
```html
<body id="error-page"> <div class="wp-die-message"><h1>Error establishing a database connection</h1></div></body> </html>
```  
This is not a Wordpress code issue (the image is `:latest` with some network utilities added). What you need to know is that WP uses "WORDPRESS_DB_" environment variables to create the MySQL connection string. See the `./html/wp-config.php` WP config file for example (from `/home/admin`).  

**Test:** `sudo docker exec wordpress mysqladmin -h mysql -u root -ppassword ping`. The wordpress container is able to connect to the database in the mariadb container and returns `mysqld is alive`.  


---

### Solution:
#### 1. Reconnaissance on the server
`cd /home/admin/ && ls -la`  
```console
total 44
drwxr-xr-x 7 admin            admin            4096 Aug 31  2022 .
drwxr-xr-x 3 root             root             4096 Aug  4  2022 ..
-rw------- 1 admin            admin               5 Aug 31  2022 .bash_history
-rw-r--r-- 1 admin            admin             220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin            admin            3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin            admin            4096 Aug 31  2022 .local
-rw-r--r-- 1 admin            admin             807 Aug  4  2021 .profile
drwx------ 2 admin            admin            4096 Aug  4  2022 .ssh
drwxr-xr-x 2 admin            admin            4096 Aug 31  2022 agent
drwxr-xr-x 6 systemd-timesync systemd-timesync 4096 Aug  4  2022 database
drwxr-xr-x 5 www-data         www-data         4096 Aug  4  2022 html
```

`ls -la html/`  
```console
total 244
drwxr-xr-x  5 www-data www-data  4096 Aug  4  2022 .
drwxr-xr-x  7 admin    admin     4096 Feb  5 11:31 ..
-rw-r--r--  1 www-data www-data   523 Aug  4  2022 .htaccess
-rw-r--r--  1 www-data www-data   405 Feb  6  2020 index.php
-rw-r--r--  1 www-data www-data 19915 Jan  1  2022 license.txt
-rw-r--r--  1 www-data www-data  7401 Mar 22  2022 readme.html
-rw-r--r--  1 www-data www-data  7165 Jan 21  2021 wp-activate.php
drwxr-xr-x  9 www-data www-data  4096 Jul 12  2022 wp-admin
-rw-r--r--  1 www-data www-data   351 Feb  6  2020 wp-blog-header.php
-rw-r--r--  1 www-data www-data  2338 Nov  9  2021 wp-comments-post.php
-rw-rw-r--  1 www-data www-data  5480 Aug  3  2022 wp-config-docker.php
-rw-r--r--  1 www-data www-data  3001 Dec 14  2021 wp-config-sample.php
-rw-r--r--  1 www-data www-data  5584 Aug  4  2022 wp-config.php
drwxr-xr-x  5 www-data www-data  4096 Aug  4  2022 wp-content
-rw-r--r--  1 www-data www-data  3943 Apr 28  2022 wp-cron.php
drwxr-xr-x 26 www-data www-data 16384 Jul 12  2022 wp-includes
-rw-r--r--  1 www-data www-data  2494 Mar 19  2022 wp-links-opml.php
-rw-r--r--  1 www-data www-data  3973 Apr 12  2022 wp-load.php
-rw-r--r--  1 www-data www-data 48498 Apr 29  2022 wp-login.php
-rw-r--r--  1 www-data www-data  8577 Mar 22  2022 wp-mail.php
-rw-r--r--  1 www-data www-data 23706 Apr 12  2022 wp-settings.php
-rw-r--r--  1 www-data www-data 32051 Apr 11  2022 wp-signup.php
-rw-r--r--  1 www-data www-data  4748 Apr 11  2022 wp-trackback.php
-rw-r--r--  1 www-data www-data  3236 Jun  8  2020 xmlrpc.php
```

`ls -la database/`  
```console
total 126968
drwxr-xr-x 6 systemd-timesync systemd-timesync      4096 Aug  4  2022 .
drwxr-xr-x 7 admin            admin                 4096 Feb  5 11:31 ..
-rw-rw---- 1 systemd-timesync systemd-timesync  16719872 Aug  4  2022 aria_log.00000001
-rw-rw---- 1 systemd-timesync systemd-timesync        52 Aug  4  2022 aria_log_control
-rw-rw---- 1 systemd-timesync systemd-timesync      2028 Aug  4  2022 ib_buffer_pool
-rw-rw---- 1 systemd-timesync systemd-timesync 100663296 Aug  4  2022 ib_logfile0
-rw-rw---- 1 systemd-timesync systemd-timesync  12582912 Aug  4  2022 ibdata1
-rw-rw---- 1 systemd-timesync systemd-timesync         0 Aug  4  2022 multi-master.info
drwx------ 2 systemd-timesync systemd-timesync      4096 Aug  4  2022 mysql
-rw-r--r-- 1 systemd-timesync systemd-timesync        14 Aug  4  2022 mysql_upgrade_info
drwx------ 2 systemd-timesync systemd-timesync      4096 Aug  4  2022 performance_schema
drwx------ 2 systemd-timesync systemd-timesync     12288 Aug  4  2022 sys
drwx------ 2 systemd-timesync systemd-timesync      4096 Aug  4  2022 wordpress
```

`cat ./agent/check.sh `  
```bash
#!/usr/bin/bash
res=$(sudo docker exec wordpress mysqladmin -h mysql -u root -ppassword --connect-timeout 2 ping)
res=$(echo $res|tr -d '\r')

if [[ "$res" = "mysqld is alive" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`./agent/check.sh `  
```console
mysqladmin: connect to server at 'mysql' failed
error: 'Unknown MySQL server host 'mysql' (-2)'
Check that mysqld is running on mysql and that the port is 3306.
You can check this by doing 'telnet mysql 3306'
```

`curl -s localhost:80 |tail -4`  
```html
<body id="error-page">
        <div class="wp-die-message"><h1>Error establishing a database connection</h1></div></body>
</html>
```

`mysql`  
```console
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/run/mysqld/mysqld.sock' (2)
```

<details>

  <summary>cat ./html/wp-config.php</summary>

```console
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * This has been slightly modified (to read environment variables) for use in Docker.
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// IMPORTANT: this file needs to stay in-sync with https://github.com/WordPress/WordPress/blob/master/wp-config-sample.php
// (it gets parsed by the upstream wizard in https://github.com/WordPress/WordPress/blob/f27cb65e1ef25d11b535695a660e7282b98eb742/wp-admin/setup-config.php#L356-L392)

// a helper function to lookup "env_FILE", "env", then fallback
if (!function_exists('getenv_docker')) {
        // https://github.com/docker-library/wordpress/issues/588 (WP-CLI will load this file 2x)
        function getenv_docker($env, $default) {
                if ($fileEnv = getenv($env . '_FILE')) {
                        return rtrim(file_get_contents($fileEnv), "\r\n");
                }
                else if (($val = getenv($env)) !== false) {
                        return $val;
                }
                else {
                        return $default;
                }
        }
}

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', getenv_docker('WORDPRESS_DB_NAME', 'wordpress') );

/** Database username */
define( 'DB_USER', getenv_docker('WORDPRESS_DB_USER', 'example username') );

/** Database password */
define( 'DB_PASSWORD', getenv_docker('WORDPRESS_DB_PASSWORD', 'example password') );

/**
 * Docker image fallback values above are sourced from the official WordPress installation wizard:
 * https://github.com/WordPress/WordPress/blob/f9cc35ebad82753e9c86de322ea5c76a9001c7e2/wp-admin/setup-config.php#L216-L230
 * (However, using "example username" and "example password" in your database is strongly discouraged.  Please use strong, random credentials!)
 */

/** Database hostname */
define( 'DB_HOST', getenv_docker('WORDPRESS_DB_HOST', 'mysql') );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', getenv_docker('WORDPRESS_DB_CHARSET', 'utf8') );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', getenv_docker('WORDPRESS_DB_COLLATE', '') );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         getenv_docker('WORDPRESS_AUTH_KEY',         'cb590d03fd0c0f92ed218c9da9b82608fdce5f10') );
define( 'SECURE_AUTH_KEY',  getenv_docker('WORDPRESS_SECURE_AUTH_KEY',  'cac6899b8e895c3c700873250a466d4d1ef18e02') );
define( 'LOGGED_IN_KEY',    getenv_docker('WORDPRESS_LOGGED_IN_KEY',    '12598f339422cbf1de5b5737ad38b1bccd52edea') );
define( 'NONCE_KEY',        getenv_docker('WORDPRESS_NONCE_KEY',        '8a2a9e8f765e7e11e6ab684c05e60636ae476f50') );
define( 'AUTH_SALT',        getenv_docker('WORDPRESS_AUTH_SALT',        '78ac5204181f05a35591e321993bda5a2add6b8d') );
define( 'SECURE_AUTH_SALT', getenv_docker('WORDPRESS_SECURE_AUTH_SALT', '019b45076411769976199712e837dc1f2334b66d') );
define( 'LOGGED_IN_SALT',   getenv_docker('WORDPRESS_LOGGED_IN_SALT',   '9c8308b24ec87fd97ad38ca23797c326d448cc41') );
define( 'NONCE_SALT',       getenv_docker('WORDPRESS_NONCE_SALT',       '77b505d87e096da3f9269054d8f23c82533092fd') );
// (See also https://wordpress.stackexchange.com/a/152905/199287)

/**#@-*/

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = getenv_docker('WORDPRESS_TABLE_PREFIX', 'wp_');

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', !!getenv_docker('WORDPRESS_DEBUG', '') );

/* Add any custom values between this line and the "stop editing" line. */

// If we're behind a proxy server and using HTTPS, we need to alert WordPress of that fact
// see also https://wordpress.org/support/article/administration-over-ssl/#using-a-reverse-proxy
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && strpos($_SERVER['HTTP_X_FORWARDED_PROTO'], 'https') !== false) {
        $_SERVER['HTTPS'] = 'on';
}
// (we include this by default because reverse proxying is extremely common in container environments)

if ($configExtra = getenv_docker('WORDPRESS_CONFIG_EXTRA', '')) {
        eval($configExtra);
}

/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

</details>

`sudo docker ps`  
```console
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                    NAMES
6ffb084b515c   wordpress:sad    "docker-entrypoint.s…"   18 months ago   Up 4 minutes   0.0.0.0:80->80/tcp       wordpress
0eef97284c44   mariadb:latest   "docker-entrypoint.s…"   18 months ago   Up 4 minutes   0.0.0.0:3306->3306/tcp   mariadb
```

