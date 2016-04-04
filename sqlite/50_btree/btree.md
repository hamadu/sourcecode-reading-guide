# BTREE

役割 :

# 用語

`free-list leaf page` :
`the pointer map (of page)` :


`BtShared *pBt` : 各BTreeオブジェクトで共通して使う部分
`BtShared *pBt; pBt->pPager` : ページャー
`BtShared *pBt; pBt->pHasContent bitvec` :

`MemPage *pPage` : Cellを複数含むページ。B-Tree層で使われる
`DbPage *pDbPage` : Pagerから得られる

`CellInfo *pInfo`

`BtCursor *cursor` :
`BtCursor *cursor;  cursor->apPage[] pages` :
`BtCursor *cursor;  cursor->iPage` : カーソルが持つページの数(-1)
`BtCursor *cursor;  cursor->(pKey, nKey)` : カーソルのキー番号(データサイズ)とそのデータへのポインタ
`BtCursor *cursor;  cursor->skipNext` :
`BtCursor *cursor;  cursor->eState` : カーソルの状態
  `CURSOR_VALID`
  `CURSOR_SKIPNEXT`
  `CURSOR_REQUIRESEEK`
  `CURSOR_INVALID`
`BtCursor *cursor;  cursor->curFlags` :
  `BTCF_ValidNKey`
  `BTCF_ValidOvfl`
  `BTCF_AtLast`

`packed index record` : OP_MakeRecordによって作成される？




## open
## close


## `static int querySharedCacheTableLock(Btree *p, Pgno iTab, u8 eLock)`

指定ページのロック(READ_LOCK or WRITE_LOCK)が得られるか否か

## `static int setSharedCacheTableLock(Btree *p, Pgno iTable, u8 eLock)`

指定ページに実際にロックをかける

## `static void clearAllSharedCacheTableLocks(Btree *p)`

Btreeが持つロックを全て解除

## `static void downgradeAllSharedCacheTableLocks(Btree *p)`

## `static void btreeClearHasContent(BtShared *pBt)`

BtShared.pHasContent bitvec をゼロに。

## `static void btreeReleaseAllCursorPages(BtCursor *pCur)`

cursorが持つ全ページをリリース

## `static int saveCursorKey(BtCursor *pCur)`
## `static int saveCursorPosition(BtCursor *pCur)`

cursor->(pKey, nKey) をセット

## `saveAllCursors(BtShared *pBt, Pgno iRoot, BtCursor *pExcept)`

** This routine is called just before cursor pExcept is used to modify the
** table, for example in BtreeDelete() or BtreeInsert().

pExcept以外のカーソル位置を全て保存

## `void sqlite3BtreeClearCursor(BtCursor *pCur)`

カーソル位置をゼロ、状態を CURSOR_INVALID に

##

```
static int btreeMoveto(
  BtCursor *pCur,     /* Cursor open on the btree to be searched */
  const void *pKey,   /* Packed key if the btree is an index */
  i64 nKey,           /* Integer key for tables.  Size of pKey for indices */
  int bias,           /* Bias search to the high end */
  int *pRes           /* Write search results here */
)
```

pKeyがパックされてたらunpackして
sqlite3BtreeMovetoUnpacked(pCur, pIdxKey, nKey, bias, pRes);
を呼ぶ

## `static int btreeRestoreCursorPosition(BtCursor *pCur)`

saveCursorPosition() で保存したカーソル位置を復元する。
int skipNext;
btreeMoveto(pCur, pCur->pKey, pCur->nKey, 0, &skipNext)
pCur->skipNext = skipNext;

## `int sqlite3BtreeCursorRestore(BtCursor *pCur, int *pDifferentRow)`

btreeRestoreCursorPosition(p) を必要なら呼ぶ
** such as a btree rebalance or a row having been deleted out from under the cursor


## `static Pgno ptrmapPageno(BtShared *pBt, Pgno pgno)`

ページ番号に対する the page number for the pointer-map を返す
** Return 0 (not a valid page) for pgno==1 since there is no pointer map associated with page 1

## `static void ptrmapPut(BtShared *pBt, Pgno key, u8 eType, Pgno parent, int *pRC){`

(指定ページ番号<key>を、指定したタイプ、親ページ番号を指すように更新。)
Write an entry into the pointer map.
** This routine updates the pointer map entry for page number 'key'
** so that it maps to type 'eType' and parent page number 'pgno'.

// iPtrmap = PTRMAP_PAGENO(pBt, key);
// pDbPage = sqlite3PagerGet(pBt->pPager, iPtrmap, &pDbPage, 0);
// pPtrmap = sqlite3PagerGetData(pDbPage)
// pPtrmap[offset] : タイプ
// get4byte(&pPtrmap[offset+1]) : 親番号

