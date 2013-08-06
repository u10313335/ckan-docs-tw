ckan 佈署
========================================

這邊使用 nginx+uwsgi 示範。

1. 新增 production.ini 設定檔
--------------------------------
   .. code-block:: bash

      cp /etc/ckan/default/development.ini /etc/ckan/default/production.ini

2. 修改 production.ini
------------------------
   搜尋並修改/新增下列字串：

   .. code-block:: python

      [server:main]
      use = egg:Paste#http
      host = site.domain
      port = 80
      #...
      ckan.site_url = http://site.domain
      #...
      #add the following lines at the bottom
      [uwsgi]
      socket = /tmp/uwsgi.sock
      master = true
      chmod-socket = 666

3. 安裝 nginx
----------------
   .. code-block:: bash

      sudo apt-get install nginx

4. nginx 伺服器設定
----------------------
a. 新增 /etc/nginx/sites-available/ckan 檔案，並編輯加入以下設定：

   .. code-block:: php

      proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=cache:30m max_size=250m;
      proxy_temp_path /tmp/nginx_proxy 1 2;

      server {
          client_max_body_size 100M;
          location / {
             include uwsgi_params;
             uwsgi_pass unix:///tmp/uwsgi.sock;
             uwsgi_param SCRIPT_NAME '';
          }
      }

b. 建立 alies 至 sites-enabled：

   .. code-block:: bash

      sudo ln -s /etc/nginx/sites-available/ckan /etc/nginx/sites-enabled/ckan

5. uwsgi 設定
----------------
   在 virtual env 下安裝 uwsgi：

   .. code-block:: bash

      . /usr/lib/ckan/default/bin/activate
      pip install uwsgi

6. 執行與測試
-------------------------
a. 不要離開 virtual env，執行 nginx 與 uwsgi：

   .. code-block:: bash

      sudo service nginx start
      uwsgi --ini-paste /etc/ckan/default/production.ini

b. 打開瀏覽器，前往 http://127.0.0.1/ ，若能看到頁面，恭喜您已經完成所有設定！
