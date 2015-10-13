ckanext-geoview
================

ckanext-geoview 是一個 CKAN 的擴充套件 (extension)，提供地理資料預覽功能。

詳細介紹可以參考 `ckanext-geoview 的 github 頁面 <https://github.com/ckan/ckanext-geoview>`_ 。

功能簡介
--------

OpenLayers Viewer
^^^^^^^^^^^^^^^^^
此功能可以地圖方式呈現 WMS、WFS、GeoJSON、GML、KML、Google Fusion Tables 等地理資料/服務涵括的地理範圍。

欲使用此功能，請在 ckan.plugins 加入 resource_proxy 與 geo_view (2.2 含以下版本則為 geo_preview)。

若僅想預覽部分格式，可在 CKAN 設定檔加入 ckanext.geoview.ol_viewer.formats 變數，詳細設定方式可參考 `擴充套件的說明 <https://github.com/ckan/ckanext-geoview#openlayers-viewer>`_ 。

Leaflet GeoJSON Viewer
^^^^^^^^^^^^^^^^^^^^^^
以地圖檢視 GeoJSON 檔案。支援 ``geojson`` 與 ``gjson`` 兩種檔案格式名稱定義。

欲使用此功能，請在 ckan.plugins 加入 resource_proxy 與 geojson_view (2.2 含以下版本則為 geojson_preview)。

Leaflet WMTS Viewer
^^^^^^^^^^^^^^^^^^^^^^
以地圖檢視 WMTS 服務圖層。支援 ``wmts`` 格式名稱定義。

欲使用此功能，請在 ckan.plugins 加入 resource_proxy 與 wmts_view (2.2 含以下版本則為 wmts_preview)。


安裝
-----
自 github ckeckout source 並安裝：

   .. code-block:: bash

      $ cd /usr/lib/ckan/default/src
      $ git clone https://github.com/ckan/ckanext-geoview.git
      $ cd ckanext-geoview
      (pyenv) $ pip install -e . 
