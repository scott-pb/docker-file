# docker-file
server {
    listen 81;
    index index.php index.html;
    server_name localhost;
    error_log  /var/log/nginx/error.log notice;
    access_log /var/log/nginx/access.log;
    root /home/www-data/pms/public;
    rewrite_log on;
    
    set $request_url $request_uri;

    if ($request_url ~ ^/pages(.*)$ ) {
        set $request_url /pages/login;
    }
    
    location / {
         try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param REQUEST_URI $request_url;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}