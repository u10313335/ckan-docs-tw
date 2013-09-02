ckanext-harvest
===============

ckanext-harvest 是一個 ckan 的延伸套件（extension），提供一可自訂之介面（interface），以擷取其他網站（或服務）之 metadata，並匯入為 ckan 資料集。

harvest 的運作大致可分為三步驟（同時也是設計 harvesting interface 的主要結構）:

* gather: 取得 harvest source 的 id, 數量等基本資訊。
* fetch: 取得 source 中每個 object（物件，或稱資料集）之 metadata。
* import: 將上一階段取得的 metadata 轉換並建立為 ckan package（資料集）。

外掛主要功能簡介與使用
----------------------

新增 harvest source
^^^^^^^^^^^^^^^^^^^^

使用瀏覽器開啟 SITE_URL/harvest，選取右上之 "Add Harvest source"，依照畫面輸入 source 網址及選取 source 類別。

執行 harvest 工作（手動）
^^^^^^^^^^^^^^^^^^^^^^^^^

a. 進入 virtualenv，執行 gather 與 fetch handler：

   .. code-block:: bash

      (pyenv) $ paster --plugin=ckanext-harvest harvester gather_consumer -c /etc/ckan/default/production.ini
      (pyenv) $ paster --plugin=ckanext-harvest harvester fetch_consumer -c /etc/ckan/default/production.ini

   .. note::

      請勿關閉這兩個 handler

b. 使用瀏覽器開啟 SITE_URL/harvest，進入剛才建立的 harvest source，選擇右上的「管理者」按鈕，在接下來的頁面選取 ``Reharvest`` ，將此 harvest 工作送入排程。

c. 最後進入 virtualenv，執行 run handler：

   .. code-block:: bash

      (pyenv) $ paster --plugin=ckanext-harvest harvester run -c /etc/ckan/default/production.ini


   即會立即開始執行剛才加入的工作排程。

   .. note::

      手動執行時 harvest 工作並不會自行停止，因為上述 paster harvester run 指令同時也用來確認 harvest 工作是否完成。因此若您確定 harvest 工作已經完成（或已發生錯誤），可以再次執行 run 指令，即可透過下述 d. 的方式檢視此次工作的結果

執行 harvest 工作（自動）
^^^^^^^^^^^^^^^^^^^^^^^^^

在 production 環境時，我們會希望系統可以每隔一段時間自動進行 harvesting，此時可以使用 Supervisor 與 cron 來達到目的：

* `Supervisor <http://supervisord.org/>`_ : 一套任務管理工具，可以在背景執行指定之工作，我們用它來在背景執行 harvest 的 ``gather_consumer`` 與 ``fetch_consumer`` 兩個常駐工作。
* cron: unix/linux 系統工具，可以定時執行之工作，我們用它來定時執行 harvest ``run`` 工作。

a. 首先我們要安裝 Supervisor：

   .. code-block:: bash

      $ sudo apt-get install supervisor

   您可以透過以下指令確定 Supervisor 是否正在執行：

   .. code-block:: bash

      $ ps aux | grep supervisord

   若 Supervisor 正在執行，則會看到類似以下的輸出：

   .. code-block:: bash

      root      9224  0.0  0.3  56420 12204 ?        Ss   15:52   0:00 /usr/bin/python /usr/bin/supervisord

b. Supervisor 的設定檔位於 /etc/supervisor/conf.d 目錄下，我們新增一個新的設定檔，命名為 ckan_harvesting.conf，內容如下：

   .. code-block:: python

      ; ===============================
      ; ckan harvester
      ; ===============================

      [program:ckan_gather_consumer]

      command=/usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester gather_consumer -c /etc/ckan/default/production.ini

      ; user that owns virtual environment.
      user=okfn

      numprocs=1
      stdout_logfile=/var/log/ckan/default/gather_consumer.log
      stderr_logfile=/var/log/ckan/default/gather_consumer.log
      autostart=true
      autorestart=true
      startsecs=10

      [program:ckan_fetch_consumer]

      command=/usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester fetch_consumer -c /etc/ckan/default/production.ini

      ; user that owns virtual environment.
      user=okfn

      numprocs=1
      stdout_logfile=/var/log/ckan/default/fetch_consumer.log
      stderr_logfile=/var/log/ckan/default/fetch_consumer.log
      autostart=true
      autorestart=true
      startsecs=10

   其中 ``user=okfn`` 請代換成 python virtual environment 的擁有者， ``/var/log/ckan/default`` 目錄請自行新增，擁有者同樣為 virtualenv 擁有者

