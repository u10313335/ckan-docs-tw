ckanext-spatial
================

ckanext-spatial 是一個 ckan 的延伸套件 (extension)，提供地理資訊相關功能。

詳細介紹可以參考 `ckanext-spatial 的官方 github <https://github.com/okfn/ckanext-spatial>`_ 。

外掛主要功能簡介                                                                                                               
-----------------                                                                                                              
                                                                                                                               
spatial_metadata                                                                                                               
^^^^^^^^^^^^^^^^^^                                                                                                             
建立地理空間資訊之索引。                                                                                                       
                                                                                                                               
Spatial Search Widget                                                                                                          
^^^^^^^^^^^^^^^^^^^^^^                                                                                                         
按地圖搜尋資料集 "spatial" 欄位的地理空間資訊，僅支援 solr 3.1+。安裝完成後，即可在資料集清單顯示頁面的左下角看到 "Filter by location" 的區塊，此區塊並可放大後，依照使用者選取的地理區域篩選出符合的資料集。
                                                                                                                               
欲使用此功能，請在 ckan.plugins 加入 spatial_metadata 與 spatial_query。                                                       
                                                                                                                  
Dataset Extent Map                                                                                                             
^^^^^^^^^^^^^^^^^^^                                                                                                            
以地圖顯示資料集 "spatial" 欄位所述之地理空間資訊 (僅支援 geojson 格式)。如下圖所示，在「額外的資訊」中填寫的 spatial geojson 資訊，將顯示在左下角的 Dataset extent 中。

欲使用此功能，請在 ckan.plugins 加入 spatial_metadata。                                                                        
                                                                                                                               
.. image:: extent-map.png                                                                                                      

WMS Preview                                                                                                                    
^^^^^^^^^^^^                                                                                                                   
此功能可以地圖方式呈現 wms 服務所涵括的地理範圍 (GetCapabilities)，並可切換圖層。

欲使用此功能，請在 ckan.plugins 加入 wms_preview。                                                                             
                                                                                                                               
以 `NASA Earth Observations <http://neowms.sci.gsfc.nasa.gov/wms/wms?version=1.1.1&service=WMS&request=GetCapabilities>`_ 為例>：                                                                                                                             
                                                                                                                               
.. image:: wms-preview.png                                                                                                     
   :scale: 70 %                                                                                                                
                                                                                                                               
GeoJSON Preview                                                                                                                
^^^^^^^^^^^^^^^^^                                                                                                              
以地圖檢視 GeoJSON 檔案（ckan 內建之 preview 僅支援以樹狀結構顯示 json 格式文件）。支援 geojson 與 gjosn 兩種檔案格式名稱定義。                                                                                                                               
欲使用此功能，請在 ckan.plugins 加入 geojson_preview 與 resource_proxy。                                                       
                                                                                                                               
CSW Server                                                                                                                     
^^^^^^^^^^^                                                                                                                    
提供 CSW 服務介面（研究中）。                                                                                                  
                                                                                                                               
.. _spatial-harvesters:                                                                                                        

Spatial Harvesters                                                                                                             
^^^^^^^^^^^^^^^^^^^                                                                                                            
提供地理空間相關的 harvesters，可以將 CSW, WAF, spatial metadata document 等資料目錄來源的後設資料擷取下來並匯入 ckan 之中。須注意的是，資料本身仍然位於原資料目錄之網站。此 harvester 係實作 ckanext-harvest 套件之 harvester interface。

欲使用此功能，請安裝 `ckanext-harvest 外掛 <https://github.com/okfn/ckanext-harvest>`_ 並在 ckan.plugins 加入 csw_harvester, doc_harvester 與 waf_harvester。

.. note::
   
   ckanext-spatial 提供的 havester 現階段 (0.2) 並不穩定，匯入大量資料很緩慢（實測 11,400 筆左右需時 3 小時），且容易因 source 缺少某些欄位值而引發 python exception。另外於 import stage 時若持續發生 error，即使在 conf 檔案設定忽略，也會發生卡死的情況。
                                                                                                                           
   並且，只要一卡死，harvest 工作就不算完成，要重新開始僅能清除所有已下載下來的 metadata，再執行一次。                         
                                                                                                                               
   * 目前僅測試 csw harvester 成功。

使用步驟說明：

a. 新增 harvest source：                                                                                                       
                                                                                                                               
   使用瀏覽器開啟 SITE_URL/harvest，選取右上之 "Add Harvest source"，依照畫面輸入 source 網址及選取 source 類別。              
   .. note::                                                                                                                   
                                                                                                                               
      若您有成功安裝 ckanext-spatial 套件並啟用上述三個 plugins，應該可以看到 "CKAN, CSW Server, Web Accessible Folder (WAF), Single spatial metadata document" 四種 source 類別                                                                              
                                                                                                                               
