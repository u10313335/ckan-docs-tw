DataStore 與 DataPusher
=======================

`DataStore <http://docs.ckan.org/en/latest/maintaining/datastore.html>`_ 是一個內建於 CKAN 的擴充套件 (extension)，透過一獨立資料庫儲存上傳至 CKAN 之結構資料內容（CSV 或 XLS 檔案，無論為上傳至本機的檔案或僅有連結）。

`DataPusher <http://docs.ckan.org/projects/datapusher/en/latest/>`_ 是一個 CKAN 的擴充套件 (extnsion)，用以於新增前述的結構資料至 CKAN 時，自動上傳資料內容至 DataStore 資料庫。


功能簡介
--------
* DataStore
  
  * 上傳至資料庫的資料內容，可提供 *資料預覽外掛* 使用。
  * 提供 `DataStore API <http://docs.ckan.org/en/latest/maintaining/datastore.html#the-datastore-api>`_ 可供開發者以 RESTful API 取得 JSON 格式資料。

* DataPusher

  * 自動上傳資料內容至 DataStore 資料庫。
  * 可自資料編輯頁面之「DataStore」頁籤確認上傳狀態或手動上傳資料至 DataStore 資料庫。

系統需求
---------
* CKAN (>=2.1)
* PostgreSQL (>=9.0)

.. note::

   * 若依照本文件的教學安裝 CKAN，你應該已經滿足所有套件需求

DataStore 設定
--------------
a. 啟用 DataStore：

   修改 CKAN 設定檔（一般位於 /etc/ckan/default/），在 ckan.plugins 最後加上：

   .. code-block:: ini

      ckan.plugins = datastore

b. 新增 DataStore 使用之 PostgreSQL 使用者：

   .. code-block:: bash

      $ sudo -u postgres createuser -S -D -R -P -l datastore_default

c. 新增 DataStore 使用之資料庫：

   .. code-block:: bash

      $ sudo -u postgres createdb -O ckan_default datastore_default -E utf-8

d. DataStore 資料庫連線設定：

   修改 CKAN 設定檔，搜尋下面字串，並將帳號密碼與 db 名稱依照上一步所新增的 db 設定：

   .. code-block:: ini

      ckan.datastore.write_url = postgresql://ckan_default:pass@localhost/datastore_default
      ckan.datastore.read_url = postgresql://datastore_default:pass@localhost/datastore_default

   .. note::

      write_url 的第一個 ckan_default 是 CKAN 資料庫使用者名稱，pass 請填寫 db 密碼，最後的 datastore_default 填入 db 名稱，read_url 同理。

e. DataStore 資料庫權限設定：

   .. code-block:: bash

      (pyenv) $ paster --plugin=ckan datastore set-permissions -c /etc/ckan/default/development.ini

f. 重新啟動 CKAN

g. 測試 DataStore，可輸入以下指令：

   .. code-block:: bash

      $ curl -X GET "http://127.0.0.1/api/3/action/datastore_search?resource_id=_table_metadata"

DataPusher 安裝
---------------
a. 安裝必須套件：

   .. code-block:: bash

      $ sudo apt-get install python-dev python-virtualenv build-essential libxslt1-dev libxml2-dev git

b. 新增一個虛擬環境供 DataPusher 使用：

   .. code-block:: bash

      $ sudo mkdir -p /usr/lib/ckan/datapusher
      $ sudo chown `whoami` /usr/lib/ckan/datapusher
      $ virtualenv --no-site-packages /usr/lib/ckan/datapusher

c. 進入剛才新增的虛擬環境：

   .. code-block:: bash

      $ . /usr/lib/ckan/datapusher/bin/activate

d. 自 github ckeckout source 並安裝：

   .. code-block:: bash

      $ cd /usr/lib/ckan/datapusher/src
      $ git clone https://github.com/ckan/datapusher.git
      (pyenv) $ pip install -e .

e. 安裝所需 Python 套件：

   .. code-block:: bash

      (pyenv) $ pip install -r requirements.txt

