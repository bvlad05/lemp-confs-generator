<template>
  <div id="app">
    <div class="container py-3">
      <div class="row justify-content-center">
        <div class="col-md-8 col-lg-6">
          <form class="card">
            <div class="card-body">
              <h5 class="card-title">System user details</h5>
              <div class="mt-3">
                <input class="form-control" placeholder="Username" aria-label="Username" v-model="system.username">
              </div>
              <h5 class="card-title mt-5">Website details</h5>
              <div class="mt-3">
                <select class="form-select" aria-label="CMS" v-model="site.cms">
                  <option value="october-cms">OctoberCMS</option>
                  <option value="wordpress" disabled>WordPress</option>
                  <option value="laravel" disabled>Laravel</option>
                </select>
              </div>
              <div class="mt-3">
                <input class="form-control" placeholder="Site name" aria-label="Site name" v-model="site.name">
              </div>
              <div class="mt-3">
                <input class="form-control" placeholder="Domain" aria-label="Domain" v-model="site.domain">
              </div>
              <div class="mt-3">
                <input class="form-control" placeholder="Site name" aria-label="Site name" v-model="site.php_v">
              </div>
              <div class="mt-3">
                <input class="form-control" placeholder="Site name" aria-label="Site name" v-model="site.php_timezone">
              </div>
              <h5 class="card-title mt-5">DB details</h5>
              <div class="mt-3">
                <input class="form-control" placeholder="Username" aria-label="Username" v-model="db.username">
              </div>
              <div class="mt-3">
                <input class="form-control" placeholder="DB name" aria-label="DB name" v-model="db.name">
              </div>
              <div class="mt-3">
                <input class="form-control" placeholder="Password" aria-label="Password" v-model="db.password">
              </div>
            </div>
          </form>
          <div class="mt-5">
            <h4>Add user</h4>
            <pre class="code"><code>useradd -d /home/{{ system.username}} -g www-data -m -s /bin/bash {{ system.username}}
usermod -a -G www-data {{ system.username}}
usermod -a -G sudo {{ system.username}}
passwd {{ system.username}}
</code></pre>
            <h4>PHP-Fpm configuration</h4>
            <pre class="code"><code>cd /etc/php/{{ site.php_v }}/fpm/pool.d/
sudo rm www.conf
sudo nano {{ site.name }}.conf

<span class="comment"># And add next lines of code</span>
[{{ site.domain }}]
listen = /run/php/php-{{ site.name }}.sock
listen.mode = 0666
user = {{ system.username }}
group = www-data

php_admin_value[upload_tmp_dir] = /home/{{ system.username }}/tmp
php_admin_value[date.timezone] = {{ site.php_timezone }}
php_admin_value[open_basedir] = /home/{{ system.username }}
php_admin_value[post_max_size] = 512M
php_admin_value[upload_max_filesize] = 512M
php_admin_value[cgi.fix_pathinfo] = 0
php_admin_value[short_open_tag] = On
php_admin_value[memory_limit] = 512M
php_admin_value[session.gc_probability] = 1
php_admin_value[session.gc_divisor] = 100
php_admin_value[session.gc_maxlifetime] = 28800;
php_admin_value[error_log] = /home/{{ system.username }}/logs/php_errors.log;

pm = dynamic
pm.max_children = 10
pm.start_servers = 2
pm.min_spare_servers = 2
pm.max_spare_servers = 4
</code></pre>
            <h4>Nginx configuration</h4>
<pre class="code"><code>cd /etc/nginx/sites-available
sudo nano 00-default.conf
<span class="comment"># And add next lines of code</span>
server {
  listen [::]:80;
  listen      80;
  server_name "";
  return      444;
}

<span class="comment"># Next</span>
cd /etc/nginx/sites-available
sudo nano 01-{{ site.domain }}.conf
<span class="comment"># And add next lines of code</span>
<component :is="site.cms" :system="system" :site="site" :db="db"></component>

<span class="comment"># Create symbolic link</span>
cd ../sites-enabled
sudo ln -s 00-default.conf
sudo ln -s 01-{{ site.domain }}.conf
</code></pre>
            <h4>Nginx MySQL</h4>
<pre class="code"><code>sudo -s
mysql -u root
> USE mysql;
> UPDATE user SET plugin = 'mysql_native_password' WHERE User = 'root';
> ALTER USER 'root'@'localhost' IDENTIFIED BY '1234567890';
> FLUSH PRIVILEGES;
> EXIT;
# Create user and table for website
CREATE DATABASE IF NOT EXISTS {{ db.name }} CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER '{{ db.username }}'@'localhost' IDENTIFIED BY '{{ db.password }}';
GRANT USAGE ON *.* TO '{{ db.username }}'@'localhost' IDENTIFIED BY '{{ db.password }}' REQUIRE NONE WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0;
GRANT ALL PRIVILEGES ON `{{ db.name }}`.* TO '{{ db.username }}'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
su {{ system.username }}
</code></pre>
            <h4>Finish</h4>
<pre class="code"><code>sudo -u {{ system.username }} mkdir /home/{{ system.username }}/tmp
sudo -u {{ system.username }} mkdir /home/{{ system.username }}/logs
sudo -u {{ system.username }} mkdir /home/{{ system.username }}/{{ site.domain }}
sudo -u {{ system.username }} mkdir /home/{{ system.username }}/{{ site.domain }}/www
sudo -u {{ system.username }} mkdir /home/{{ system.username }}/.ssh
sudo -u {{ system.username }} chmod 755 /home/{{ system.username }}/.ssh
sudo -u {{ system.username }} ssh-keygen
sudo service php{{ site.php_v }}-fpm restart
sudo service nginx restart
sudo service mysql restart
</code></pre>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import OctoberCms from './components/Confs/OctoberCms';
export default {
  name: 'App',
  components: {
    OctoberCms,
  },
  data: () => ({
    system: {
      username: '',
    },
    site: {
      name: '',
      domain: '',
      php_v: '7.4',
      php_timezone: 'Europe/Kiev',
      cms: 'october-cms',
    },
    db: {
      username: '',
      name: '',
      password: '',
    },
  }),
}
</script>

<style lang="scss">
@import '~bootstrap/scss/bootstrap';
@import '~highlight.js/scss/atom-one-dark';
.code {
  background-color: #3f3f3f;
  border-radius: 5px;
  padding: 10px 15px;
  color: rgba(#fff, .75);
  margin-bottom: 30px;
  .comment {
    color: rgba(#fff, .25);
  }
}
</style>