b. 執行 harvest 工作（手動）：                                                                                                 
                                                                                                                               
   進入 virtualenv，執行 gather 與 fetch handler：                                                                             
                                                                                                                               
   .. code-block:: bash                                                                                                        
                                                                                                                               
      (pyenv) $ paster --plugin=ckanext-harvest harvester gather_consumer -c /etc/ckan/default/production.ini                  
      (pyenv) $ paster --plugin=ckanext-harvest harvester fetch_consumer -c /etc/ckan/default/production.ini                   
                                                                                                                               
   .. note::     
                                                                                                              
      請勿關閉這兩個 handler                                                                                                   
                                                                                                                               
   使用瀏覽器開啟 SITE_URL/harvest，進入剛才建立的 harvest source，選擇右上的「管理者」按鈕，在接下來的頁面選取 "Reharvest"，將此 harvest 工作送入排程。

   最後進入 virtualenv，執行 run handler：                                                                                     
                                                                                                                               
   .. code-block:: bash                                                                                                        
                                                                                                                               
      (pyenv) $ paster --plugin=ckanext-harvest harvester run -c /etc/ckan/default/production.ini                              
                                                                                                                               
   即會立即開始執行剛才加入的工作排程。

   .. note::

      手動執行時 harvest 工作並不會自行停止，因為上述 paster harvester run 指令同時也用來確認 harvest 工作是否完成。因此若您確定 harvest 工作已經完成（或已發生錯誤），可以再次執行 run 指令，即可透過下述 d. 的方式檢視此次工作的結果
      
c. 執行 harvest 工作（自動）：

d. 確認 harvest 工作的執行狀況：

   我們可以在網頁介面，harvest source 的「管理者」頁面確認 harvest 工作的執行狀況，包括錯誤、新增、更新、完成的資料集數目，如下圖所示：

   .. image:: harvest-job-status.png
                                                                                                                         
系統需求
---------
* Python (2 or 3) 安裝於 virtualenv
* ckan (>=1.8)
* solr (>=3.1)
* `ckanext-harvest <https://github.com/okfn/ckanext-harvest>`_ (ckan 延伸套件)：外掛 :ref:`spatial-harvesters` 需要

.. note::

   * 若依照本文件的教學安裝 ckan，你應該已經擁有上述前三套套件
   * Dataset Extent Map 與 Spatial Search Widget 兩個 snippets 需要 ckan>=2.0

安裝
-----
a. 安裝 ckanext-spatial 套件：

   .. code-block:: bash

      (pyenv) $ pip install -e git+https://github.com/okfn/ckanext-spatial.git@release-v2.0#egg=ckanext-spatial

   .. note::

      release-v2.0 請自行依 ckan 版本替換之

b. 安裝其他需要的 Python 套件：

   .. code-block:: bash

      (pyenv) $ pip install -r pip-requirements.txt

c. 安裝 PostGIS：

   請直接參考 `官方的安裝說明 <https://github.com/okfn/ckanext-spatial#setting-up-postgis>`_ 。

d. 修改 ckan 設定檔：

   打開 ckan 設定檔（一般位於 /etc/ckan/default/），並加入：

   .. code-block:: python
      
      ckanext.spatial.search_backend = solr

   並修改 ckan.plugins 參數，增加需要的外掛（參見下文介紹）。

e. 修改 solr schema：

   打開 solr schema（一般位於 /usr/share/solr/collection1/conf/solrconfig.xml），找到 <fields> 區段，加上：

   .. code-block:: xml
      
      <fields>
          <!-- ... -->
          <field name="bbox_area" type="float" indexed="true" stored="true" />
          <field name="maxx" type="float" indexed="true" stored="true" />
          <field name="maxy" type="float" indexed="true" stored="true" />
          <field name="minx" type="float" indexed="true" stored="true" />
          <field name="miny" type="float" indexed="true" stored="true" />
      </fields>

f. 新增 Spatial Search Widget：

   打開 cakn source 目錄下的 ./ckan/templates/package/search.html，在 {% block secondary_content %} 段落中加入

   .. code-block:: python

      {% snippet "spatial/snippets/spatial_query.html" %}

g. 新增 Dataset Extent Map (widget)：

   打開 cakn source 目錄下的 ./ckan/templates/package/read.html，在最後加入

   .. code-block:: python

      {% block secondary_content %}
        {{ super() }}

        {% set dataset_extent = h.get_pkg_dict_extra(c.pkg_dict, 'spatial', '') %}
        {% if dataset_extent %}
          {% snippet "spatial/snippets/dataset_map_sidebar.html", extent=dataset_extent %}
        {% endif %}

      {% endblock %}
