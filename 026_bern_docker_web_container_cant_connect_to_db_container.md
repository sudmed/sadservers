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


<details>

  <summary>sudo docker logs 0eef97284c4</summary>

```console
2022-08-04 03:22:33+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2022-08-04 03:22:34+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-08-04 03:22:34+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2022-08-04 03:22:34+00:00 [Note] [Entrypoint]: MariaDB upgrade not required
2022-08-04  3:22:34 0 [Note] mariadbd (server 10.8.3-MariaDB-1:10.8.3+maria~jammy) starting as process 1 ...
2022-08-04  3:22:34 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2022-08-04  3:22:34 0 [Note] InnoDB: Number of transaction pools: 1
2022-08-04  3:22:34 0 [Note] InnoDB: Using crc32 + pclmulqdq instructions
2022-08-04  3:22:34 0 [Note] mariadbd: O_TMPFILE is not supported on /tmp (disabling future attempts)
2022-08-04  3:22:34 0 [Warning] mariadbd: io_uring_queue_init() failed with ENOMEM: try larger memory locked limit, ulimit -l, or https://mariadb.com/kb/en/systemd/#configuring-limitmemlock under systemd (262144 bytes required)
2022-08-04  3:22:34 0 [Warning] InnoDB: liburing disabled: falling back to innodb_use_native_aio=OFF
2022-08-04  3:22:34 0 [Note] InnoDB: Initializing buffer pool, total size = 128.000MiB, chunk size = 2.000MiB
2022-08-04  3:22:34 0 [Note] InnoDB: Completed initialization of buffer pool
2022-08-04  3:22:34 0 [Note] InnoDB: File system buffers for log disabled (block size=512 bytes)
2022-08-04  3:22:34 0 [Note] InnoDB: 128 rollback segments are active.
2022-08-04  3:22:34 0 [Note] InnoDB: Setting file './ibtmp1' size to 12.000MiB. Physically writing the file full; Please wait ...
2022-08-04  3:22:34 0 [Note] InnoDB: File './ibtmp1' size is now 12.000MiB.
2022-08-04  3:22:34 0 [Note] InnoDB: log sequence number 46827; transaction id 14
2022-08-04  3:22:34 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
2022-08-04  3:22:34 0 [Note] Plugin 'FEEDBACK' is disabled.
2022-08-04  3:22:34 0 [Warning] You need to use --log-bin to make --expire-logs-days or --binlog-expire-logs-seconds work.
2022-08-04  3:22:34 0 [Note] InnoDB: Buffer pool(s) load completed at 220804  3:22:34
2022-08-04  3:22:34 0 [Note] Server socket created on IP: '0.0.0.0'.
2022-08-04  3:22:34 0 [Note] Server socket created on IP: '::'.
2022-08-04  3:22:34 0 [Note] mariadbd: ready for connections.
Version: '10.8.3-MariaDB-1:10.8.3+maria~jammy'  socket: '/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution
2022-08-04  3:23:49 0 [Note] mariadbd (initiated by: unknown): Normal shutdown
2022-08-04  3:23:49 0 [Note] InnoDB: FTS optimize thread exiting.
2022-08-04  3:23:49 0 [Note] InnoDB: Starting shutdown...
2022-08-04  3:23:49 0 [Note] InnoDB: Dumping buffer pool(s) to /var/lib/mysql/ib_buffer_pool
2022-08-04  3:23:49 0 [Note] InnoDB: Buffer pool(s) dump completed at 220804  3:23:49
2022-08-04  3:23:49 0 [Note] InnoDB: Removed temporary tablespace data file: "./ibtmp1"
2022-08-04  3:23:49 0 [Note] InnoDB: Shutdown completed; log sequence number 46827; transaction id 15
2022-08-04  3:23:49 0 [Note] mariadbd: Shutdown complete

2022-08-04 03:24:53+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2022-08-04 03:24:54+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-08-04 03:24:54+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2022-08-04 03:24:54+00:00 [Note] [Entrypoint]: MariaDB upgrade not required
2022-08-04  3:24:54 0 [Note] mariadbd (server 10.8.3-MariaDB-1:10.8.3+maria~jammy) starting as process 1 ...
2022-08-04  3:24:54 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2022-08-04  3:24:54 0 [Note] InnoDB: Number of transaction pools: 1
2022-08-04  3:24:54 0 [Note] InnoDB: Using crc32 + pclmulqdq instructions
2022-08-04  3:24:54 0 [Note] mariadbd: O_TMPFILE is not supported on /tmp (disabling future attempts)
2022-08-04  3:24:54 0 [Warning] mariadbd: io_uring_queue_init() failed with ENOMEM: try larger memory locked limit, ulimit -l, or https://mariadb.com/kb/en/systemd/#configuring-limitmemlock under systemd (262144 bytes required)
2022-08-04  3:24:54 0 [Warning] InnoDB: liburing disabled: falling back to innodb_use_native_aio=OFF
2022-08-04  3:24:54 0 [Note] InnoDB: Initializing buffer pool, total size = 128.000MiB, chunk size = 2.000MiB
2022-08-04  3:24:54 0 [Note] InnoDB: Completed initialization of buffer pool
2022-08-04  3:24:54 0 [Note] InnoDB: File system buffers for log disabled (block size=512 bytes)
2022-08-04  3:24:54 0 [Note] InnoDB: 128 rollback segments are active.
2022-08-04  3:24:54 0 [Note] InnoDB: Setting file './ibtmp1' size to 12.000MiB. Physically writing the file full; Please wait ...
2022-08-04  3:24:54 0 [Note] InnoDB: File './ibtmp1' size is now 12.000MiB.
2022-08-04  3:24:54 0 [Note] InnoDB: log sequence number 46827; transaction id 14
2022-08-04  3:24:54 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
2022-08-04  3:24:54 0 [Note] Plugin 'FEEDBACK' is disabled.
2022-08-04  3:24:54 0 [Warning] You need to use --log-bin to make --expire-logs-days or --binlog-expire-logs-seconds work.
2022-08-04  3:24:54 0 [Note] Server socket created on IP: '0.0.0.0'.
2022-08-04  3:24:54 0 [Note] Server socket created on IP: '::'.
2022-08-04  3:24:54 0 [Note] InnoDB: Buffer pool(s) load completed at 220804  3:24:54
2022-08-04  3:24:54 0 [Note] mariadbd: ready for connections.
Version: '10.8.3-MariaDB-1:10.8.3+maria~jammy'  socket: '/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution
2022-08-04  3:31:49 0 [Note] mariadbd (initiated by: unknown): Normal shutdown
2022-08-04  3:31:49 0 [Note] InnoDB: FTS optimize thread exiting.
2022-08-04  3:31:49 0 [Note] InnoDB: Starting shutdown...
2022-08-04  3:31:49 0 [Note] InnoDB: Dumping buffer pool(s) to /var/lib/mysql/ib_buffer_pool
2022-08-04  3:31:49 0 [Note] InnoDB: Buffer pool(s) dump completed at 220804  3:31:49
2022-08-04  3:31:49 0 [Note] InnoDB: Removed temporary tablespace data file: "./ibtmp1"
2022-08-04  3:31:49 0 [Note] InnoDB: Shutdown completed; log sequence number 46827; transaction id 15
2022-08-04  3:31:49 0 [Note] mariadbd: Shutdown complete

2022-08-31 15:27:39+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2022-08-31 15:27:45+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-08-31 15:27:46+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2022-08-31 15:27:46+00:00 [Note] [Entrypoint]: MariaDB upgrade not required
2022-08-31 15:27:46 0 [Note] mariadbd (server 10.8.3-MariaDB-1:10.8.3+maria~jammy) starting as process 1 ...
2022-08-31 15:27:46 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2022-08-31 15:27:46 0 [Note] InnoDB: Number of transaction pools: 1
2022-08-31 15:27:46 0 [Note] InnoDB: Using crc32 + pclmulqdq instructions
2022-08-31 15:27:46 0 [Note] mariadbd: O_TMPFILE is not supported on /tmp (disabling future attempts)
2022-08-31 15:27:46 0 [Warning] mariadbd: io_uring_queue_init() failed with ENOMEM: try larger memory locked limit, ulimit -l, or https://mariadb.com/kb/en/systemd/#configuring-limitmemlock under systemd (262144 bytes required)
2022-08-31 15:27:46 0 [Warning] InnoDB: liburing disabled: falling back to innodb_use_native_aio=OFF
2022-08-31 15:27:46 0 [Note] InnoDB: Initializing buffer pool, total size = 128.000MiB, chunk size = 2.000MiB
2022-08-31 15:27:46 0 [Note] InnoDB: Completed initialization of buffer pool
2022-08-31 15:27:46 0 [Note] InnoDB: File system buffers for log disabled (block size=512 bytes)
2022-08-31 15:27:46 0 [Note] InnoDB: 128 rollback segments are active.
2022-08-31 15:27:46 0 [Note] InnoDB: Setting file './ibtmp1' size to 12.000MiB. Physically writing the file full; Please wait ...
2022-08-31 15:27:46 0 [Note] InnoDB: File './ibtmp1' size is now 12.000MiB.
2022-08-31 15:27:46 0 [Note] InnoDB: log sequence number 46827; transaction id 14
2022-08-31 15:27:46 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
2022-08-31 15:27:46 0 [Note] Plugin 'FEEDBACK' is disabled.
2022-08-31 15:27:46 0 [Warning] You need to use --log-bin to make --expire-logs-days or --binlog-expire-logs-seconds work.
2022-08-31 15:27:46 0 [Note] Server socket created on IP: '0.0.0.0'.
2022-08-31 15:27:46 0 [Note] Server socket created on IP: '::'.
2022-08-31 15:27:46 0 [Note] InnoDB: Buffer pool(s) load completed at 220831 15:27:46
2022-08-31 15:27:46 0 [Note] mariadbd: ready for connections.
Version: '10.8.3-MariaDB-1:10.8.3+maria~jammy'  socket: '/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution
2022-08-31 15:40:19 0 [Note] mariadbd (initiated by: unknown): Normal shutdown
2022-08-31 15:40:19 0 [Note] InnoDB: FTS optimize thread exiting.
2022-08-31 15:40:19 0 [Note] InnoDB: Starting shutdown...
2022-08-31 15:40:19 0 [Note] InnoDB: Dumping buffer pool(s) to /var/lib/mysql/ib_buffer_pool
2022-08-31 15:40:19 0 [Note] InnoDB: Buffer pool(s) dump completed at 220831 15:40:19
2022-08-31 15:40:19 0 [Note] InnoDB: Removed temporary tablespace data file: "./ibtmp1"
2022-08-31 15:40:19 0 [Note] InnoDB: Shutdown completed; log sequence number 46827; transaction id 15
2022-08-31 15:40:19 0 [Note] mariadbd: Shutdown complete

2024-02-05 11:23:32+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2024-02-05 11:23:33+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2024-02-05 11:23:33+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.8.3+maria~jammy started.
2024-02-05 11:23:33+00:00 [Note] [Entrypoint]: MariaDB upgrade not required
2024-02-05 11:23:33 0 [Note] mariadbd (server 10.8.3-MariaDB-1:10.8.3+maria~jammy) starting as process 1 ...
2024-02-05 11:23:33 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2024-02-05 11:23:33 0 [Note] InnoDB: Number of transaction pools: 1
2024-02-05 11:23:33 0 [Note] InnoDB: Using crc32 + pclmulqdq instructions
2024-02-05 11:23:33 0 [Note] mariadbd: O_TMPFILE is not supported on /tmp (disabling future attempts)
2024-02-05 11:23:33 0 [Warning] mariadbd: io_uring_queue_init() failed with ENOMEM: try larger memory locked limit, ulimit -l, or https://mariadb.com/kb/en/systemd/#configuring-limitmemlock under systemd (262144 bytes required)
2024-02-05 11:23:33 0 [Warning] InnoDB: liburing disabled: falling back to innodb_use_native_aio=OFF
2024-02-05 11:23:33 0 [Note] InnoDB: Initializing buffer pool, total size = 128.000MiB, chunk size = 2.000MiB
2024-02-05 11:23:33 0 [Note] InnoDB: Completed initialization of buffer pool
2024-02-05 11:23:33 0 [Note] InnoDB: File system buffers for log disabled (block size=512 bytes)
2024-02-05 11:23:33 0 [Note] InnoDB: 128 rollback segments are active.
2024-02-05 11:23:33 0 [Note] InnoDB: Setting file './ibtmp1' size to 12.000MiB. Physically writing the file full; Please wait ...
2024-02-05 11:23:33 0 [Note] InnoDB: File './ibtmp1' size is now 12.000MiB.
2024-02-05 11:23:33 0 [Note] InnoDB: log sequence number 46827; transaction id 14
2024-02-05 11:23:33 0 [Note] Plugin 'FEEDBACK' is disabled.
2024-02-05 11:23:33 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
2024-02-05 11:23:33 0 [Warning] You need to use --log-bin to make --expire-logs-days or --binlog-expire-logs-seconds work.
2024-02-05 11:23:33 0 [Note] InnoDB: Buffer pool(s) load completed at 240205 11:23:33
2024-02-05 11:23:33 0 [Note] Server socket created on IP: '0.0.0.0'.
2024-02-05 11:23:33 0 [Note] Server socket created on IP: '::'.
2024-02-05 11:23:33 0 [Note] mariadbd: ready for connections.
Version: '10.8.3-MariaDB-1:10.8.3+maria~jammy'  socket: '/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution
```