f. 執行 DataPusher：

   .. code-block:: bash

      (pyenv) $ JOB_CONFIG='/usr/lib/ckan/datapusher/src/datapusher/deployment/datapusher_settings.py' python wsgi.py

g. 測試 DataPusher，可在瀏覽器輸入 http://127.0.0.1:8800

h. 啟用 DataPusher CKAN 外掛：

   修改 CKAN 設定檔（一般位於 /etc/ckan/default/），在 ckan.plugins 最後加上：

   .. code-block:: ini

      ckan.plugins = datapusher

i. 重新啟動 CKAN

DataPusher 佈署
---------------
DataPusher 的 Production 安裝與 CKAN 類似，使用 nginx + uwsgi 的方式。

.. note::

   本教學部份內容係參考 `How To Set Up uWSGI and Nginx to Serve Python Apps on Ubuntu 14.04 (DigitalOcean) <https://www.digitalocean.com/community/tutorials/how-to-set-up-uwsgi-and-nginx-to-serve-python-apps-on-ubuntu-14-04>`_ 與 `Serving Flask With Nginx (Vladik Khononov) <http://vladikk.com/2013/09/12/serving-flask-with-nginx-on-ubuntu/>`_

a. 安裝 uwsgi：

   .. code-block:: bash

      (pyenv) $ pip install uwsgi

b. 修改 wsgi.py：

   為配合 uwsgi，我們需要將 wsgi.py 做小修改。

   開啟 /usr/lib/ckan/datapusher/src/datapusher/wsgi.py，修改如下：

   .. code-block:: python

      import ckanserviceprovider.web as web
      import datapusher.jobs as jobs
      import os

      # check whether jobs have been imported properly
      assert(jobs.push_to_datastore)

      os.environ['JOB_CONFIG'] = '/usr/lib/ckan/datapusher/src/datapusher/deployment/datapusher_settings.py'

      web.init()
      web.app.run(web.app.config.get('HOST'), web.app.config.get('PORT'))

c. 建立 uwsgi 設定檔：

   新增 /etc/ckan/default/datapusher.ini，內容如下：

   .. code-block:: ini

      [uwsgi]
      wsgi-file = /usr/lib/ckan/datapusher/src/datapusher/wsgi.py
      socket = /tmp/datapusher.sock
      master = true
      processes = 1
      chmod-socket = 664
      vacuum = true
      die-on-term = true
      logto = /etc/ckan/default/log/datapusher.log

d. 建立 Upstart 檔案：

   .. code-block:: bash

      $ sudo vi /etc/init/datapusher.conf

e. 在開啟的 vi 編輯器中，輸入以下內容：

   .. code-block:: bash

      description "uWSGI instance to serve DataPusher"

      start on runlevel [2345]
      stop on runlevel [!2345]

      setuid (填入 /usr/lib/ckan/datapusher 目錄的擁有者)
      setgid www-data

      script
          cd /etc/ckan/default
          . /usr/lib/ckan/datapusher/bin/activate
          uwsgi --ini /etc/ckan/default/datapusher.ini
      end script

f. 之後便可使用以下指令啟動 DataPusher：

   .. code-block:: bash

      $ sudo start datapusher

g. 你可以使用以下指令確認 DataPusher 是否正常運作：

   .. code-block:: bash

      $ ps aux | grep datapusher

   你應該可以看到類似下面的輸出：

   .. code-block:: bash

      demo 1009  0.0  0.2 266332 37512 ?        Sl   Sep14   2:49 uwsgi --ini /etc/ckan/default/datapusher.ini

.. note::

   目前此佈署方法無法使用 sudo stop datapusher 的方式停止 DataPusher，請直接使用 kill 指令。

h. 修改 CKAN 設定檔（一般位於 /etc/ckan/default/），修改 ckan.datapusher.url 為：

   .. code-block:: ini

      ckan.datapusher.url = http://0.0.0.0:8800/

i. 重新啟動 CKAN
