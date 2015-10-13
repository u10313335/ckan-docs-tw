CKAN 佈署
========================================

由於 CKAN 使用 pylons 開發，只要使用任何支援 WSGI 標準的網頁伺服器 (及相關套件) 即可佈署 CKAN。 `官方文件 <http://docs.ckan.org/en/latest/maintaining/installing/deployment.html>`_ 提供多種佈署方式，此教學使用 nginx + uwsgi 方式，較官方示範之 Apache + modwsgi + nginx 單純。

.. note::

   本教學部份內容係參考 `How To Set Up uWSGI and Nginx to Serve Python Apps on Ubuntu 14.04 (DigitalOcean) <https://www.digitalocean.com/community/tutorials/how-to-set-up-uwsgi-and-nginx-to-serve-python-apps-on-ubuntu-14-04>`_

1. 新增 production.ini 設定檔
--------------------------------
   .. code-block:: bash

      $ cp /etc/ckan/default/development.ini /etc/ckan/default/production.ini

2. 修改 production.ini
------------------------
   開啟 production.ini，並修改 [app:main] 的相關設定如下：

   .. code-block:: ini

      [app:main]
      ckan.site_url = http://site.domain

   並在檔案的最下方加入：

   .. code-block:: ini

      [uwsgi]
      socket = /tmp/ckan_socket.sock
      master = true
      processes = 1
      chmod-socket = 664
      vacuum = true
      die-on-term = true
      logto = (欲存放程式除錯紀錄檔之目錄)

3. 安裝 uwsgi
----------------
   在虛擬環境下安裝 uwsgi：

   .. code-block:: bash

      $ . /usr/lib/ckan/default/bin/activate
      (pyenv) $ pip install uwsgi

4. 設定開機自動執行
-------------------------
a. 建立 Upstart 檔案：

   .. code-block:: bash

      $ sudo vi /etc/init/ckan.conf

b. 在開啟的 vi 編輯器中，輸入以下內容：

   .. code-block:: bash

      description "uWSGI instance to serve CKAN"

      start on runlevel [2345]
      stop on runlevel [!2345]

      setuid (填入 /usr/lib/ckan/default 目錄的擁有者)
      setgid www-data

      script
          cd /etc/ckan/default
          . /usr/lib/ckan/default/bin/activate
          uwsgi --ini-paste /etc/ckan/default/production.ini
      end script

c. 之後便可使用以下指令啟動網站：

   .. code-block:: bash

      $ sudo start ckan

d. 你可以使用以下指令確認網站是否正常運作：

   .. code-block:: bash

      $ ps aux | grep ckan

   你應該可以看到類似下面的輸出：

   .. code-block:: bash

      demo 12575  0.0  0.5 249060 85144 ?        S    Sep15   0:41 uwsgi --ini-paste /etc/ckan/default/production.ini

e. 你可以使用以下指令停止網站：

   .. code-block:: bash

      $ sudo stop ckan

5. 安裝 nginx 伺服器
----------------------
.. code-block:: bash

   $ sudo apt-get install nginx

6. nginx 伺服器設定
----------------------
a. 新增 /etc/nginx/sites-available/ckan 檔案，並編輯加入以下設定：

   .. code-block:: php

      proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=cache:30m max_size=250m;

      server {
          listen 80;
          server_name server_domain_or_IP;
          client_max_body_size 1000M;
          access_log /var/log/nginx/ckan_access.log;
          error_log /var/log/nginx/ckan_error.log error;

          location / {
              include uwsgi_params;
              uwsgi_pass unix:///tmp/ckan_socket.sock;
              uwsgi_param SCRIPT_NAME '';
          }
      }

b. 建立 alies 至 sites-enabled：

   .. code-block:: bash

      $ sudo ln -s /etc/nginx/sites-available/ckan /etc/nginx/sites-enabled/ckan

c. 重新啟動 nginx：

   .. code-block:: bash

      $ sudo service nginx restart

7. 執行與測試
-------------------------
打開瀏覽器，前往 http://127.0.0.1/ ，若能看到頁面，恭喜您已經完成所有部署設定！