</details>


<details>

  <summary>sudo docker inspect wordpress</summary>

```console
[
    {
        "Id": "6ffb084b515ca482ac58fad406b10837b44fb55610acbb35b8ed4a0fb24de50c",
        "Created": "2022-08-04T03:22:49.885388997Z",
        "Path": "docker-entrypoint.sh",
        "Args": [
            "apache2-foreground"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 900,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2024-02-05T11:23:32.617312869Z",
            "FinishedAt": "2022-08-31T15:40:20.366876609Z"
        },
        "Image": "sha256:0a3bdd32ae210e34ed07cf49669dccbdb9deac143ebf27d5cc83136a0e3d9063",
        "ResolvConfPath": "/var/lib/docker/containers/6ffb084b515ca482ac58fad406b10837b44fb55610acbb35b8ed4a0fb24de50c/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/6ffb084b515ca482ac58fad406b10837b44fb55610acbb35b8ed4a0fb24de50c/hostname",
        "HostsPath": "/var/lib/docker/containers/6ffb084b515ca482ac58fad406b10837b44fb55610acbb35b8ed4a0fb24de50c/hosts",
        "LogPath": "/var/lib/docker/containers/6ffb084b515ca482ac58fad406b10837b44fb55610acbb35b8ed4a0fb24de50c/6ffb084b515ca482ac58fad406b10837b44fb55610acbb35b8ed4a0fb24de50c-json.log",
        "Name": "/wordpress",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "html:/var/www/html"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "80/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "80"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "always",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/ce38c2ce47ad7d4bbbd8f5c7c4ed567023d2121aef7d943d0fa2550593cfbeea-init/diff:/var/lib/docker/overlay2/94106dc67f8c47bdc7f21b7607cae55e2cf94644b34d7698b3e174a3755275a6/diff:/var/lib/docker/overlay2/2936726c07d706d506728ed14cf8cab6e23916647e0838f0f79f86552c7010b3/diff:/var/lib/docker/overlay2/069acc5fa9bb5677b2aeb93fc0ef5773a23d6d7b1218cf75ef82aca71c6bda47/diff:/var/lib/docker/overlay2/c8afefd30aa9208d8c1d54d5fd332fc36608816b4dc77f94c1137c882a4ec821/diff:/var/lib/docker/overlay2/420654ce2976260a0830db88a78d5a347fb80aa7b790b2d9309ec7ae76d91d83/diff:/var/lib/docker/overlay2/422f6df2f56d58836998a749e631cab9406fd37800a3d4f7e4eda1f14f008d08/diff:/var/lib/docker/overlay2/3dfa1839ce3bd0b607fcddd25305125d66c7d2870a6e78dc5a01ba82134e1c42/diff:/var/lib/docker/overlay2/acdc673318ff934a6a1735e4a7c3ebfa98ba0b0e1fa7753bed3c36f7be05c5d7/diff:/var/lib/docker/overlay2/3b59127b861d3c1821590e1bbe53361762ec93b9e2471688cf1f16ed75698536/diff:/var/lib/docker/overlay2/3cdfd4f802d2ed04fe9f783aea4f7875b46338caa8d9cc83df921644933a7697/diff:/var/lib/docker/overlay2/c905fa690140ab324cdda21324f759f356b6672242462ddef823f3a8f2456463/diff:/var/lib/docker/overlay2/922e6ceb9fdab2f37d87a09a651497fa659d3c319b013db7427e8eed7487749b/diff:/var/lib/docker/overlay2/904795f943098776cc0a2c8385029bbeb398ae81f6aeedde6f459b7a06130bdd/diff:/var/lib/docker/overlay2/deea2b08a5738a2f6b20e9a9086884a3f181a21408bb140147395e2d3b6b2b88/diff:/var/lib/docker/overlay2/8fc975a0a76589eeb04b246a231d986b411913521b6bedf2e3a5d05908cce7c2/diff:/var/lib/docker/overlay2/0b6935279d515d0669f8a5565cbc0e4bf36dd21bd6632069d3192c34ec4f7d01/diff:/var/lib/docker/overlay2/11de313511ee3247d7543f251bf73d074c52306b38cb468d4406033996bb5579/diff:/var/lib/docker/overlay2/19a9f2c255dac237587a18b8f354abd11410dafc3284412d728c2bf7056313a0/diff:/var/lib/docker/overlay2/04234c51c40e87365dc7808e823f054571bbf2af947366973be0d82abe2d0a60/diff:/var/lib/docker/overlay2/80fe0530020bda727c7a7e05f8261966265ca1a07871152cf5c223d362ec80c3/diff:/var/lib/docker/overlay2/1abcbc54c55abf66822fefb38dbb5816fc59bdb685ced0528db4a3bcf62ebe9b/diff:/var/lib/docker/overlay2/51d8beae216b4b10c5755ae738c4abd12efa13403c8d1c5b0ced1fd44b9b48f3/diff",
                "MergedDir": "/var/lib/docker/overlay2/ce38c2ce47ad7d4bbbd8f5c7c4ed567023d2121aef7d943d0fa2550593cfbeea/merged",
                "UpperDir": "/var/lib/docker/overlay2/ce38c2ce47ad7d4bbbd8f5c7c4ed567023d2121aef7d943d0fa2550593cfbeea/diff",
                "WorkDir": "/var/lib/docker/overlay2/ce38c2ce47ad7d4bbbd8f5c7c4ed567023d2121aef7d943d0fa2550593cfbeea/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "html",
                "Source": "/var/lib/docker/volumes/html/_data",
                "Destination": "/var/www/html",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "6ffb084b515c",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "WORDPRESS_DB_PASSWORD=password",
                "WORDPRESS_DB_USER=root",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "PHPIZE_DEPS=autoconf \t\tdpkg-dev \t\tfile \t\tg++ \t\tgcc \t\tlibc-dev \t\tmake \t\tpkg-config \t\tre2c",
                "PHP_INI_DIR=/usr/local/etc/php",
                "APACHE_CONFDIR=/etc/apache2",
                "APACHE_ENVVARS=/etc/apache2/envvars",
                "PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64",
                "PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64",
                "PHP_LDFLAGS=-Wl,-O1 -pie",
                "GPG_KEYS=42670A7FE4D0441C8E4632349E4FDC074A4EF02D 5A52880781F755608BF815FC910DEB46F53EA312",
                "PHP_VERSION=7.4.30",
                "PHP_URL=https://www.php.net/distributions/php-7.4.30.tar.xz",
                "PHP_ASC_URL=https://www.php.net/distributions/php-7.4.30.tar.xz.asc",
                "PHP_SHA256=ea72a34f32c67e79ac2da7dfe96177f3c451c3eefae5810ba13312ed398ba70d"
            ],
            "Cmd": [
                "apache2-foreground"
            ],
            "Image": "wordpress:sad",
            "Volumes": {
                "/var/www/html": {}
            },
            "WorkingDir": "/var/www/html",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "com.docker.compose.config-hash": "fa7c866c125b6bdbe158555fca49a8d871d32282817db2f396e0ff43d7a93eb4",
                "com.docker.compose.container-number": "1",
                "com.docker.compose.oneoff": "False",
                "com.docker.compose.project": "admin",
                "com.docker.compose.project.config_files": "docker-compose.yaml",
                "com.docker.compose.project.working_dir": "/home/admin",
                "com.docker.compose.service": "wordpress",
                "com.docker.compose.version": "1.25.0"
            },
            "StopSignal": "SIGWINCH"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "15933d22254f177f9ec1717cc15e11628b51802fb573633f9d831fa09e88e67d",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "80"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/15933d22254f",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "0720fd9f938e2052e13d4b0fe73663fbae31b784dea4942e53fa5d96e78982f5",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.3",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:03",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "41e1fb33abcef2772dadcf2943d42ec330cb6bb2cd191a2bd1243b12cfda7f18",
                    "EndpointID": "0720fd9f938e2052e13d4b0fe73663fbae31b784dea4942e53fa5d96e78982f5",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:03",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

</details>


<details>

  <summary>sudo docker inspect mariadb</summary>

```console
[
    {
        "Id": "0eef97284c44a702a82c97921983d398e6268dfbb9db242b622598d2d863ccdf",
        "Created": "2022-08-04T03:22:33.454861483Z",
        "Path": "docker-entrypoint.sh",
        "Args": [
            "mariadbd"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 893,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2024-02-05T11:23:32.550263251Z",
            "FinishedAt": "2022-08-31T15:40:19.473381119Z"
        },
        "Image": "sha256:40b966d7252f541b41677fc35f8660fa90d14df0f33edc8085e6ca2dc0c5b247",
        "ResolvConfPath": "/var/lib/docker/containers/0eef97284c44a702a82c97921983d398e6268dfbb9db242b622598d2d863ccdf/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/0eef97284c44a702a82c97921983d398e6268dfbb9db242b622598d2d863ccdf/hostname",
        "HostsPath": "/var/lib/docker/containers/0eef97284c44a702a82c97921983d398e6268dfbb9db242b622598d2d863ccdf/hosts",
        "LogPath": "/var/lib/docker/containers/0eef97284c44a702a82c97921983d398e6268dfbb9db242b622598d2d863ccdf/0eef97284c44a702a82c97921983d398e6268dfbb9db242b622598d2d863ccdf-json.log",
        "Name": "/mariadb",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "database:/var/lib/mysql"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "3306/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "3306"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "always",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/82783479443dcee9683d0c77cad8d52df60284aef1b1df35e4da31b2530cf4f5-init/diff:/var/lib/docker/overlay2/7740e691233a1c197e662ad77ec81479ed9ee9d74da2a5ce2cfec614ff7b4053/diff:/var/lib/docker/overlay2/7ddaa46c247ab6759a7b0f24731bccbfee84d6fae9e785da4f90be92ce36fa76/diff:/var/lib/docker/overlay2/921909c8634e43b6aca256b4e0f7436a35ec1115c3ea11823d7b4de7b447388d/diff:/var/lib/docker/overlay2/77f6448b303e9263bacfd0d8f7df4bee6d6da5f4a37b830f909836ffde96dbe2/diff:/var/lib/docker/overlay2/921e829602a7cac3a7bc75e7af38769b64cd2062aaeb1ed5f653e5c14210135c/diff:/var/lib/docker/overlay2/98cffa49a98858ce3df1aa89daa3782ed3197d3d2dea86abcd3be8fc33ce38d9/diff:/var/lib/docker/overlay2/855a0892a98c90f48dc35b608cd29d019cb5cb591080b60f60823b482d340695/diff:/var/lib/docker/overlay2/29bad57646c95050481a5f7bda7fb1139921db773d5cfa5fe2b7408c81b9e87b/diff:/var/lib/docker/overlay2/e319e6d9991ec24f650a11b56be67f812eb33540d12b665a85d467a881aeba05/diff:/var/lib/docker/overlay2/c1abd008bb694bedefebce77264f80634e94eba834240e926b0213ce74574fea/diff:/var/lib/docker/overlay2/9ab120a896d5de2091e831d4bb3d8e2db53a05e261f7308d4b1b6f889cc66535/diff",
                "MergedDir": "/var/lib/docker/overlay2/82783479443dcee9683d0c77cad8d52df60284aef1b1df35e4da31b2530cf4f5/merged",
                "UpperDir": "/var/lib/docker/overlay2/82783479443dcee9683d0c77cad8d52df60284aef1b1df35e4da31b2530cf4f5/diff",
                "WorkDir": "/var/lib/docker/overlay2/82783479443dcee9683d0c77cad8d52df60284aef1b1df35e4da31b2530cf4f5/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "database",
                "Source": "/var/lib/docker/volumes/database/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "0eef97284c44",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "3306/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "MYSQL_ROOT_PASSWORD=password",
                "MYSQL_DATABASE=wordpress",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "GOSU_VERSION=1.14",
                "MARIADB_MAJOR=10.8",
                "MARIADB_VERSION=1:10.8.3+maria~jammy"
            ],
            "Cmd": [
                "mariadbd"
            ],
            "Image": "mariadb:latest",
            "Volumes": {
                "/var/lib/mysql": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "076abbc6fa2b2494592f53e4aad7ac41e3f50c24f04fcd09e61e53cff9b8110a",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "3306/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "3306"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/076abbc6fa2b",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "5420d8833d7a64bda9e3dd4ac2df0549f0b8c8cd4e3cdf9600ed4af19c102634",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "41e1fb33abcef2772dadcf2943d42ec330cb6bb2cd191a2bd1243b12cfda7f18",
                    "EndpointID": "5420d8833d7a64bda9e3dd4ac2df0549f0b8c8cd4e3cdf9600ed4af19c102634",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

</details>


`sudo docker exec wordpress env |grep WORDPRESS_DB_`  
```console
WORDPRESS_DB_PASSWORD=password
WORDPRESS_DB_USER=root
```

`grep WORDPRESS_DB_ ./html/wp-config.php`  
```console
define( 'DB_NAME', getenv_docker('WORDPRESS_DB_NAME', 'wordpress') );
define( 'DB_USER', getenv_docker('WORDPRESS_DB_USER', 'example username') );
define( 'DB_PASSWORD', getenv_docker('WORDPRESS_DB_PASSWORD', 'example password') );
define( 'DB_HOST', getenv_docker('WORDPRESS_DB_HOST', 'mysql') );
define( 'DB_CHARSET', getenv_docker('WORDPRESS_DB_CHARSET', 'utf8') );
define( 'DB_COLLATE', getenv_docker('WORDPRESS_DB_COLLATE', '') );
```

`sudo docker exec mariadb mysqladmin -h localhost -u root -ppassword ping`  
```console
mysqld is alive
```

`sudo docker exec mariadb mysql -h 127.0.0.1 -u root -ppassword`  
```console
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.8.3-MariaDB-1:10.8.3+maria~jammy mariadb.org binary distribution
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]>
SELECT user ();
+-----------------+
| user ()         |
+-----------------+
| root@172.17.0.1 |
+-----------------+
1 row in set (0.002 sec)

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
5 rows in set (0.006 sec)

MariaDB [(none)]>use wordpress;
Database changed
MariaDB [wordpress]> show tables;
Empty set (0.000 sec)
MariaDB [wordpress]> SELECT user FROM mysql.user;
+-------------+
| User        |
+-------------+
| root        |
| mariadb.sys |
| root        |
+-------------+
3 rows in set (0.012 sec)
MariaDB [(none)]>
```

##### Everything seems OK, except for the absence of network connectivity between two docker containers. In order to solve this issue, we will use a Docker Compose.


#### 2. Create docker-compose.yaml
`sudo docker-compose -v`  
```console
docker-compose version 1.25.0, build unknown
```

`vi /home/admin/docker-compose.yaml`  
```console
version: "3"

services:
  mysql:
    image: mariadb:latest
    container_name: mysql
    hostname: mysql
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: wordpress
    volumes:
      - /home/admin/database:/var/lib/mysql

  wordpress:
    image: wordpress:sad
    container_name: wordpress
    hostname: wordpress
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_PASSWORD: password
      WORDPRESS_DB_USER: root
    volumes:
      - /home/admin/html:/var/www/html
    depends_on:
      - mysql
```


#### 3. Run it
`sudo docker stop $(docker ps -a -q) && sudo docker rm $(docker ps -a -q)`  
```console
6ffb084b515c
0eef97284c44
6ffb084b515c
0eef97284c44
```

`sudo docker-compose up -d`  
```console
Creating network "admin_default" with the default driver
Creating mysql ... done
Recreating wordpress ... done
```

`sudo docker ps`  
```console
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                    NAMES
0a79c78de0cf   wordpress:sad    "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   0.0.0.0:80->80/tcp       wordpress
64aad9db9a97   mariadb:latest   "docker-entrypoint.s…"   9 seconds ago   Up 8 seconds   0.0.0.0:3306->3306/tcp   mysql
```


#### 4. Check web server
<details>

  <summary>curl -v -L localhost:80</summary>

```html
*   Trying 127.0.0.1:80...
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: localhost
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 302 Found
< Date: Mon, 05 Feb 2024 14:29:36 GMT
< Server: Apache/2.4.54 (Debian)
< X-Powered-By: PHP/7.4.30
< Expires: Wed, 11 Jan 1984 05:00:00 GMT
< Cache-Control: no-cache, must-revalidate, max-age=0
< X-Redirect-By: WordPress
< Location: http://localhost/wp-admin/install.php
< Content-Length: 0
< Content-Type: text/html; charset=UTF-8
< 
* Connection #0 to host localhost left intact
* Issue another request to this URL: 'http://localhost/wp-admin/install.php'
* Found bundle for host localhost: 0x559782ab72f0 [serially]
* Can not multiplex, even if we wanted to!
* Re-using existing connection! (#0) with host localhost
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET /wp-admin/install.php HTTP/1.1
> Host: localhost
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Mon, 05 Feb 2024 14:29:36 GMT
< Server: Apache/2.4.54 (Debian)
< X-Powered-By: PHP/7.4.30
< Expires: Wed, 11 Jan 1984 05:00:00 GMT
< Cache-Control: no-cache, must-revalidate, max-age=0
< Vary: Accept-Encoding
< Content-Length: 6960
< Content-Type: text/html; charset=utf-8
< 
<!DOCTYPE html>
<html lang="en-US" xml:lang="en-US">
<head>
        <meta name="viewport" content="width=device-width" />
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <meta name="robots" content="noindex,nofollow" />
        <title>WordPress &rsaquo; Installation</title>
        <link rel='stylesheet' id='dashicons-css'  href='http://localhost/wp-includes/css/dashicons.min.css?ver=6.0.1' type='text/css' media='all' />
<link rel='stylesheet' id='buttons-css'  href='http://localhost/wp-includes/css/buttons.min.css?ver=6.0.1' type='text/css' media='all' />
<link rel='stylesheet' id='forms-css'  href='http://localhost/wp-admin/css/forms.min.css?ver=6.0.1' type='text/css' media='all' />
<link rel='stylesheet' id='l10n-css'  href='http://localhost/wp-admin/css/l10n.min.css?ver=6.0.1' type='text/css' media='all' />
<link rel='stylesheet' id='install-css'  href='http://localhost/wp-admin/css/install.min.css?ver=6.0.1' type='text/css' media='all' />
</head>
<body class="wp-core-ui">
<p id="logo">WordPress</p>

        <h1>Welcome</h1>
<p>Welcome to the famous five-minute WordPress installation process! Just fill in the information below and you&#8217;ll be on your way to using the most extendable and powerful personal publishing platform in the world.</p>

<h2>Information needed</h2>
<p>Please provide the following information. Do not worry, you can always change these settings later.</p>

                <form id="setup" method="post" action="install.php?step=2" novalidate="novalidate">
        <table class="form-table" role="presentation">
                <tr>
                        <th scope="row"><label for="weblog_title">Site Title</label></th>
                        <td><input name="weblog_title" type="text" id="weblog_title" size="25" value="" /></td>
                </tr>
                <tr>
                        <th scope="row"><label for="user_login">Username</label></th>
                        <td>
                                                        <input name="user_name" type="text" id="user_login" size="25" value="" />
                                <p>Usernames can have only alphanumeric characters, spaces, underscores, hyphens, periods, and the @ symbol.</p>
                                                        </td>
                </tr>
                                <tr class="form-field form-required user-pass1-wrap">
                        <th scope="row">
                                <label for="pass1">
                                        Password                                </label>
                        </th>
                        <td>
                                <div class="wp-pwd">
                                                                                <input type="password" name="admin_password" id="pass1" class="regular-text" autocomplete="new-password" data-reveal="1" data-pw="AYNFoXD#DJrgQ$T8ux" aria-describedby="pass-strength-result" />
                                        <button type="button" class="button wp-hide-pw hide-if-no-js" data-start-masked="0" data-toggle="0" aria-label="Hide password">
                                                <span class="dashicons dashicons-hidden"></span>
                                                <span class="text">Hide</span>
                                        </button>
                                        <div id="pass-strength-result" aria-live="polite"></div>
                                </div>
                                <p><span class="description important hide-if-no-js">
                                <strong>Important:</strong>
                                                                You will need this password to log&nbsp;in. Please store it in a secure location.</span></p>
                        </td>
                </tr>
                <tr class="form-field form-required user-pass2-wrap hide-if-js">
                        <th scope="row">
                                <label for="pass2">Repeat Password                                      <span class="description">(required)</span>
                                </label>
                        </th>
                        <td>
                                <input name="admin_password2" type="password" id="pass2" autocomplete="new-password" />
                        </td>
                </tr>
                <tr class="pw-weak">
                        <th scope="row">Confirm Password</th>
                        <td>
                                <label>
                                        <input type="checkbox" name="pw_weak" class="pw-checkbox" />
                                        Confirm use of weak password                            </label>
                        </td>
                </tr>
                                <tr>
                        <th scope="row"><label for="admin_email">Your Email</label></th>
                        <td><input name="admin_email" type="email" id="admin_email" size="25" value="" />
                        <p>Double-check your email address before continuing.</p></td>
                </tr>
                <tr>
                        <th scope="row">Search engine visibility</th>
                        <td>
                                <fieldset>
                                        <legend class="screen-reader-text"><span>Search engine visibility </span></legend>
                                                                                        <label for="blog_public"><input name="blog_public" type="checkbox" id="blog_public" value="0"  />
                                                Discourage search engines from indexing this site</label>
                                                <p class="description">It is up to search engines to honor this request.</p>
                                                                        </fieldset>
                        </td>
                </tr>
        </table>
        <p class="step"><input type="submit" name="Submit" id="submit" class="button button-large" value="Install WordPress"  /></p>
        <input type="hidden" name="language" value="" />
</form>
        <script type="text/javascript">var t = document.getElementById('weblog_title'); if (t){ t.focus(); }</script>
        <script type='text/javascript' src='http://localhost/wp-includes/js/jquery/jquery.min.js?ver=3.6.0' id='jquery-core-js'></script>
<script type='text/javascript' src='http://localhost/wp-includes/js/jquery/jquery-migrate.min.js?ver=3.3.2' id='jquery-migrate-js'></script>
<script type='text/javascript' id='zxcvbn-async-js-extra'>
/* <![CDATA[ */
var _zxcvbnSettings = {"src":"http:\/\/localhost\/wp-includes\/js\/zxcvbn.min.js"};
/* ]]> */
</script>
<script type='text/javascript' src='http://localhost/wp-includes/js/zxcvbn-async.min.js?ver=1.0' id='zxcvbn-async-js'></script>
<script type='text/javascript' src='http://localhost/wp-includes/js/dist/vendor/regenerator-runtime.min.js?ver=0.13.9' id='regenerator-runtime-js'></script>
<script type='text/javascript' src='http://localhost/wp-includes/js/dist/vendor/wp-polyfill.min.js?ver=3.15.0' id='wp-polyfill-js'></script>
<script type='text/javascript' src='http://localhost/wp-includes/js/dist/hooks.min.js?ver=c6d64f2cb8f5c6bb49caca37f8828ce3' id='wp-hooks-js'></script>
<script type='text/javascript' src='http://localhost/wp-includes/js/dist/i18n.min.js?ver=ebee46757c6a411e38fd079a7ac71d94' id='wp-i18n-js'></script>
<script type='text/javascript' id='wp-i18n-js-after'>
wp.i18n.setLocaleData( { 'text direction\u0004ltr': [ 'ltr' ] } );
</script>
<script type='text/javascript' id='password-strength-meter-js-extra'>
/* <![CDATA[ */
var pwsL10n = {"unknown":"Password strength unknown","short":"Very weak","bad":"Weak","good":"Medium","strong":"Strong","mismatch":"Mismatch"};
/* ]]> */
</script>
<script type='text/javascript' src='http://localhost/wp-admin/js/password-strength-meter.min.js?ver=6.0.1' id='password-strength-meter-js'></script>
<script type='text/javascript' src='http://localhost/wp-includes/js/underscore.min.js?ver=1.13.3' id='underscore-js'></script>
<script type='text/javascript' id='wp-util-js-extra'>
/* <![CDATA[ */
var _wpUtilSettings = {"ajax":{"url":"\/wp-admin\/admin-ajax.php"}};
/* ]]> */
</script>
<script type='text/javascript' src='http://localhost/wp-includes/js/wp-util.min.js?ver=6.0.1' id='wp-util-js'></script>
<script type='text/javascript' id='user-profile-js-extra'>
/* <![CDATA[ */
var userProfileL10n = {"user_id":"0","nonce":""};
/* ]]> */
</script>
<script type='text/javascript' src='http://localhost/wp-admin/js/user-profile.min.js?ver=6.0.1' id='user-profile-js'></script>
<script type="text/javascript">
jQuery( function( $ ) {
        $( '.hide-if-no-js' ).removeClass( 'hide-if-no-js' );
} );
</script>
</body>
</html>
* Connection #0 to host localhost left intact
```

</details>



#### 5. Validate the task
`sudo docker exec wordpress mysqladmin -h mysql -u root -ppassword ping`  
```console
mysqld is alive
```

`./agent/check.sh`  
```console
OK
```