## `static int ptrmapGet(BtShared *pBt, Pgno key, u8 *pEType, Pgno *pPgno)`

(指定ページ番号<key>のデータを取り出す。タイプ : `*pEType` and 親ページ番号 : `*pPgno` が返却される)

## `static SQLITE_NOINLINE void btreeParseCellAdjustSizeForOverflow(略)`

** for the case when the cell does not fit entirely on a single B-tree page
セルが単一のページに収まらない時、必要な調整を行う

##

```
** btreeParseCellPtr()        =>   table btree leaf nodes
** btreeParseCellNoPayload()  =>   table btree internal nodes
** btreeParseCellPtrIndex()   =>   index btree nodes
```

指定セルのコンテンツを読みだして、`CellInfo *pInfo` に入れる。

## `static int defragmentPage(MemPage *pPage)`

指定ページの「Defragment」を行う。ページ内の全セルがページの最後に移り、巨大なフリースペースをヘッダとデータの間に作る。

## `static u8 *pageFindSlot(MemPage *pPg, int nByte, int *pRc)`

ページ内のフリースペースを探してそのポインタを返す

## ```static int allocateSpace(MemPage *pPage, int nByte, int *pIdx)```

ページ内のフリースペースを探してそのポインタを返す。必要に応じ「Defragment」する。

## `static int btreeInitPage(MemPage *pPage)`

ページを初期化

## `static MemPage *btreePageFromDbPage(DbPage *pDbPage, Pgno pgno, BtShared *pBt)`

** Convert a DbPage obtained from the pager into a MemPage used by the btree layer.

##

```
static int btreeGetPage(
  BtShared *pBt,       /* The btree */
  Pgno pgno,           /* Number of the page to fetch */
  MemPage **ppPage,    /* Return the page in this parameter */
  int flags            /* PAGER_GET_NOCONTENT or PAGER_GET_READONLY */
)
```

pagerからMemPageを得る。

##

```
static int getAndInitPage(
  BtShared *pBt,                  /* The database file */
  Pgno pgno,                      /* Number of the page to get */
  MemPage **ppPage,               /* Write the page pointer here */
  BtCursor *pCur,                 /* Cursor to receive the page, or NULL */
  int bReadOnly                   /* True for a read-only page */
)
```


##

```
int sqlite3BtreeOpen(
  sqlite3_vfs *pVfs,      /* VFS to use for this b-tree */
  const char *zFilename,  /* Name of the file containing the BTree database */
  sqlite3 *db,            /* Associated database handle */
  Btree **ppBtree,        /* Pointer to new Btree object written here */
  int flags,              /* Options */
  int vfsFlags            /* Flags passed through to sqlite3_vfs.xOpen() */
)
```

指定のデータベースファイルをオープンし、BTreeオブジェクト、BtSharedオブジェクト、そこで使われるVFSを初期化。

: sqlite3PagerOpen(pVfs, &pBt->pPager, zFilename, EXTRA_SIZE, flags, vfsFlags, pageReinit);
  : sqlite3PagerReadFileheader
: sqlite3PagerSetBusyhandler
: sqlite3PagerSetPagesize(pBt->pPager, &pBt->pageSize, nReserve);

## `int sqlite3BtreeSetSpillSize(Btree *p, int mxPage)`

ページ数が mxPage を超えると、pagerが journal に page を「零す」ようにする。

## `int sqlite3BtreeSetMmapLimit(Btree *p, sqlite3_int64 szMmap)`

メモリにマップできるデータベースファイル数をセット

## `int sqlite3BtreeSetPageSize(Btree *p, int pageSize, int nReserve, int iFix)`

ページサイズをセット。must be a power of 2 between 512 and 65536
: sqlite3PagerSetPagesize(pBt->pPager, &pBt->pageSize, nReserve)

## `static int newDatabase(BtShared *pBt)`

空のファイルを指してたら新DBを作る

## `int sqlite3BtreeBeginTrans(Btree *p, int wrflag)`

トランザクションを開始しようとする

## `static void btreeEndTransaction(Btree *p)`

## `int sqlite3BtreeInsert()` : 新エントリをカーソル位置に追加する

```
int sqlite3BtreeInsert(
  BtCursor *pCur,                /* Insert data into the table of this cursor */
  const void *pKey, i64 nKey,    /* The key of the new record */
  const void *pData, int nData,  /* The data of the new record */
  int nZero,                     /* Number of extra 0 bytes to append to data */
  int appendBias,                /* True if this is likely an append */
  int seekResult                 /* Result of prior MovetoUnpacked() call */
)
```

## `int sqlite3BtreeDelete(BtCursor *pCur, int bPreserve)` : カーソル位置のエントリの削除

