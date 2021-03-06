.. EN-Revision: none
.. _performance.database:

Zend_Dbパフォーマンス
==============

``Zend_Db``\ はデータベースを抽象化するレイヤーで、 *SQL*\ 操作のための共通 *API*\
を提供する意図があります。 ``Zend\Db\Table``\ はテーブルデータのゲートウェイで、
テーブルレベルでの抽象的な共通のデータベース操作を提供する意図があります。
それらの抽象的な性質とそれらの操作を行なうために隠れて行なうマジックのせいで、
しばしばパフォーマンスのオーバーヘッドをもたらします。

.. _performance.database.tableMetadata:

テーブルのメタデータを取得する際にZend\Db\Tableによってもたらされる オーバーヘッドをどのようにしたら減らせますか？
----------------------------------------------------------------

できる限り使い方を簡単に保つために、
また、開発最中にスキーマの変更を絶えずサポートするためにも、 ``Zend\Db\Table``\
はあるマジックを隠れて行ないます:
使い始める時にテーブルのスキーマを読み込んでオブジェクトのメンバに保存します。
この操作は一般的に高くつき、データベースを気にかけません。
それは製品のボトルネックに寄与します。

幸運なことにその状況を改善する技術があります。

.. _performance.database.tableMetadata.cache:

メタデータキャッシュの利用
^^^^^^^^^^^^^

``Zend\Db\Table``\ ではテーブルのメタデータをキャッシュする ために ``Zend_Cache``\
を任意で利用できます。 これはデータベース自身からメタデータを読み込むよりも、
一般的に早くアクセスでき、高くつきません。

:ref:`Zend\Db\Table
のドキュメントにメタデータをキャッシュすることについて情報があります。
<zend.db.table.metadata.caching>`

.. _performance.database.tableMetadata.hardcoding:

テーブル定義でメタデータをハードコーディングする
^^^^^^^^^^^^^^^^^^^^^^^^

1.7.0では、 ``Zend\Db\Table``\ は
:ref:`テーブルの定義でメタデータをハードコーディングすることをサポート
<zend.db.table.metadata.caching.hardcoding>` することもします。
これは高度な使用例であり、テーブルのスキーマが変更されそうにないと知っているか、
または定義を最新状態に保ち続けられる時のみ利用されるべきでしょう。

.. _performance.database.select:

Zend\Db\Selectで生成されたSQLがインデックスにヒットしません。 どのようにしたらより良く出来ますか？
----------------------------------------------------------

``Zend\Db\Select``\ は比較的その仕事が得意です。
ただし、結合や副選択を必要とする複雑なクエリを実行するなら、
しばしばかなりの素人になりえます。

.. _performance.database.select.writeyourown:

自分で最適化したSQLを書く
^^^^^^^^^^^^^^

現実的な唯一の答えは自分で *SQL*\ を書くことです; 自分で用意できるなら、
``Zend_Db``\ で必ず ``Zend\Db\Select``\ を使わずに、 *SQL*\ のselect文を調整する
ことがもっとも完璧に筋の通った道です。

``EXPLAIN`` をクエリに対して実行して、
最もパフォーマンスのあるインデックスを本当に当てるまで、
さまざまな取り組みをテストしてください。
それからクラスのプロパティーや定数として *SQL*\
をハードコーディングしてください。

もし *SQL*\ が変数の引数を必要とする場合、 *SQL*\
にプレースホルダーを用意してください。 そして、 *SQL*\ に値を挿入するために
``vsprintf`` と ``array_walk`` の組み合わせを利用してください。

.. code-block:: php
   :linenos:

   // $adapter はDBアダプターです。Zend\Db\Tableでは、
   // $this->getAdapter() を使ってそれを参照します
   $sql = vsprintf(
       self::SELECT_FOO,
       array_walk($values, array($adapter, 'quoteInto'))
   );


