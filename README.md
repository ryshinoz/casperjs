# CasperJS

CasperJSメモ

## 試したバージョン

```bash
$ casperjs --version
1.1.0-beta2
$ phantomjs --version
1.9.2
```

## 機能テスト

公式サイトのやつ、説明いらない

```coffeescript
casper.test.begin 'Google search retrieves', 5, (test) ->
  casper.start 'http://www.google.co.jp', ->
    test.assertTitle 'Google', 'google homepage title is the one expected'
    test.assertExists 'form[action="/search"]', 'main form is found'
    @fill 'form[action="/search"]', { q: 'casperjs' }, true

  casper.then ->
    test.assertTitle 'casperjs - Google 検索', 'google title is ok'
    test.assertUrlMatch /q=casperjs/, 'search term has been submitted'
    test.assertEval ->
      __utils__.findAll('h3.r').length >= 10
    , 'google search for "casperjs" retrives 10 or more results'

  casper.run ->
    test.done()
```

## JavaScriptエラー取得

[page.errorイベント](http://docs.casperjs.org/en/latest/events-filters.html#page-error)を利用する

```coffeescript
casper.on 'page.error', (msg, trace) ->
  @echo 'Error:    ' + msg, 'ERROR'
  @echo 'file:     ' + trace[0].file, 'WARNING'
  @echo 'line:     ' + trace[0].line, 'WARNING'
  @echo 'function: ' + trace[0]['function'], 'WARNING'
```

## Cookies

`phantom.cookies`を利用する

取得例（ファイルに保存）

```coffeescript
fs = require 'fs'
cookies = JSON.stringify phantom.cookies
fs.write cookies_file, cookies, 644
```

設定例（ファイルから読み込み）

```coffeescript
fs = require 'fs'
if fs.exists cookies_file
    data = fs.read cookies_file
    phantom.cookies = JSON.parse data
```

`--incluedes`オプションとあわせるとテスト毎にcookiesを設定するなどができる（まだ試していない）

## ユーザーエージェント

[casper.userAgent](http://casperjs.readthedocs.org/en/latest/modules/casper.html#useragent)を利用する

```coffeescript
casper.userAgent 'Mozilla/5.0 (iPhone; CPU iPhone OS 6_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/6.0 Mobile/10A403 Safari/8536.25'
```

## Jenkins連携

テスト結果出力にxUnitフォーマットが対応しているのでそれを利用する

```bash
$ casperjs test googletesting.coffee --xunit=result.xml 
```

## テストのイベント処理

テスト時には以下のイベントが取得できる

- `success` - 成功
- `fail` - 失敗
- `skipped` - スキップ

### 例）テスト失敗時に`dump`する

```coffeescript
casper.test.on 'fail', (result) ->
    require('utils').dump result
```

`result`にはテスト結果やテスト実施したファイルが設定されています

```json
{
    "success": false,
    "type": "assertEquals",
    "standard": "Subject equals the expected value",
    "file": "test.coffee",
    "doThrow": true,
    "values": {
        "subject": 1,
        "expected": 2
    },
    "line": 9,
    "suite": "test",
    "time": 60
}
```

## 画面キャプチャ

[casper.capture](http://casperjs.readthedocs.org/en/latest/modules/casper.html#capture)を利用する

テスト失敗時に画面キャプチャする

```coffeescript
casper.test.on 'fail', ->
  casper.capture('fail.png')
```

背景色のやつを後で記述

## Links（参考にしたサイトなど）

- [CasperJS](http://casperjs.org/)
- [PhantomJS](http://phantomjs.org/)
- [続・Casper.JSのススメ](http://saisa.hateblo.jp/entry/2013/09/12/222035)
- [Functional testing with CasperJS](http://code.hootsuite.com/functional-testing-with-casperjs/)
- [Scrape Site for JS Errors with Casper.js](https://coderwall.com/p/uzaaaw)
