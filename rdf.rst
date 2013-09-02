Linked Data and RDF
===================

資源描述架構（Resource Description Framework）及 RDF Schema 係由 W3C 制定，用來解決資源描述問題的規範，利用階層式的概念及屬性描述資料的 metadata。

ckan 自 1.7 版後開始內建支援 RDF 格式輸出，使用非常容易。

使用方法
--------

以下兩種方式均可獲得特定資料集的 RDF 格式描述：

方法一
^^^^^^

.. code-block:: bash
   
   curl -L -H "Accept: application/rdf+xml" http://thedatahub.org/dataset/gold-prices

方法二
^^^^^^

.. code-block:: bash

   curl -L http://thedatahub.org/dataset/gold-prices.rdf

Schema Mapping
--------------

ckan 資料集的所有欄位都可以自 RDF 取得，其對應請參見官方說明： `Schema Mapping <http://docs.ckan.org/en/ckan-2.0.2/linked-data-and-rdf.html#schema-mapping>`_
