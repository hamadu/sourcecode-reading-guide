# BTREE概要

## Btreeの仕組み


## Btreeとページ

各テーブルごとにただひとつの root page を持つ。

各ページは複数のテーブルのエントリと、それによって「分割される」子供のページヘのポインタを持つ。

```
----------------------------------------------------------------
|  Ptr(0) | Key(0) | Ptr(1) | Key(1) | ... | Key(N-1) | Ptr(N) |
----------------------------------------------------------------
```

## ページとセル

page は複数の cell を持つ。

## ページのカーソル

BtSharedは「特定のページ」を指すカーソルを持っていて、こいつがVDBEの命令であちこち動いたりする。
たとえばINDEX付きデータSELECTの際は、カーソルがルートページからINDEX情報を頼りに所定の箇所に移動する。
また、データのINSERT時には（殆どの場合）カーソルが最後のページに移動し、そのページの空きセルにデータが書き込まれる。

## バランシング処理！

データがINSERTされたりDELETEされたりした場合、Btreeは必要に応じて「バランス」処理を行う。
バランス処理は、各ページが同程度の free space を持つように行われる。



カーソルはその名の通り特定のページを指しており

「データ検索」の内部的な本質はこのページを行ったり来たりすることである。

また、ページは「バランス」機構を内蔵しており、



## 構造体達

### Btree

* db : DBコネクション
* pBt : BtShared
* pNext, pPrev : 同じDBの他のBtreeを指す

### BtShared

各データベースファイルで共通のデータを表す。

* pCursor : カーソル(の一覧)
* pPager : ページャー
* pHasContent : bitvecらしい

### BtCursor

BTree構造の中で、今注目してる位置をこのカーソルで表す。

* pBtree : カーソルが属するBtreeオブジェクト
* pgnoRoot : ルートページのページ番号
* apPage[] pages : rootから現在のpageまで下っていった時の履歴(MemPageへのポインタのリスト)
* iPage : 今の深さ
* pKey, nKey
* skipNext : ?
* eState
* curFlags

### MemPage

* pBt : BtShared
* pDbPage :
* pgno : ページ番号

### DbPage

* ?

### CellInfo

* pPayload : Pointer to the start of payload
* nPayload : Bytes of payload
* nLocal : Amount of payload held locally, not on overflow
* nSize : Size of the cell content on the main b-tree page
