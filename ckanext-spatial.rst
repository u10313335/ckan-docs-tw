ckanext-spatial
================

ckanext-spatial 是一個 ckan 的延伸套件 (extension)，提供地理資訊相關功能。

詳細介紹可以參考 `ckanext-spatial 的官方 github <https://github.com/okfn/ckanext-spatial>`_ 。

系統需求
---------
* Python (2 or 3) 安裝於 virtualenv
* ckan (>=1.8)
* solr (>=3.1)
* ckanext-harvest (ckan 延伸套件)：外掛 :ref:`cswserver` 需要

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

e. 新增 Spatial Search Widget：

   打開 cakn source 目錄下的 ./ckan/templates/package/search.html，在 {% block secondary_content %} 段落中加入

   .. code-block:: python

      {% snippet "spatial/snippets/spatial_query.html" %}

f. 新增 Dataset Extent Map (widget)：

   打開 cakn source 目錄下的 ./ckan/templates/package/read.html，在最後加入

   .. code-block:: python

      {% block secondary_content %}
        {{ super() }}

        {% set dataset_extent = h.get_pkg_dict_extra(c.pkg_dict, 'spatial', '') %}
        {% if dataset_extent %}
          {% snippet "spatial/snippets/dataset_map_sidebar.html", extent=dataset_extent %}
        {% endif %}

      {% endblock %}


外掛主要功能簡介
-----------------

spatial_metadata
^^^^^^^^^^^^^^^^^^
建立地理空間資訊之索引。

Spatial Search Widget
^^^^^^^^^^^^^^^^^^^^^^
按地圖搜尋資料集 "spatial" 欄位的地理空間資訊，僅支援 solr 3.1+。

欲使用此功能，請在 ckan.plugins 加入 spatial_metadata 與 spatial_query。


Dataset Extent Map
^^^^^^^^^^^^^^^^^^^
以地圖顯示資料集 "spatial" 欄位所述之地理空間資訊 (僅支援 geojson 格式)。

欲使用此功能，請在 ckan.plugins 加入 spatial_metadata 與 spatial_query。

如下圖所示，在「額外的資訊」中填寫的 spatial geojson 資訊，將顯示在左下角的 Dataset extent 中。

.. image:: extent-map.png
   :scale: 70 %


WMS Preview
^^^^^^^^^^^^
用來檢視 wms 服務所能提供的地理範圍 (GetCapabilities)。

欲使用此功能，請在 ckan.plugins 加入 wms_preview。

以 `NASA Earth Observations <http://neowms.sci.gsfc.nasa.gov/wms/wms?version=1.1.1&service=WMS&request=GetCapabilities>`_ 為例：

.. image:: wms-preview.png
   :scale: 70 %

GeoJSON Preview
^^^^^^^^^^^^^^^^^
以地圖檢視 GeoJSON 檔案。

欲使用此功能，請在 ckan.plugins 加入 geojson_preview 與 resource_proxy。

支援 geojson 與 gjson 兩種檔案格式名稱定義。

.. _cswserver:

CSW Server
^^^^^^^^^^^
提供 WMS 服務介面（研究中）。

Spatial Harvesters
^^^^^^^^^^^^^^^^^^^
提供地理空間相關的 harvester（研究中）。
