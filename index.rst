.. ckan-2.0_ins documentation master file, created by
   sphinx-quickstart on Sun Jul 21 11:15:04 2013.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

ckan 2.0 安裝使用教學
========================================

ckan 是著名的開放原始碼資料入口平台（Data Portal Platform），
由 Open data 界大名鼎鼎的 Open Knowledge Foundation（OKF）支持發展。

他的功能非常多，除了 data repository 外，還支援 visualize、search、tag、revision、share、organization…，
更有許多的 plugins 可以強化其功能。

使用 ckan 最有名的專案，係英國政府開放資料平台 data.gov.uk。

ckan 使用以 Python 為基礎的 Pylons 網頁框架開發，template 使用 jinja（神社）2，多國語言支援採用 Babel 系統，資料庫使用 PostgreSQL，ORM 是 Pylons 推薦的 SQLAlchemy，搜尋功能則使用 Apache Solr 實作，同時搭配 jetty 作為 servlet container。

這篇教學文將說明如何自 \*nix OS 從無到有安裝一個 ckan 2.0 系統。環境為 Linux Mint 14 (based on Ubuntu 12.10），大致按照官方文件 `Install from Source <http://docs.ckan.org/en/ckan-2.0/install-from-source.html>`_ 的方式。其他相關的使用心得也會一併公布於此。


.. toctree::
   :maxdepth: 2

   install
   deployment
   ckanext-spatial


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