## `static int btreeCreateTable(Btree *p, int *piTable, int createTabFlags)` : テーブルの作成
## `int sqlite3BtreeCreateTable(Btree *p, int *piTable, int flags)`



# ページ関連

##

```
static int allocateBtreePage(
  BtShared *pBt,         /* The btree */
  MemPage **ppPage,      /* Store pointer to the allocated page here */
  Pgno *pPgno,           /* Store the page number here */
  Pgno nearby,           /* Search for a page near this one */
  u8 eMode               /* BTALLOC_EXACT, BTALLOC_LT, or BTALLOC_ANY */
)
```

新ページを割当


# セル関連

## セルに対する操作

### `static int clearCell(MemPage *pPage, unsigned char *pCell, u16 *pnSize)`
### `static void dropCell(MemPage *pPage, int idx, int sz, int *pRC)`
### `static int fillInCell()`
```
static int fillInCell(
  MemPage *pPage,                /* The page that contains the cell */
  unsigned char *pCell,          /* Complete text of the cell */
  const void *pKey, i64 nKey,    /* The key */
  const void *pData,int nData,   /* The data */
  int nZero,                     /* Extra zero bytes to append to pData */
  int *pnSize                    /* Write cell size here */
)
```
### `static void dropCell(MemPage *pPage, int idx, int sz, int *pRC)`

pPage からのリファレンスを消すだけ。

### `static void insertCell()`

```
static void insertCell(
  MemPage *pPage,   /* Page into which we are copying */
  int i,            /* New cell becomes the i-th cell of the page */
  u8 *pCell,        /* Content of the new cell */
  int sz,           /* Bytes of content in pCell */
  u8 *pTemp,        /* Temp storage space for pCell, if needed */
  Pgno iChild,      /* If non-zero, replace first 4 bytes with this value */
  int *pRC          /* Read and write return code from here */
)
```


# カーソル関連

## 移動

### `int sqlite3BtreeFirst(BtCursor *pCur, int *pRes)`
### `int sqlite3BtreeLast(BtCursor *pCur, int *pRes)`

###

```
int sqlite3BtreeMovetoUnpacked(
  BtCursor *pCur,          /* The cursor to be moved */
  UnpackedRecord *pIdxKey, /* Unpacked index key */
  i64 intKey,              /* The table key */
  int biasRight,           /* If true, bias the search to the high end */
  int *pRes                /* Write search results here */
)
```

指定キー(INTKEY tablesなら intKey, index tablesなら pIdxKey)の近場にカーソルを動かす。
大事なロジックが書かれている？

### `static SQLITE_NOINLINE int btreeNext(BtCursor *pCur, int *pRes)`
### `static SQLITE_NOINLINE int btreePrevious(BtCursor *pCur, int *pRes)`
### `int sqlite3BtreeNext(BtCursor *pCur, int *pRes)`
### `int sqlite3BtreePrevious(BtCursor *pCur, int *pRes)`

カーソルを前後のエントリを指すように動かす。


### `static int moveToRoot(BtCursor *pCur)`
### `static void moveToParent(BtCursor *pCur)`
### `static int moveToChild(BtCursor *pCur, u32 newPgno)`
### `static int moveToRightmost(BtCursor *pCur)`
### `static int moveToLeftmost(BtCursor *pCur)`

## 今いる位置のデータを読む

### `int sqlite3BtreeKey(BtCursor *pCur, u32 offset, u32 amt, void *pBuf)` : キー
### `int sqlite3BtreeData(BtCursor *pCur, u32 offset, u32 amt, void *pBuf)` : なかみ

(内部で accessPayload(pCur, offset, amt, pBuf, 0) を呼んでる)

## バランシング

### `static int balance(BtCursor *pCur)`

状況に応じて

balance_quick()
balance_deeper()
balance_nonroot()

のどれかを呼ぶ

### `static int balance_deeper(MemPage *pRoot, MemPage **ppChild)`

BTreeのルートページが overfull の場合に呼ばれる。
新しい子ページが作られ、 overflow cells を含む全データがコピーされる。
その後、ルートページは空になり、右ポインタが新ページを指すようになる。

### `static int balance_quick(MemPage *pParent, MemPage *pPage, u8 *pSpace)`

よくあるケースで呼ばれる。そのケースとは、新エントリが最も右端に配置される時。
より一般的な処理である下の `balance_nonroot` よりも高速に動作する。

###

```
static int balance_nonroot(
  MemPage *pParent,               /* Parent page of siblings being balanced */
  int iParentIdx,                 /* Index of "the page" in pParent */
  u8 *aOvflSpace,                 /* page-size bytes of space for parent ovfl */
  int isRoot,                     /* True if pParent is a root-page */
  int bBulk                       /* True if this call is part of a bulk load */
)
```

全ページが同程度の free space を持つようにバランスさせる。
