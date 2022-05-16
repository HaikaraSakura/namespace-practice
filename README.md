# 名前空間とPSR-4

PSR-4は名前空間の利用方法に関する規約である。  
名前空間はPHP5.3から導入された。クラス名などの衝突を回避する名前解決のための機能。  

昨今の、特にComposer対応のライブラリは必ず名前空間を利用しているので、概要を把握しておきたい。

## 名前空間はどのような問題を解決するか

何らかのライブラリを利用しているとき、そのライブラリ内部で`Admin`というクラス名が使われていると、  
アプリケーションの開発者は同じ名前のクラスを作ることができず、`MyAdmin`のように命名することで衝突を避ける必要がある。  
ところが別のライブラリを追加で導入したとき、今度はその内部で`MyAdmin`が使われていたりするのである。  

また、自分たちが開発するアプリケーション内においても同様の懸念があり、  
たとえば一覧画面のControllerのクラス名を、単純に`ListController`などとすると、他の一覧画面のControllerと見分けがつかないので、  
`AdListController`や`UsrListController`のように、プレフィックスを付けることで区別しなければならない。  

名前空間はこうした不都合を解決するべく導入された言語機能である。  
PHPが中〜大規模開発でも広く利用されるようになったことで必要性が生じた機能といえる。

## namespace

```PHP
namespace App\Admin;

class IndexController {
    public function get() {...}
}

new \App\Admin\IndexController;
```

`IndexController`という名前に、`App\Admin`という名字を与えているようなイメージ。  

これによって、クラス名そのものにプレフィックスを付けて命名する必要がなくなるので、  
これまで`AdIndexController`としていたところを、`\App\Admin\IndexController`などとすることができる。

同じIndexControllerでも、名前空間が異なれば別のクラスになる。  

```PHP
$admin_index_controller = new \App\Admin\IndexController;
$products_index_controller new \App\Products\IndexController;

var_dump($admin_index_controller instanceof $products_index_controller); // false
```

## use

同一のファイル内で何度も同じクラス名を記述する必要がある場合、useで省略することができる。  

```PHP
// このファイル内でConvertといえば\Knp\ImageHandler\Convertのことである、という宣言。
use \Knp\ImageHandler\Convert;

$main = new Convert('./main_image.png');
$usecase1 = new Convert('./usecase_image1.png');
$usecase2 = new Convert('./usecase_image2.png');
```

同じファイル内で名前空間の異なる同じクラス名を呼び分けたい場合、  
完全修飾名で記述する以外に、別名を付与することでも名前解決ができる。  

```PHP
use \App\Admin\IndexController as AdminIndexController;
use \App\Products\IndexController as ProductsIndexController;

new AdminIndexController;
new ProductsIndexController;
```

useによって省略したクラス名の完全修飾名を文字列として必要とする場合、::classで取得することができる。

```PHP
// このファイル内でIndexControllerといえば\App\Admin\IndexControllerのことである、という宣言。
use \App\Admin\IndexController;

var_dump(IndexController::class === '\App\Admin\IndexController'); // true
```

### 先頭にバックスラッシュをつける理由

`use \Knp\ImageHandler\Convert`の先頭につける'\\'は、名前空間のルートを意味する。ファイルパスの'/'と似たような役割。  
仮に'\\'をつけなかった場合、名前空間は相対的に解決されてしまう。

```PHP
namespace \App\Admin;

use Knp\Controller\Controller;

class IndexController extends Controller {...}
```

上のように記述した場合、`use Knp\Controller\Controller`は`\App\Admin\Knp\Controller\Controller`を意味する。

## 記述順

namespaceとuse、その他の記述順の例

```PHP
<?php

declare(strict_types=1);

namespace \App\Admin;

use \Knp\Controller\Controller;

class IndexController extends Controller {...}
```

## 名前空間の命名規則

アプリケーション内の名前空間のトップレベルは、慣例的に`App`とすることが多い。  

パッケージ化してパッケージ化して使い回すライブラリなどは、  
`Knp\ImageHandler`のように`ベンダー名\ライブラリ名`という形式の名前空間を与える。  

## クラス以外の名前空間

ここまではクラスと名前空間について説明したが、名前空間は関数や定数に対しても機能する。

以下の例の場合、time関数の呼び出しは、まず\App\Admin\timeという関数が定義されているかが走査され、  
見つからなければグローバルのtimeを実行するという順序になるため、処理の流れとして無駄がある（大抵の場合、有意な差が生じるほどではない）。

```PHP
namespace \App\Admin;

echo time();
```

先頭に'\\'を付ければ、グローバルに定義された関数を直接呼び出せるので、処理速度の低下を防ぐことができる。  

```PHP
namespace \App\Admin;

echo \time();
```

### 定数と名前空間

defineによる定数定義はnamespaceの影響を受けない。  

```PHP
namespace App;

define('BASE_URL', 'http://localhost/test_project');

echo BASE_URL; // 出力される
echo \App\BASE_URL; // エラー
```

ただし、完全修飾名で定義することで、名前空間を利用することはできる。

```PHP
namespace App;

use \App\BASE_URL;

define('\App\BASE_URL', 'http://localhost/test_project');

echo BASE_URL; // 出力される
echo \App\BASE_URL; // 出力される
```

一方、constによる定数定義はnamespaceの影響を受ける。

```PHP
namespace App;

use \App\BASE_URL;

const BASE_URL = 'http://localhost/test_project';

echo BASE_URL; // 出力される
echo \App\BASE_URL; // 出力される
```

## PSR-4とオートロード

PSR-4は名前空間を利用したautoloaderに関する規約。  

Composerを利用する場合、設定ファイルのconmposer.jsonで名前空間のトップレベルとファイルパスを紐付ける。  
以下のように記述した場合、`src`ディレクトリが名前空間`App`に対するオートロードの起点になるので、  
アプリケーション内で記述するクラスファイルは、すべて`src`ディレクトリの配下に設置しなければならない。

```JSON
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

`src/Admin`に`IndexController`というクラスのファイルを設置する場合、  
以下のように、`IndexController`クラスは名前空間`App\Admin`に属する必要がある。

```PHP
<?php

declare(strict_types=1);

namespace \App\Admin;

class IndexController {...}
```

なお、Composerを利用しない場合、autoloaderを自分で定義する必要がある。  
PSR-4に準拠したautoloaderの定義の例は下記リンクを参照のこと。  
[https://github.com/TaroMori0321/class-autoload](https://github.com/TaroMori0321/class-autoload)
