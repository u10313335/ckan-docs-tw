.. ckan-2.0_ins documentation master file, created by
   sphinx-quickstart on Sun Jul 21 11:15:04 2013.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

CKAN2 安裝使用教學
========================================

CKAN 是著名的開放原始碼資料入口平台（Data Portal Platform），由非營利的 `CKAN Association <http://ckan.org/about/association/>`_ 支持發展。

他的功能非常多，除了 data repository 外，還支援 visualization、search、tag、revision、share、organization...，更有許多的 plugins 可以強化其功能。

使用 CKAN 最有名的專案，係英國政府開放資料平台 `data.gov.uk <http://data.gov.uk>`_ 。

CKAN 使用 Pylons 網頁框架開發，template 使用 jinja2，多國語言採用 Babel ，資料庫使用 PostgreSQL，ORM 是 Pylons 推薦的 SQLAlchemy，搜尋功能則採用 Apache Solr。

這篇教學文將說明如何從無到有安裝一個 CKAN2 網站。環境為 Ubuntu 14.04 LTS，大致按照官方文件 `Install from Source <http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html>`_ 的方式。其他相關的使用心得也會一併公布於此。


.. toctree::
   :maxdepth: 2

   install
   deployment
   datastore-datapusher
   ckanext-spatial
   ckanext-geoview
   ckanext-harvest
   rdf

Indices and tables
==================

* :ref:`search`
