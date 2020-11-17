<template>
<pre>
server {
  listen [::]:80;
  listen 80;
  server_name {{ site.domain }} www.{{ site.domain }};

  access_log /home/{{ system.username }}/logs/nginx_access.log;
  error_log /home/{{ system.username }}/logs/nginx_error.log;

  client_max_body_size 100M;

  gzip on;
  gzip_min_length 1000;
  gzip_proxied any;
  gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";
  gzip_comp_level 6;


  root /home/{{ system.username }}/{{ site.domain }}/www;
  index index.php;

  location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
    access_log off;
    log_not_found off;
    expires max;
  }

  location / {
    rewrite ^/.*$ /index.php last;
  }

  location ~ ^/index.php {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php-{{ site.name }}.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
  }

  # Whitelist
  ## Let October handle if static file not exists
  location ~ ^/favicon\.ico { try_files $uri /index.php; }
  location ~ ^/sitemap\.xml { try_files $uri /index.php; }
  location ~ ^/robots\.txt { try_files $uri /index.php; }
  location ~ ^/humans\.txt { try_files $uri /index.php; }
  ## Let nginx return 404 if static file not exists
  location ~ ^/storage/app/uploads/public { try_files $uri 404; }
  location ~ ^/storage/app/media { try_files $uri 404; }
  location ~ ^/storage/temp/public { try_files $uri 404; }
  location ~ ^/modules/.*/assets { try_files $uri 404; }
  location ~ ^/modules/.*/resources { try_files $uri 404; }
  location ~ ^/modules/.*/behaviors/.*/assets { try_files $uri 404; }
  location ~ ^/modules/.*/behaviors/.*/resources { try_files $uri 404; }
  location ~ ^/modules/.*/widgets/.*/assets { try_files $uri 404; }
  location ~ ^/modules/.*/widgets/.*/resources { try_files $uri 404; }
  location ~ ^/modules/.*/formwidgets/.*/assets { try_files $uri 404; }
  location ~ ^/modules/.*/formwidgets/.*/resources { try_files $uri 404; }
  location ~ ^/modules/.*/reportwidgets/.*/assets { try_files $uri 404; }
  location ~ ^/modules/.*/reportwidgets/.*/resources { try_files $uri 404; }
  location ~ ^/plugins/.*/.*/assets { try_files $uri 404; }
  location ~ ^/plugins/.*/.*/resources { try_files $uri 404; }
  location ~ ^/plugins/.*/.*/behaviors/.*/assets { try_files $uri 404; }
  location ~ ^/plugins/.*/.*/behaviors/.*/resources { try_files $uri 404; }
  location ~ ^/plugins/.*/.*/reportwidgets/.*/assets { try_files $uri 404; }
  location ~ ^/plugins/.*/.*/reportwidgets/.*/resources { try_files $uri 404; }
  location ~ ^/plugins/.*/.*/formwidgets/.*/assets { try_files $uri 404; }
  location ~ ^/plugins/.*/.*/formwidgets/.*/resources { try_files $uri 404; }
  location ~ ^/plugins/.*/.*/widgets/.*/assets { try_files $uri 404; }
  location ~ ^/plugins/.*/.*/widgets/.*/resources { try_files $uri 404; }
  location ~ ^/themes/.*/assets { try_files $uri 404; }
  location ~ ^/themes/.*/resources { try_files $uri 404; }
}
</pre>
</template>
<script>
export default {
  props: ['system', 'site', 'db'],
}
</script>