c. 接著啟動 Supervisor，請依序輸入以下指令：

   .. code-block:: bash

      $ sudo supervisorctl reread
      $ sudo supervisorctl add ckan_gather_consumer
      $ sudo supervisorctl add ckan_fetch_consumer
      $ sudo supervisorctl start ckan_gather_consumer
      $ sudo supervisorctl start ckan_fetch_consumer

   您可以透過以下指令確定工作是否正在執行：

   .. code-block:: bash

      $ sudo supervisorctl status

   若 Supervisor 正在執行，則會看到類似以下的輸出：

   .. code-block:: bash

      ckan_fetch_consumer              RUNNING    pid 6983, uptime 0:22:06
      ckan_gather_consumer             RUNNING    pid 6968, uptime 0:22:45

d. 最後我們要建立定時執行 ``run`` 排程，執行下列指令打開排程設定檔：

   .. code-block:: bash

      $ sudo crontab -e -u okfn

   ``okfn`` 請代換為 virtualenv 擁有者

e. 進行排程設定，請加入以下文字於 crontab 設定中：

   # m  h  dom mon dow   command

   \*/15 *  *   *   *     /usr/lib/ckan/default/bin/paster --plugin=ckanext-harvest harvester run -c /etc/ckan/default/production.ini

確認 harvest 工作的執行狀況
^^^^^^^^^^^^^^^^^^^^^^^^^^^

我們可以在網頁介面，harvest source 的「管理者」頁面確認 harvest 工作的執行狀況，包括錯誤、新增、更新、完成的資料集數目，如下圖所示：

   .. image:: harvest-job-status.png

撰寫自定義 harvesting interface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如先前所述，ckanext-harvest 提供可以自行定義的 interface，因此您可以為某個網站，或某種資料來源，特別製作 harvester。

* 本人撰寫之 `中研院調查研究中心（SRDA）資料庫 <https://srda.sinica.edu.tw/>`_ harvester： `SRDAHarvester <https://github.com/u10313335/ckanext-harvest/blob/master/ckanext/harvest/harvesters/srdaharvester.py>`_
* ckan 官方提供之 csv harvester 範例： `DataLondonGovUkHarvester <https://github.com/okfn/ckanext-pdeu/blob/master/ckanext/pdeu/harvesters/london.py>`_

系統需求
--------
* Python (2 or 3) 安裝於 virtualenv
* ckan
* RabbitMQ 或 Redis

安裝
----
a. 安裝 RabbitMQ 或 Redis（兩者則一安裝即可，以下用 Redis 作示範）：

   .. code-block:: bash

      $ sudo apt-get install rabbitmq-server
      $ sudo apt-get install redis-server

b. 安裝 ckanext-harvest 套件：

   .. code-block:: bash

      (pyenv) $ pip install -e git+https://github.com/okfn/ckanext-harvest.git@release-v2.0#egg=ckanext-harvest

   .. note::

      release-v2.0 請自行依 ckan 版本替換之

c. 安裝其他需要的 Python 套件：

   .. code-block:: bash

      (pyenv) $ pip install -r pip-requirements.txt

d. 修改 ckan 設定檔，修改 ckan.plugins，加入：

   .. code-block:: python

      ckan.plugins = harvest ckan_harvester
      ...
      ckan.harvest.mq.type = redis

e. 初始化 harvest 資料庫：

   .. code-block:: bash

      (pyenv) $ paster --plugin=ckanext-harvest harvester initdb -c /etc/ckan/default/production.ini
