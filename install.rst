ckan 安裝
========================================

1. 安裝必須套件
------------------------
   .. code-block:: bash

      sudo apt-get install python-dev postgresql libpq-dev python-pip python-virtualenv git-core jetty8 openjdk-7-jdk

2. Virtual environment 設定
----------------------------
a. 新增一個 virtual environment (virtualenv) 供 ckan 使用：

   .. code-block:: bash

      sudo mkdir -p /usr/lib/ckan/default
      sudo chown `whoami` /usr/lib/ckan/default
      virtualenv --no-site-packages /usr/lib/ckan/default

b. 進入剛才新增的 virtualenv：

   .. code-block:: bash

      . /usr/lib/ckan/default/bin/activate

   .. note::

      要離開 virtualenv，可使用 deactivate 指令。若需要返回 virtualenv，可以再執行一次 . /usr/lib/ckan/default/bin/activate 即可。

3. 安裝 ckan 2.0
-----------------
   自 github ckeckout source (這邊以 release-2.0 為例）並安裝：

   .. code-block:: bash

      pip install -e 'git+https://github.com/okfn/ckan.git@ckan-2.0#egg=ckan'

   安裝所需 Python 套件：

   .. code-block:: bash

      pip install -r /usr/lib/ckan/default/src/ckan/pip-requirements.txt

4. 設定資料庫
--------------
a. 新增 ckan 使用之 postgreSQL 使用者：

   .. code-block:: bash

      sudo -u postgres createuser -S -D -R -P ckan_default

b. 新增 ckan 使用之資料庫：

   .. code-block:: bash

      sudo -u postgres createdb -O ckan_default ckan_default -E utf-8

5. 建立 ckan 設定檔
--------------------
a. 新增放置 ckan 設定檔之目錄：

   .. code-block:: bash

      sudo mkdir -p /etc/ckan/default
      sudo chown -R `whoami` /etc/ckan/

b. 回到 ckan source 目錄，透過 paster 新增範例設定檔：

   .. important::

      執行任何 paster 指令時，請確認是在 virtualenv 下，且在 /usr/lib/ckan/default/src/ckan 目錄下

   .. code-block:: bash

      cd /usr/lib/ckan/default/src/ckan
      paster make-config ckan /etc/ckan/default/development.ini

c. 修改前面新增的 development.ini，搜尋下面字串，並將帳號密碼與 db 名稱依照 4. 所新增的 db 設定：

   .. code-block:: ini

      sqlalchemy.url = postgresql://ckan_default:pass@localhost/ckan_default
   .. note::

      第一個 ckan_default 是使用者名稱，pass 請填寫 db 密碼，最後的 ckan_default 填入 db 名稱）

6. 設定 jetty8 與 solr4（w/搜尋中文支援）
-----------------------------------------
a. 修改 jetty 設定（位於 /etc/default/jetty8）：

   .. code-block:: ini

      NO_START=0
      JETTY_HOST=127.0.0.1
      JETTY_PORT=8983
      JAVA_OPTIONS="-Dsolr.solr.home=/usr/share/solr $JAVA_OPTIONS" 

b. 安裝 solr4：

   至官網 http://lucene.apache.org/solr/ 下載 solr-4.3.1
   
   解壓縮下載回來的壓縮檔
   
   並複製 ./dist 下的 solr-4.3.1.war 至 jetty webapps 目錄（solr 目錄請自行建立）：

   .. code-block:: bash

      sudo cp solr-4.3.1.war /usr/share/jetty8/webapps/solr/solr.war

   複製以下目錄至指定位置：

   複製 ./example/solr 至 /usr/share（此即為 solr_home）

   複製 ./contrib 至 /usr/share/solr/bin

   複製 ./dist 至 /usr/share/solr

   修改 solr 目錄權限，使 jetty 可以存取：
   
   .. code-block:: bash
   
      sudo chown -R jetty:adm /usr/share/solr

   新增 schema symlink：

   .. code-block:: bash

      sudo mv /usr/share/solr/collection1/conf/schema.xml /usr/share/solr/collection1/conf/schema.xml.bak
      sudo ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema-2.0.xml /usr/share/solr/collection1/conf/schema.xml

   為放置 IKA，需解開 solr-4.3.1.war：
   
   .. code-block:: bash
      
      jar -xvf solr.war

c. 設定 solr：

   打開 /usr/share/solr/collection1/conf/solrconfig.xml，尋找 <lib dir> 區段並修改為：

   .. code-block:: xml

      <lib dir="/usr/share/solr/bin/contrib/extraction/lib" regex=".*\.jar" />
      <lib dir="/usr/share/solr/dist/" regex="solr-cell-\d.*\.jar" />

      <lib dir="/usr/share/solr/bin/contrib/clustering/lib/" regex=".*\.jar" />
      <lib dir="/usr/share/solr/dist/" regex="solr-clustering-\d.*\.jar" />

      <lib dir="/usr/share/solr/bin/contrib/langid/lib/" regex=".*\.jar" />
      <lib dir="/usr/share/solr/dist/" regex="solr-langid-\d.*\.jar" />

      <lib dir="/usr/share/solr/bin/contrib/velocity/lib" regex=".*\.jar" />
      <lib dir="/usr/share/solr/dist/" regex="solr-velocity-\d.*\.jar" />

   並刪除或註解掉此行:
   
   .. code-block:: xml

      <lib dir="/non/existent/dir/yields/warning" />
  
   .. note::

      去除此行，是一個已知問題的暫時解法: https://issues.apache.org/jira/browse/SOLR-4890

d. 安裝 IKAnalyzer：

   下載 IKAnalyzer https://ik-analyzer.googlecode.com/files/IK%20Analyzer%202012FF_hf1.zip 並解壓縮

   複製 IKAnalyzer2012FF_fh1.jar 至 /var/lib/jetty8/webapps/solr/WEB-INF/lib
  
   複製 IKAnalyzer.cfg.xml 和 stopword.dic 至 /var/lib/jetty8/webapps/solr/WEB-INF/class

e. 設定 IKAnalyzer：

   修改 schema.xml，fieldType name="text" 區段修改為：

   .. code-block:: xml

      <fieldType name="text" class="solr.TextField">
         <analyzer type="index" class="org.wltea.analyzer.lucene.IKAnalyzer" isMaxWordLength="false"/>
         <analyzer type="query" class="org.wltea.analyzer.lucene.IKAnalyzer" isMaxWordLength="false"/>
         <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
         <filter class="solr.WordDelimiterFilterFactory" generateWordParts="1" generateNumberParts="1" catenateWords="0" catenateNumbers="0" catenateAll="0" splitOnCaseChange="1"/>
         <filter class="solr.SnowballPorterFilterFactory" language="English" protected="protwords.txt"/>
         <filter class="solr.LowerCaseFilterFactory"/>
         <filter class="solr.ASCIIFoldingFilterFactory"/>
      </fieldType>

   .. note::

       schema.xml 位於 /usr/share/solr/collection1/conf/schema.xml

f. 啟動 jetty：

   .. code-block:: bash

      sudo service jetty8 start

g. 打開瀏覽器，前往 http://127.0.0.1:8983/solr ，若能看到畫面則代表安裝完成


7. 初始化資料庫
------------------------
a. 回到 ckan source 目錄，透過 paster 初始化 ckan db：

   .. code-block:: bash

      cd /usr/lib/ckan/default/src/ckan
      paster db init -c /etc/ckan/default/development.ini

b. 如果一切正常，則會看到此訊息：Initialising DB: SUCCESS

8. 建立 who.ini link
------------------------
   .. code-block:: bash

      ln -s /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/who.ini

9. 新增 ckan 系統管理者
------------------------
   回到 ckan source 目錄，透過 paster 新增 ckan 系統管理者：

   .. code-block:: bash

      cd /usr/lib/ckan/default/src/ckan
      paster sysadmin add admin -c /etc/ckan/default/development.ini

   .. note::

      admin 請代換為您需要的使用者名稱，並依照程式提示設定密碼

10. 在 development 環境下執行
------------------------------
a. 回到 ckan source 目錄，透過 paster serve 新安裝的 ckan instance：

   .. code-block:: bash

      cd /usr/lib/ckan/default/src/ckan
      paster serve /etc/ckan/default/development.ini

b. 打開瀏覽器，前往 http://127.0.0.1:5000/ ，至此 ckan 安裝完成
