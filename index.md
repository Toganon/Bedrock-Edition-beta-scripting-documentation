<!-- TOC -->

- [Scripting System](#scripting-system)
    - [サンプル](#サンプル)
    - [既知の不具合と対処法](#既知の不具合と対処法)
    - [重大な変更](#重大な変更)
    - [前提条件](#前提条件)
    - [始めよう](#始めよう)
    - [フォルダ構造](#フォルダ構造)
    - [スクリプトの構造](#スクリプトの構造)
    - [デバッグ](#デバッグ)
- [Script API オブジェクト](#script-api-オブジェクト)
    - [Entity JS API オブジェクト](#entity-js-api-オブジェクト)
    - [Level JS API オブジェクト](#level-js-api-オブジェクト)
    - [Component JS API オブジェクト](#component-js-api-オブジェクト)
    - [Query JS API オブジェクト](#query-js-api-オブジェクト)
    - [ItemStack JS API オブジェクト](#itemstack-js-api-オブジェクト)
    - [Block JS API オブジェクト](#block-js-api-オブジェクト)
    - [Ticking Area JS API オブジェクト](#ticking-area-js-api-オブジェクト)
- [Script関数](#script関数)
    - [Entity関数](#entity関数)
    - [Component関数](#component関数)
    - [Event関数](#event関数)
    - [Entityクエリ](#entityクエリ)
    - [コマンド](#コマンド)
    - [Block関数](#block関数)
- [スクリプトコンポーネント](#スクリプトコンポーネント)
    - [Levelコンポーネント](#levelコンポーネント)
    - [Serverコンポーネント](#serverコンポーネント)
    - [Clientコンポーネント](#clientコンポーネント)
    - [Blockコンポーネント](#blockコンポーネント)
- [ユーザー定義 コンポーネント](#ユーザー定義-コンポーネント)
- [スクリプトイベント](#スクリプトイベント)
    - [クライアントイベント](#クライアントイベント)
    - [サーバーイベント](#サーバーイベント)

<!-- /TOC -->

これは、Bedrock Editionベータ1.13.0.5のスクリプトドキュメントです。このリリースの機能は最終的なものではなく、最終リリース（1.13.0）の前に予告なく変更される可能性があります。アドオンが正常に機能しない場合は、必ずドキュメントを確認してください。ベータ版用に作成されたリソースパックと動作パックは、最終リリースでの動作が保証されていません。 

## Scripting System 
Minecraft スクリプトエンジンはJavaScriptを使用しています。  
JavaScriptでスクリプトを書き、ビヘイビアパックにまとめることでゲーム上の出来事を監視、反応したり、エンティティの持つデータを取得、改変したり、ゲームに影響を与えることができます。 

### サンプル 
スクリプトを書き始めるためのサンプルです。解凍してコードを見たり、.mcpackとしてインポートしてみてください。  
|サンプル|最終更新|ダウンロード|
|:--:|:--:|:--:|
|ターン制RPG|2019/4/17|https://aka.ms/minecraftscripting_turnbased|
|アリーナ|2019/4/17|https://aka.ms/minecraftscripting_mobarena|

### 既知の不具合と対処法  
|不具合|対処法|
|:--:|:--:|
|アーカイブされたパックから適切にスクリプトが読み込めない|スクリプトパックをワールドに適用する前に解凍します。拡張子が.mcpackのスクリプトパックをインポートすると、解凍されます|
|カスタムUIがVR、MRで動作しない|現在カスタムUIは通常モードでしか使用できません|
|カスタムUIが再起動、再開時に状態を保持しない|対処法無し|
|ScriptingAPIを使用してないワールドを終了した後にScriptingAPIを使用しているワールドに入ると指定したワールドとは違うものがロードされる|ScriptingAPIを使用しているワールドに入る前にMinecraftを再起動する|
|死んでいるエンティティにremoveEntity(destroyEntity？)を使用するとクラッシュする|removeEntityをエンティティのHPを0にしたと同時に使用しない。エンティティを削除するとすぐに削除されます(死亡モーションが出ない？)。死んだエンティティを保存しておいて次のフレームで削除しましょう(サンプルが参考になるでしょう)|

### 重大な変更  
実験的にScripting APIの作業と改良を続けるため、現在のスクリプトを破壊するような変更を加える必要があるかもしれません。スクリプトが期待どおりに機能しない場合は、この項を確認してください。  
|カテゴリ|変更点|
|:--:|:--:|
|UI|engine.onはファセット名を取ります。スクリプトエンジンの起動を確認してUIに接続するには、次が必要です。<br>`engine.on("facet:updated:core.scripting", ...);`<br>初期化の完了後、次を実行します。<br>`engine.trigger("facet:request", ["core.scripting"]);`|
|コンポーネント|getComponentを呼び出すと、'data'内のコンポーネントのパラメーターが返されるようになりました。|
|イベント|イベントデータオブジェクトは、イベントデータオブジェクトの 'data'内にイベントのパラメーターを保持するようになりました。|
|イベント|カスタムイベントは、使用する前に登録する必要があります。詳細については、registerEventDataのドキュメントを参照してください。|
|イベント|カスタムイベントには名前空間が必要です。"minecraft"名前空間は使用できません、この名前空間は組み込みイベント専用です。|
|イベント|イベントデータが標準化されました。 <br>createEventDataを呼び出して、イベントのすべてのフィールドとデフォルト値が事前に入力されたイベントデータオブジェクトを作成します。<br>イベントをトリガーするには、このイベントデータオブジェクトが必要です。|

### 前提条件  
独自のスクリプトを作成するために必要なソフトウェアのリストです。  
注意：現在、スクリプトはWindows 10 PCでのみサポートされています。スクリプトをサポートしていないデバイスでスクリプトを使用してワールドを開こうとすると、ワールドに入ることができないことを知らせるエラーメッセージが表示されます。  

|ソフトウェア|要求|推奨|
|:--:|:--:|:--:|
|テキストエディタ|Visual Studio Codeかその他のテキストエディタ|「JavaScript diagnostics」、「JavaScript and TypeScript language support」、「Just-In-Time debugger」がインストールされたVisual Studio Community 2017|
|デバッガ|N/A|Visual Studio Community 2017|
|その他||バニラのビエイビアパック https://aka.ms/behaviorpacktemplate|
|ストレージ|1.0GB以上の空き容量|3.0GB以上の空き容量|

### 始めよう  
最初に最新のバニラのビヘイビアパックをダウンロードする https://aka.ms/behaviorpacktemplate  
ビヘイビアパックをダウンロードしたら解凍し、動かしたいスクリプトを全て入れるscriptsフォルダを探す。  
scriptsフォルダにはclientとserverフォルダがある。  
- clientフォルダ
  - 各クライアント側で動作するスクリプトを格納する。イベントに応答したり、各プレイヤーに固有の何かを管理するのによい
- serverフォルダ
  - サーバー側で動作するスクリプトを格納する。エンティティのスポーン、コンポーネントの追加、変更などが含まれる  
  
clientかserverのどちらを使うか決まったら、新しい空の.jsファイルを適切なフォルダに作り、用意したテキストエディタで開き、スクリプトを書きましょう。  
フォルダには何個でもJavaScriptファイルを置けて(名前は自由)それぞれが独立して動作します。  
スクリプトを書き終わってテストしたくなったら、フォルダをzipで圧縮し、拡張子を.zipから.mcpackに書き換え、ダブルクリックでインポートできます。  
新しくパックを有効にしたワールドを作りテストしましょう。https://www.minecraft.net/en-us/addons/  

注意  
スクリプトを動作させたいワールドの試験的ゲームプレイの項目を有効にする必要があります。これはScriptingAPIがβ版である間ずっと必要です。  
クライアントスクリプトが有効なワールドに入ると、デバイスでスクリプトを実行することを許可するように求められます。(これは、ローカルプレイとマルチプレイの両方で表示されます)  
さらに、パックにクライアントスクリプトが含まれる場合は、パックのmanifest.jsonにclient_dataモジュールを含める必要があります。これはscriptsフォルダかclientフォルダのいずれかをクライアントに送信する必要があることをゲームに伝えます。パックのmanifest.jsonの内容の詳細については、アドオンのドキュメントのページを参照してください。  

### フォルダ構造  
- クライアントスクリプトに必要なmanifestの例  
  ```json
    {
        "description": "Example client scripts module",
        "type": "client_data",
        "uuid": "c05a992e-482a-455f-898c-58bbb4975e47",
        "version": [0, 0, 1]
    }
    ```
- パック構造  
    ```
    ├── scripts  
    │   ├── client  
    │   │   └── myClientScript.js  
    │   └── server  
    │       └── myServerScript.js  
    ├── manifest.json  
    └── pack_icon.png  
    ```  

### スクリプトの構造
この項ではMinecraft Scripting Engineのスクリプトファイルの基本構造の簡単な説明をします。もしJavaScriptについてもっと知りたくなったり、基礎的なチュートリアルを受けたくなったら、公式ドキュメントをチェックしてください https://developer.mozilla.org/docs/Web/JavaScript  
ここに書かれているものはある意味必須なものですが、すべてではありません。必要に応じてさらにメソッドを作ることが可能です  

#### 1.システム登録  
まず、ファイル用にシステムを登録する必要があります。これにより、クライアントまたはサーバーでスクリプトを実行する準備がされます。一般的に、スクリプトファイルをclientフォルダーとserverフォルダーのどちらに配置したかで選択します。やることは簡単でclientかserverのどちらかのregisterSystemを呼び出し、必要なAPIのバージョンを指定するだけです  

|型|名前|説明|
|:--:|:--:|:--:|
|整数|majorVersion|Minecraft Script Engineのメジャーバージョン|
|整数|minorVersion|Minecraft Script Engineのマイナーバージョン|

##### サンプルコード
- Client System  
```js
let sampleClientSystem = client.registerSystem(0, 0);
```  
- Server System  
```js
let sampleServerSystem = server.registerSystem(0, 0);
```  

#### 2.システム初期化  
これは、システムが登録された直後に呼び出される最初のメソッドです。ワールドを開いてスクリプトが読み込まれるとすぐに実行されます。  
カスタムコンポーネントやイベントの登録、イベントリスナの登録など、スクリプトのセットアップができます。これは<strong>プレイヤーがスポーンする前に</strong>実行されます。この関数は、変数を初期化し、イベントリスナを設定するために使用します。この時点でエンティティをスポーンさせたり操作しないでください。また、プレイヤーの準備が整う前に実行されるため、UIの操作やチャットの送信も避ける必要があります。  

##### サンプルコード  
```js  
sampleSystem.initialize = function() {
	////イベント、コンポーネント、クエリの登録など  
};  
```  

#### 3.システム更新  
このメソッドはゲームチック毎に(1/20秒に1回)呼び出されます。コンポーネントの変更を取得、確認、対応するのに適しています。  

##### サンプルコード  
```js  
sampleSystem.update = function() {
     //全てのアップデート
};
```  

#### 4.システム終了  
このメソッドは、スクリプトエンジンが終了するときに呼び出されます。クライアントなら、プレイヤーがワールドを出るとき、サーバーの場合、最後のプレイヤーがワールドを出た時に呼び出されます。  

##### サンプルコード  
```js  
sampleSystem.shutdown = function() {
    //スクリプトのクリーンアップ
};
```

### デバッグ  
スクリプトが機能しなかったり、希望どおりに動作していなっかたりしますか？心配無用です！スクリプトエンジンにデバッグ機能を組み込み、スクリプトで何が起こっているのかを把握できるようにしました。  
プログラミングが初めての場合、またはデバッグについて詳しく知りたい場合は、Visual Studioでのデバッグに関するドキュメントを参照してください。 https://docs.microsoft.com/visualstudio/debugger  
スクリプトで何か問題が発生したときに何が起こったかを知るには、ゲーム内で知る方法とリアルタイムで監視する方法の2つの方法があります。これについては以下で説明します。ゲーム内でのデバッグ方法にはゲームとスクリプトだけで充分ですが、リアルタイムでのデバッグ方法はwindow10のPCとVisual Stadioが必要です。  

#### ゲーム内  
ゲームでスクリプトを実行すると、何か問題が発生した場合はスクリプトエンジンがエラーメッセージを出力します。たとえば、スクリプトエンジンが認識していないコンポーネントを取得しようとすると、エラーが発生します。  
これらのメッセージを確認するには、生成されたすべてのエラーメッセージが表示されるチャット画面を開くか、ゲームが生成したログファイルを開きます。  
ログファイルの場所はプラットフォームによって異なります。windows10のPCならログファイルは以下です。
```
%APPDATA%\..\Local\Packages\Microsoft.MinecraftUWP_8wekyb3d8bbwe\LocalState\logs
```
デバッグメッセージをスクリプトに組み込んで作業することを強くお勧めします。これは、何かが正しく機能していないかを判断するのに役立ちます。追加のヘルプが必要な場合は、公式のDiscordチャンネルに連絡してください。 https://discord.gg/Minecraft  

#### リアルタイム  
Visual StudioがインストールされたWindows 10 PCを使用している場合、Visual Studioデバッガーを接続して、リアルタイムでスクリプトをデバッグできます。  このドキュメントの「推奨」の項に記載されている構成を使用してVisual Studioをインストールした場合、Just-In-Time debuggerをインストールして有効にします。このツールは、スクリプトで例外が発生するたびにVisual Studioからメッセージをポップアップし、例外が発生した場所をVisual Studioを開くことができます。  
さらに、スクリプトエンジンに手動で接続し、コードをデバッグできます。リモートデバッグを使用して、別の端末で実行されているMinecraftに接続およびデバッグできます。リモートデバッグの使用方法については、上記のVisual Studioデバッガーのドキュメントを参照してください。  
まず、Visual Studioを起動する必要があります。インストール後に初めてVisual Studioを起動する場合は、JavaScript開発用に環境を設定し、プロンプトが表示されたらMicrosoftアカウントにログインすることをお勧めします。これにより、必要な最も重要なツールを備えたVisual Studioインターフェイスがセットアップされます。  
Visual Studioを開いたら、Minecraftを起動する必要があります。次に、試験的ゲームプレイを有効にして新しい世界を作成し、スクリプトを含むビヘイビアパックを適用します。  
ワールドを作成したら、Visual Studioに戻り、「デバッグ」メニューをクリックします。次に、「プロセスにアタッチ」をクリックします。開いたウィンドウに、「プロセスをフィルター」というタイトルの検索ボックスがあります。それをクリックして、Minecraftと入力します。  
リストが実行中のMinecraftのインスタンスのみに絞り込まれたら、種類列を見て、スクリプトエンジンが実行されていることを確認できます。これはスクリプト,(x86またはx64)と表示されているものです。  
プロセスを選択し、「アタッチ」をクリックして、デバッガーをゲームにアタッチします。これで、スクリプトコードの次の行が実行されたときに「一時停止」ボタンを押してスクリプトを一時停止できるようになります。これにより、スクリプト内の変数の値を検査し、コードでエラーが発生した場合にVisual Studioを開くことができます。  
警告：ブレークポイントにヒットしてデバッガーでコードをステップ実行すると、クライアントがタイムアウトして切断されたり、サーバーからすべてのプレーヤーが切断されたりする可能性があります。  

## Script API オブジェクト  
Script APIによって渡されるオブジェクトの定義です。  

### Entity JS API オブジェクト  
|型|プロパティ名|Entity JS API Object|説明|
|:--:|:--:|:--:|:--:|
|文字列|\_\_type__||読み取り専用: オブジェクトのタイプを定義しています。"entity"か"item_entity"のいずれかになります|
|文字列|\_\_identifier__||読み取り専用: namespace:nameという形式のオブジェクトの識別子です。たとえば、タイプが"entity"でオブジェクトがバニラ牛である場合、"minecraft:cow"となります|
|正の整数|id||読み取り専用: エンティティのユニークIDです|

### Level JS API オブジェクト  
|型|プロパティ名|Level JS API Object|説明|
|:--:|:--:|:--:|:--:|
|文字列|\_\_type__||読み取り専用: オブジェクトのタイプを定義しています。"level"になります|
|正の整数|id||読み取り専用: レベルのユニークIDです|

### Component JS API オブジェクト  
|型|プロパティ名|Component JS API Object|説明|
|:--:|:--:|:--:|:--:|
|文字列|\_\_type__||読み取り専用: オブジェクトのタイプを定義しています。"component"になります|
|JavaScript Object|data||コンポーネントの内容です|

### Query JS API オブジェクト  
|型|プロパティ名|Query JS API Object|説明|
|:--:|:--:|:--:|:--:|
|文字列|\_\_type__||読み取り専用: オブジェクトのタイプを定義しています。"query"になります|
|正の整数|query_id||読み取り専用: クエリのユニークIDです|

### ItemStack JS API オブジェクト  
|型|プロパティ名|ItemStack JS API Object|説明|
|:--:|:--:|:--:|:--:|
|文字列|\_\_type__||読み取り専用: オブジェクトのタイプを定義しています。"item_stack"になります|
|文字列|\_\_identifier__||読み取り専用: namespace:nameという形式のオブジェクトの識別子です。たとえば、オブジェクトが棒である場合、"minecraft:stick"となります|
|文字列|item||読み取り専用: アイテムの識別子です|
|文字列|count||読み取り専用: アイテムの数です|

### Block JS API オブジェクト  
|型|プロパティ名|Block JS API Object|説明|
|:--:|:--:|:--:|:--:|
|文字列|\_\_type__||読み取り専用: オブジェクトのタイプを定義しています。"block"になります|
|文字列|\_\_identifier__||読み取り専用: namespace:nameという形式のオブジェクトの識別子です。たとえば、オブジェクトが岩盤である場合、"minecraft:bedrock"となります|
|JavaScript Object|ticking_area||読み取り専用: ブロック取得に使うticking areaオブジェクトです|
|JavaScript Object|block_position||読み取り専用: ブロックの場所で、ブロックのユニークIDとしても動作します|  

#### block_position  
|型|プロパティ名|説明|
|:--:|:--:|:--:|
|整数|x|X座標|
|整数|y|Y座標|
|整数|z|Z座標|

### Ticking Area JS API オブジェクト  
EntityとLevelの二つのTicking Areaオブジェクトがり、Ticking Area関数を呼ぶときをどちらかを引数としてとります。

#### Entity Ticking Area JS API オブジェクト  
|型|プロパティ名|Entity Ticking Area JS API Object|説明|
|:--:|:--:|:--:|:--:|
|文字列|\_\_type__||読み取り専用: オブジェクトのタイプを定義しています。"entity_ticking_area"になります|
|正の整数|entity_ticking_area_id||読み取り専用: Ticking AreaのユニークIDです|

#### Level Ticking Area JS API オブジェクト  
|型|プロパティ名|Level Ticking Area JS API Object|説明|
|:--:|:--:|:--:|:--:|
|文字列|\_\_type__||読み取り専用: オブジェクトのタイプを定義しています。"level_ticking_area"になります|
|正の整数|level_ticking_area_id||読み取り専用: Ticking AreaのユニークIDです|

## Script関数  
ゲーム内の物事を変更するためのスクリプトエンジンの関数です  

### Entity関数  

#### createEntity()  
コンポーネントのない空のエンティティを作成し、ワールドに配置しません。空のエンティティはカスタムタイプで、識別子が空白になります。これは、世界に存在する有効なエンティティではなく、スクリプト上だけの空のエンティティです。  
注意：エンティティはサーバー上で最初に作成され、クライアントはその後新しいエンティティを通知されます。生成されたオブジェクトをすぐにクライアントに送信すると、作成されたエンティティがまだクライアントに存在しない可能性があります。  

##### 戻り値  
|型|値|説明|
|:--:|:--:|:--:|
|Entity JS API Object|object|新しく作成されたエンティティのオブジェクト|
|JavaScript Object|null|エンティティの作成時に問題が発生した場合返されます|

#### createEntity(Type, TemplateIdentifier)
エンティティを作成し、指定されたテンプレートをJSONで定義されているとおりに適用します。これにより、スクリプトで作成されたエンティティのベースとして、適用されたビヘイビアパックからエンティティをすばやく作成できます。エンティティは、指定された識別子のJSONファイルで定義されているすべてのコンポーネント、コンポーネントグループ、およびイベントトリガーとともに世界に生成されます。サーバースクリプトでのみ機能します。  
注意：エンティティはサーバー上で最初に作成され、クライアントはその後新しいエンティティを通知されます。生成されたオブジェクトをすぐにクライアントに送信すると、作成されたエンティティがまだクライアントに存在しない可能性があります。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|文字列|Type|テンプレートによって作成されているエンティティのタイプを指定。”entity”か”item_entity”が有効です|
|文字列|TemplateIdentifier|適用されたビヘイビアパックのエンティティ識別子のいずれかです。<br>たとえば、ここでminecraft：cowを指定すると、生成されたエンティティはJSONで定義された牛になります|

##### 戻り値  
|型|値|説明|
|:--:|:--:|:--:|
|Entity JS API Object|object|新しく作成されたエンティティのオブジェクト|
|JavaScript Object|null|エンティティの作成時に問題が発生した場合返されます|

#### destroyEntity(EntityObject)  
EntityObjectによって識別されるエンティティを破棄します。エンティティがワールドに存在する場合、ワールドからエンティティを削除し、破棄します。また、これによりEntityObjectが無効になります。エンティティを使用し終わった後、もう参照する必要のない場合のみ使用してください。これはエンティティを殺さず、死亡イベントも発生せず削除されます。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|EntityObject|createEntity()の戻り値から取得されたオブジェクトまたはエンティティイベントから取得されたオブジェクト|

##### 戻り値  
|型|値|説明|
|:--:|:--:|:--:|
|真偽値|true|エンティティの破棄に成功した場合返します|
|JavaScript Object|null|エンティティの破棄に問題が発生した場合返します|

#### isValidEntity(EntityObject)  
指定されたEntityObjectが有効なエンティティかどうかを確認します。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|EntityObject|createEntity()の戻り値から取得されたオブジェクトまたはエンティティイベントから取得されたオブジェクト|

##### 戻り値  
|型|値|説明|
|:--:|:--:|:--:|
|真偽値|true|エンティティがスクリプトエンジンのデータベースに存在する場合返します|
|真偽値|false|エンティティがスクリプトエンジンのデータベースに存在しない場合返します|
|JavaScript Object|null|エンティティの確認時に問題が発生した場合返します|

### Component関数  

#### registerComponent(ComponentIdentifier, ComponentData)
スクリプトにのみ存在するカスタムコンポーネントを作成します。その後、エンティティに追加、削除、更新できます。これらのカスタムコンポーネントは、スクリプトエンジン実行中にのみ存在します。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|文字列|ComponentIdentifier|カスタムコンポーネントの識別子。名前を組み込みコンポーネントと重複させずにで一意に参照できるように、名前空間を使用する必要があります。<br>例 “myPack:myCustomComponent”|
|JavaScript Object|ComponentData|フィールドの名前と各フィールドがコンポーネント内に保持するデータを定義したJavaScriptオブジェクト|

##### 戻り値  
|型|値|説明|
|:--:|:--:|:--:|
|真偽値|true|コンポーネントの登録に成功した場合返します|
|JavaScript Object|null|コンポーネントの登録に問題が発生した場合返します|

#### createComponent(EntityObject, ComponentIdentifier)
指定されたコンポーネントを作成し、エンティティに追加します。これは、最初に登録する必要があるカスタムコンポーネントでのみ使用してください。エンティティが既にコンポーネントを持っている場合、代わりに既に持っているコンポーネントを取得します。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|EntityObject|createEntity()の戻り値から取得されたオブジェクト、またはエンティティイベントから取得されたオブジェクト|
|文字列|ComponentIdentifier|エンティティに追加するコンポーネントの識別子。これは、組み込みコンポーネントの識別子(スクリプトコンポーネントの項を参照)、またはregisterComponent()の呼び出しで作成されたカスタムコンポーネントのいずれか|

##### 戻り値  
|型|値|説明|
|:--:|:--:|:--:|
|Component JS API Object|object|以下のフィールドおよびコンポーネントで定義されているすべてのフィールドを持つオブジェクト|
|JavaScript Object|null|コンポーネント生成に異常が発生した時に返します|

#### hasComponent(EntityObject, ComponentIdentifier)
指定されたエンティティが指定されたコンポーネントを持つかどうかを確認します  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|EntityObject|createEntity()の戻り値から取得されたオブジェクト、またはエンティティイベントから取得されたオブジェクト|
|文字列|ComponentIdentifier|確認するコンポーネントの識別子。これは、組み込みコンポーネントの識別子(スクリプトコンポーネントの項を参照)、またはregisterComponent()の呼び出しで作成されたカスタムコンポーネントのいずれか|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|真偽値|true|エンティティが指定コンポーネントを持つ場合返します|
|真偽値|false|エンティティが指定コンポーネントを持たない場合返します|
|JavaScript Object|null|エンティティの確認時に問題が発生したか、未定義のコンポーネントを渡された場合返します|

#### getComponent(EntityObject, ComponentIdentifier)
エンティティから指定されたコンポーネントを探します。存在する場合、コンポーネントからデータを取得して返します  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|EntityObject|createEntity()の戻り値から取得されたオブジェクト、またはエンティティイベントから取得されたオブジェクト|
|文字列|ComponentIdentifier|エンティティから取得するコンポーネントの識別子。これは、組み込みコンポーネントの識別子(スクリプトコンポーネントの項を参照)、またはregisterComponent()の呼び出しで作成されたカスタムコンポーネントのいずれか|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|Component JS API Object|object|以下のフィールドおよびコンポーネントで定義されているすべてのフィールドを持つオブジェクト|
|JavaScript Object|null|コンポーネントをエンティティが持たない場合かコンポーネント取得時に問題が発生した場合返します|

##### Component JS API Object
|型|プロパティ名|Component JS API Object|説明|
|:--:|:--:|:--:|:--:|
|文字列|\_\_type__||読み取り専用: オブジェクトのタイプを定義しています。"component"になります|
|JavaScript Object|data||コンポーネントの内容です|

#### applyComponentChanges(EntityObject, ComponentObject)
スクリプトで行ったコンポーネントの変更をエンティティに適用します。これは各コンポーネントごとに意味することはわずかに異なる可能性があります。つまり、新しいデータでエンティティのコンポーネントをリロードします。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|EntityObject|コンポーネントの変更を適応したいエンティティのオブジェクト|
|Component JS API Object|ComponentObject|createComponent()またはgetComponent()によってエンティティから取得したコンポーネントオブジェクト|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|真偽値|true|コンポーネントの更新が成功した場合返します|
|JavaScript Object|null|コンポーネント更新に異常が発生した時に返します|

#### destroyComponent(EntityObject, ComponentIdentifier)
指定したエンティティから指定したコンポーネントを削除します。現在、これはカスタムコンポーネントでのみ機能し、JSONでエンティティに定義されたコンポーネントの削除には使用できません。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|EntityObject|コンポーネントを削除したいエンティティのオブジェクト|
|文字列|ComponentIdentifier|エンティティから削除するコンポーネントの識別子。これは、組み込みコンポーネントの識別子(スクリプトコンポーネントの項を参照)、またはregisterComponent()の呼び出しで作成されたカスタムコンポーネントのいずれか|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|真偽値|true|コンポーネントの削除が成功した場合返します|
|JavaScript Object|null|コンポーネント削除に異常が発生した時に返します|

### Event関数
イベントの処理に使用される関数です。反応できるイベントのリストについては、このドキュメントの「イベント」の項を確認してください  

#### registerEventData(EventIdentifier, EventData)
イベントをスクリプトエンジンに登録します。これにより、createEventDataを呼び出してイベントデータを作成し、デフォルトのデータとフィールドで初期化することができます。カスタムイベントのみ登録できます。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|文字列|EventIdentifier|登録するカスタムイベントの識別子。名前空間は必須であり、"minecraft"に設定することはできない。|
|JavaScript Object|EventData|イベントのフィールドとデフォルト値を持つJavaScriptオブジェクト|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|真偽値|true|イベントの登録が成功した場合返します|
|JavaScript Object|null|イベントの登録に異常が発生した時に返します|

#### createEventData(EventIdentifier)
指定したイベントのすべての必須フィールドとデフォルトデータを含むオブジェクトを作成します。イベントがカスタムイベントの場合、事前に登録されている必要があります。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|文字列|EventIdentifier|作成するカスタムイベントの識別子|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|JavaScript Object||イベントデータを格納したオブジェクト|
|JavaScript Object|null|イベントの作成に異常が発生した時に返します|

#### listenForEvent(EventIdentifier, CallbackObject)
指定したイベントがブロードキャストされるたびに呼び出されるJavaScriptオブジェクトを登録できます。イベントは、組み込みイベントまたはスクリプトで指定されたイベントのいずれかです。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|文字列|EventIdentifier|イベントリスナを登録するイベントの識別子。組み込みイベントまたはスクリプトのカスタムイベントの識別子|
|JavaScript Object|CallbackObject|イベントがブロードキャストされるたびに呼び出されるJavaScriptオブジェクト|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|真偽値|true|イベントリスナの登録に成功した場合返します|
|JavaScript Object|null|イベントリスナの登録に異常が発生した時に返します|

#### broadcastEvent(EventIdentifier, EventData)
スクリプトから目的のデータを使用してイベントを発生させます。イベントリスナを登録したスクリプトすべてに通知され、指定されたデータがそれらに渡されます。  
##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|文字列|EventIdentifier|発生させるイベントの識別子。組み込みイベントまたはスクリプトのカスタムイベントの識別子|
|JavaScript Object|EventData|イベントのデータ。リスナに渡したいパラメータを持ったJavaScriptオブジェクトを作成でき、エンジンがリスナにデータを配信します|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|真偽値|true|イベントのブロードキャストに成功した場合返します|
|JavaScript Object|null|イベントのブロードキャストに異常が発生した時に返します|

### Entityクエリ
エンティティクエリは、コンポーネントに基づいてエンティティをフィルタリングする方法です。クエリを登録したら、それに対応したすべてのエンティティを要求できます。エンティティクエリは、アクティブなエンティティのみを返します。現在ロードされていないチャンクのエンティティはクエリに含まれません。  

#### registerQuery()
クエリを登録できます。クエリには、フィルター要件を満たすすべてのエンティティが含まれます。デフォルトでフィルタは追加されないため、すべてのエンティティが対応します。

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|Query JS API Object|object|クエリのIDを格納したオブジェクト|
|JavaScript Object|null|クエリの登録に異常が発生した時に返します|

##### サンプルコード
```js
var simple_query = {};
system.initialize = function() {
	simple_query = this.registerQuery(); 
};
```

#### registerQuery(Component, ComponentField1, ComponentField2, ComponentField3)
指定されたコンポーネントを持つエンティティのみを表示するクエリを登録し、クエリからエンティティを取得するときにそのコンポーネントのどのフィールドをフィルターとして使用するかを定義できます。コンポーネント識別子のみ、またはコンポーネント識別子とテストするコンポーネントのプロパティの名前を3つまで指定できます（プロパティ名を指定する場合は、3つ指定する必要があります）。  

##### 引数  
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|Component||エンティティのフィルタリングに使用されるコンポーネントの識別子|
|文字列|ComponentField1|"x"|エンティティをフィルタリングするコンポーネントの1つ目のフィールドの名前。デフォルトでは"x"|
|文字列|ComponentField2|"y"|エンティティをフィルタリングするコンポーネントの2つ目のフィールドの名前。デフォルトでは"y"|
|文字列|ComponentField3|"z"|エンティティをフィルタリングするコンポーネントの3つ目のフィールドの名前。デフォルトでは"z"|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|Query JS API Object|object|クエリのIDを格納したオブジェクト|
|JavaScript Object|null|クエリの登録に異常が発生した時に返します|

##### サンプルコード
クエリの登録  
```js
var spacial_query = {};
system.initialize = function() {
	spacial_query = this.registerQuery("minecraft:position", "x", "y", "z"); 
};
```

#### addFilterToQuery(Query, ComponentIdentifier)
デフォルトでは、フィルターは追加されません。これにより、クエリですべてのエンティティを取得できます。

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Query JS API Object|Query|フィルターを適用したいクエリのIDを格納したオブジェクト|
|文字列|ComponentIdentifier|フィルターリストに追加されるコンポーネントの識別子。そのコンポーネントを持つエンティティのみがクエリにリストされる|

##### サンプルコード
クエリでのフィルタリング  
```js
var filtered_query = {};
system.initialize = function() {
	filtered_query = this.registerQuery(); 
	this.addFilterToQuery(filtered_query, "minecraft:explode"); 
};
```

#### getEntitiesFromQuery(Query)
クエリに対応したエンティティを取得できます。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Query JS API Object|Query|事前にregisterQuery()を使用して登録したクエリ|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|Array|array|クエリ内で見つかったエンティティを表すEntityObjectの配列|
|JavaScript Object|null|エンティティの取得時に問題が発生した場合返します|

##### サンプルコード
エンティティの取得  
```js
system.update = function() {
	let all_entities = this.getEntitiesFromQuery(simple_query);
	let exploding_creepers = this.getEntitiesFromQuery(filtered_query);
};
```

#### getEntitiesFromQuery(Query, ComponentField1_Min, ComponentField2_Min, ComponentField3_Min, ComponentField1_Max, ComponentField2_Max, ComponentField3_Max)
組み込みのコンポーネントフィルターで作成されたクエリに対応したエンティティを取得できます。返されるエンティティは、クエリが登録されたときに定義されたコンポーネントを持ち、getEntitiesFromQueryの呼び出しで指定されたクエリで定義されたコンポーネントの3つのフィールドの値が指定範囲内のエンティティのみです。  

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|Query JS API Object|Query|事前にregisterQuery(...)を使用して登録したクエリ|
|小数|ComponentField1_Min|クエリに含みたいエンティティの1つめのコンポーネントフィールドの最小値|
|小数|ComponentField2_Min|クエリに含みたいエンティティの2つめのコンポーネントフィールドの最小値|
|小数|ComponentField3_Min|クエリに含みたいエンティティの3つめのコンポーネントフィールドの最小値|
|小数|ComponentField1_Max|クエリに含みたいエンティティの1つめのコンポーネントフィールドの最大値|
|小数|ComponentField2_Max|クエリに含みたいエンティティの2つめのコンポーネントフィールドの最大値|
|小数|ComponentField3_Max|クエリに含みたいエンティティの3つめのコンポーネントフィールドの最大値|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|Array|array|クエリ内で見つかったエンティティを表すEntityObjectの配列|
|JavaScript Object|null|エンティティの取得時に問題が発生した場合返します|

##### サンプルコード
エンティティの取得  
```js
system.update = function() {
	let player_pos_component = this.getComponent(player, "minecraft:position");
	let pos = player_pos_component.data;
	let entities_near_player = this.getEntitiesFromQuery(spacial_query, pos.x - 10, pos.x + 10, pos.y - 10, pos.y + 10, pos.z - 10, pos.z + 10);
};
```

### コマンド
スクリプトからコマンドシステムを使用できます。現在、イベント（ "minecraft：execute_command"）をトリガーするか、executeCommandを使用できます。スクリプト内でのコマンド実行は現在サーバースクリプトのみで可能であり、クライアントスクリプトからは実行できません。  

#### executeCommand(Command, Callback)
サーバーでコマンドを実行できます。コマンドは、現在のフレームの最後にクエリされ、実行されます。コマンドからのすべてのデータ出力はJavaScriptオブジェクトにコンパイルされ、2番目のパラメーターで指定されたCallbackオブジェクトに送信されます。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|文字列|Command|実行するコマンド|
|JSON Object|Callback|コマンドの実行後に呼び出されるJavaScriptオブジェクト|

##### 例
```js
system.executeCommand("/fill ~ ~ ~ ~100 ~5 ~50 stone", (commandResultData) => this.commandCallback(commandResultData));

system.commandCallback = function (commandResultData) {
	let eventData = this.createEventData("minecraft:display_chat_event");
	if (eventData) {
		eventData.data.message = message;
		this.broadcastEvent("minecraft:display_chat_event", "Callback called! Command: " + commandResultData.command + " Data: " + JSON.stringify(commandResultData.data, null, "    ") );
	}
};
```

### Block関数
ブロックを操作するための関数  

#### getBlock(Ticking Area, x, y, z)
x,y,z座標を指定すると、ワールドからブロックを取得できます。ブロックはロードされているチャンク内に存在しなければいけません。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Ticking Area JS API Object|Ticking Area|ブロックの存在するTicking Area|
|整数|x|取得したいブロックのx座標|
|整数|y|取得したいブロックのy座標|
|整数|z|取得したいブロックのz座標|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|Block JS API Object|object|ブロックを格納したオブジェクト|
|JavaScript Object|null|ブロックの取得時に問題が発生した場合返します|

#### getBlock(Ticking Area, PositionObject)
Positionを指定すると、ワールドからブロックを取得できます。ブロックはロードされているチャンク内に存在しなければいけません。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Ticking Area JS API Object|Ticking Area|ブロックの存在するTicking Area|
|JavaScript Object|PositionObject|ブロックの座標を示すオブジェクト|

##### PositionObject
|型|プロパティ名|説明|
|:--:|:--:|:--:|
|整数|x|X座標|
|整数|y|Y座標|
|整数|z|Z座標|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|Block JS API Object|object|ブロックを格納したオブジェクト|
|JavaScript Object|null|ブロックの取得時に問題が発生した場合返します|

#### getBlocks(Ticking Area, x min, y min, z min, x max, y max, z max)
x,y,z座標の最小値と最大値を指定すると、ワールドからブロックを取得できます。ブロックはロードされているチャンク内に存在しなければいけません。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Ticking Area JS API Object|Ticking Area|ブロックの存在するTicking Area|
|整数|x min|取得したい範囲のx座標の最小値|
|整数|y min|取得したい範囲のy座標の最小値|
|整数|z min|取得したい範囲のz座標の最小値|
|整数|x max|取得したい範囲のx座標の最大値|
|整数|y max|取得したい範囲のy座標の最大値|
|整数|z max|取得したい範囲のz座標の最大値|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|Array|array|ブロックオブジェクトの3次元配列。インデックスは、指定された最小値に対するブロックの位置。|
|JavaScript Object|null|ブロックの取得時に問題が発生した場合返します|

#### getBlocks(Ticking Area, Minimum PositionObject, Maximum PositionObject)
最小と最大のPositionオブジェクトを指定すると、ワールドからブロックを取得できます。ブロックはロードされているチャンク内に存在しなければいけません。  

##### 引数  
|型|名前|説明|
|:--:|:--:|:--:|
|Ticking Area JS API Object|Ticking Area|ブロックの存在するTicking Area|
|JavaScript Object|Minimum PositionObject|最小のPositionオブジェクト|
|JavaScript Object|Maximum PositionObject|最大のPositionオブジェクト|

##### PositionObject
|型|プロパティ名|説明|
|:--:|:--:|:--:|
|整数|x|X座標|
|整数|y|Y座標|
|整数|z|Z座標|

##### 戻り値
|型|値|説明|
|:--:|:--:|:--:|
|Array|array|ブロックオブジェクトの3次元配列。インデックスは、指定された最小値に対するブロックの位置。|
|JavaScript Object|null|ブロックの取得時に問題が発生した場合返します|

## スクリプトコンポーネント
Minecraft Script Engineで利用可能な属性、プロパティ、およびコンポーネントのドキュメントです。  
コンポーネントには、サーバーコンポーネントとクライアントコンポーネントの2種類があります。以下のそれぞれの項でそれらが何であるかについてもう少し詳しく説明します。  
コンポーネントは、エンティティに対して追加、取得、更新、および削除できます。それらは単独では存在しません。現在、ユーザー定義のコンポーネントのみをエンティティに追加および削除できます。取得または更新することができるコンポーネントはエンティティ内に存在するもののみです。  
関数の項を確認して、コンポーネントを追加、削除、取得、および更新する方法を確認してください。このセクションでは、各コンポーネントの特定のAPIを扱います。  

### Levelコンポーネント
Levelコンポーネントです。Levelオブジェクト内にのみ存在することができ、Levelオブジェクトから削除することはできません。グローバルサーバーオブジェクトを介してコンポーネントを取得し、データを変更できます。  

```js
let levelComponent = this.getComponent(server.level, "minecraft:example_level_component");
```

#### minecraft:ticking_areas
このコンポーネントは、Levelの静的なTicking Areaへのアクセスを提供します。コンポーネントには、Ticking Areaの配列が含まれています。Ticking Areaは、名前またはUUIDでアクセスできます。  

#### minecraft:weather
ユーザーはレベルの天気を変更できます。雨と雷のレベルは個別に変更でき、デフォルトの天気変化は完全にオフにできます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|真偽値|do_weather_cycle|バニラの天気変化を使用するかどうかを決定するオプション|
|小数|rain_level|0〜1の値をとり、降雨の激しさを決定する|
|整数|rain_time|どれだけのtick時間雨が降っているか|
|小数|rain_level|0〜1の値をとり、落雷の激しさを決定する|
|整数|rain_time|どれだけのtick時間雷が鳴っているか|

### Serverコンポーネント
サーバー上で実行され、すべてのクライアント(プレイヤー)と同期されるコンポーネントです。  
可能な限り、各コンポーネントのAPIは対応するJSONに一致します（いくつかの違いは記載されています）  

#### minecraft:armor_container
このコンポーネントは、エンティティの鎧の内容を表します。コンポーネントには、鎧スロットに対応したItemStack JS API オブジェクトの配列が格納されています。  
注意：現在、アイテムとコンテナは読み取り専用です。スロットは頭から足の順に並んでいます。  

```js
// この例では、プレイヤーがエンティティを攻撃した後、プレイヤーの頭スロットを確認します。
system.listenForEvent("minecraft:player_attacked_entity", function(eventData) {
    // 鎧の取得
    let playerArmor = system.getComponent(eventData.data.player, "minecraft:armor_container");
    // プレイヤーのヘルメットを取得
    let playerHelmet = playerArmor.data[0];
    // 金のヘルメットを装備していた場合攻撃されたエンティティを破棄
    if (playerHelmet.item ==== "minecraft:golden_helmet") {
        system.destroyEntity(eventData.data.attacked_entity);
    }
});
```

#### minecraft:attack
このコンポーネントは、エンティティの攻撃ダメージ属性を制御します。現在の最小値と最大値を変更できます。変更が適用されると、エンティティの現在の攻撃は指定された最小値にリセットされます。指定された値に変更された最小値と最大値。バフやデバフはそのまま残ります。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|Range [a, b]|damage||近接攻撃が与えるランダムなダメージの範囲。<br>負の値を指定すると、エンティティにダメージを与える代わりに回復することができます|

##### damage
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|小数|range_min|0.0|エンティティが与えるダメージの最小値|
|小数|range_max|0.0|エンティティが与えるダメージの最大値|

#### minecraft:collision_box
エンティティの衝突判定を制御します。コンポーネントへの変更が適用されると、エンティティの衝突判定はすぐに更新され、新しいサイズが反映されます。  
警告：衝突判定のサイズを変更するとエンティティがブロック内に埋まる場合、エンティティは窒息し始める可能性があります。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|小数|width|1.0|ブロック単位の衝突判定の幅と奥行き。負の値は0と見なされます|
|小数|height|1.0|ブロック単位の衝突判定の高さ。負の値は0と見なされます|

#### minecraft:damage_sensor
ダメージの種類とエンティティがそれらにどのように反応するかを定義した配列です。現在、Minecraftのトリガーは適切にシリアル化できないため、applyComponentChanges()で既存のトリガーは完全に置き換えられます。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|List|on_damage||特定の種類のダメージを受けたときに呼び出すイベントを持つトリガーのリスト。エンティティの定義とイベントのフィルターを指定できます|
|真偽値|deals_damage|true|trueの場合、エンティティはダメージを受け、falseの場合エンティティはそのダメージを無視します|
|文字列|cause||ダメージの名前|

```js
// エンティティ(この場合はクリーパー)をプレイヤーが攻撃すると爆発を開始します。
// 注意：エンティティには、damage_sensorコンポーネントと、JSON記述で定義された関連イベントが必要です。
this.listenForEvent("minecraft:player_attacked_entity", function(eventData) {
	let damageSensorComponent = serverSystem.getComponent(eventData.attacked_entity, "minecraft:damage_sensor");
	damageSensorComponent.data[0].on_damage = { event:"minecraft:start_exploding", filters:[{test:"has_component", operator:"==", value:"minecraft:breathable"}] };
	serverSystem.applyComponentChanges(eventData.attacked_entity, damageSensorComponent);
});
```

#### minecraft:equipment
エンティティがその装備を定義するために使用する戦利品テーブルを定義します。変更が適用されると、装備が再ロールされ、エンティティに対して新しい装備のセットが選択されます。  
これよくわかんない  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|table||ビヘイビアパックのルートファイルに対するequimentテーブルへのファイルパス|
|List|slot_drop_chance||装備アイテムをドロップするチャンスがあるスロットのリスト|

#### minecraft:equippable
エンティティに装備できるアイテムと数を定義します。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|整数|slot|0|スロット番号|
|List|accepted_items||スロットに装備できるアイテムのリスト|
|文字列|item||このスロットに装備できるアイテムの識別子|
|文字列|interact_text||エンティティがこのアイテムを装備できるとき表示されるテキスト|
|文字列|on_equip||エンティティがアイテムを装備した時呼ばれるイベント|
|文字列|on_unequip||エンティティがアイテムを外した時呼ばれるイベント|

#### minecraft:explode
エンティティの爆発、爆発までのタイマー、タイマーがカウントダウンしているかどうかを制御します。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|Range [a, b]|fuseLength|[0.0, 0.0]|爆発する前の時間の範囲。負の値は爆発がすぐに起こることを意味する|
|小数|power|3.0|ブロック単位の爆発の半径と爆発が与えるダメージの量|
|小数|maxResistance|Infinite|爆発が発生すると、ブロック爆発抵抗はこの値で制限されます|
|真偽値|fuseLit|false|trueの場合、このコンポーネントがエンティティに追加されると、すぐに起爆準備に入る|
|真偽値|causesFire|false|trueの場合、爆発で火が付く|
|真偽値|breaks_blocks|true|trueの場合、爆発でブロックが破壊される|
|真偽値|fireAffectedByGriefing|false|trueの場合、爆発で火が付くかどうかがゲームルールに影響される|
|真偽値|destroyAffectedByGriefing|false|trueの場合、爆発でブロックが破壊されるかどうかがゲームルールに影響される|

#### minecraft:hand_container
このコンポーネントは、エンティティの手持ちを表します。コンポーネントには、手持ちコンテナの各スロットに対応したItemStack JS APIオブジェクトの配列が格納されています  
注意：現在、アイテムとコンテナは読み取り専用です。スロット0はメインハンドスロット、1はオフハンドです  

```js
// プレイヤーがエンティティを攻撃した後、オフハンドスロットをチェックします
system.listenForEvent("minecraft:player_attacked_entity", function(eventData) {
    // プレイヤーの手持ちコンテナを取得
    let handContainer = system.getComponent(eventData.data.player, "minecraft:hand_container");
    // オフハンドのアイテムを取得
    let offhandItem = handContainer.data[1];
    // オフハンドにトーテムを持っている場合攻撃されたエンティティを破棄する
    if (offhandItem.item ==== "minecraft:totem") {
        system.destroyEntity(eventData.data.attacked_entity);
    }
});
```

#### minecraft:healable
プレイヤーがエンティティを回復する方法を定義します。これは、エンティティの体力を制御するものではありません。その場合、Healthコンポーネントを使用する必要があります。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|Array|items||このエンティティを回復するために使用できるアイテムの配列<br>下記参照|
|真偽値|force_use|false|エンティティが体力満タンかどうかに関係なく、アイテムを使用できるかどうかを決定します|
|Minecraft Filter|filters||このトリガーの条件を定義するフィルター群|

##### items
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|item||使用できるアイテムの識別子|
|整数|heal_amount||このアイテムを与えたときにこのエンティティの回復量|
|Minecraft Filter|filters||このアイテムを使用してエンティティを回復するための条件を定義するフィルター群|

#### minecraft:health
エンティティの現在および最大の体力を定義します。コンポーネントをエンティティに適応すると、体力が変化します。 0以下になると、エンティティは死にます。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|整数|value|1|エンティティの現在の体力|
|整数|max|10|エンティティの最大体力|

#### minecraft:hotbar_container
このコンポーネントは、プレーヤーのホットバーのコンテンツを表します。コンポーネントには、ホットバーの各スロットに対応したItemStack JS APIオブジェクトの配列が格納されます。  
注意：現在、アイテムとコンテナは読み取り専用です。スロットは左から右に並べられています。  

```js
// プレイヤーがエンティティを攻撃した後、一番左のホットバーをチェックします。
system.listenForEvent("minecraft:player_attacked_entity", function(eventData) {
    // プレイヤーのホットバーを取得
    let playerHotbar = system.getComponent(eventData.data.player, "minecraft:hotbar_container");
    // 一番左のホットバーのアイテムを取得
    let firstHotbarSlot = playerHotbar.data[0];
    // ホットバーの一番左にりんごが入っていたら攻撃されたエンティティを破棄
    if (firstHotbarSlot.item ==== "minecraft:apple") {
        system.destroyEntity(eventData.data.attacked_entity);
    }
});
```

#### minecraft:interact
このコンポーネントが適用されるエンティティとプレイヤーの接触を定義します。(ハサミを羊につかうとかそんなんのはず)  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|Array|spawn_entities||接触が発生したときに生成されるエンティティ識別子の配列|
|文字列|on_interact||接触が発生したときに起動するイベント識別子|
|JSON Object|particle_on_start||接触の開始時に発生するパーティクル<br>下記参照|
|小数|cooldown|0.0|このエンティティが再び接触できるようになるまでの時間(秒)|
|真偽値|swing|false|trueの場合、プレイヤーはこのエンティティと接触するときに腕を振るアニメーションをします|
|真偽値|use_item|false|trueの場合、接触時にアイテムを使用します|
|整数|hurt_item|0|このエンティティとの接触に使用されたときに減る耐久値の量。0なら、アイテムの耐久値が減らないことを意味します|
|文字列|interact_text||プレイヤーがこのエンティティと接触できる場合に表示するテキスト|
|JSON Object|add_items||接触が成功したときにプレイヤーのインベントリに追加するアイテムを格納した表<br>下記参照|
|JSON Object|spawn_items||接触が成功したときにドロップするアイテムを格納した表<br>下記参照|
|文字列|transform_to_item||使用されたアイテムは、接触が成功するとこのアイテムに変換されます<br>形式 "itemName：auxValue"|
|Array|play_sounds||接触が発生したときに再生するサウンドの識別子の配列|

##### particle_on_start
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|particle_type||発生するパーティクルの種類|
|小数|particle_y_offset|0.0|y方向のオフセット|
|真偽値|particle_offset_towards_interactor|false|接触を実行した人の近くにパーティクルが表示されるかどうか|

##### add_items
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|table||loot tableファイルへの、ビヘイビアパックのパスからの相対パス|

##### spawn_items
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|table||loot tableファイルへの、ビヘイビアパックのパスからの相対パス|

#### minecraft:inventory
エンティティのインベントリを定義します。現在、これによりエンティティのインベントリを変更することはできません。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|container_type|none|このエンティティが持つコンテナのタイプ。 "horse", "minecart_chest", "minecart_hopper", "inventory", "container", "hopper"のいずれか|
|整数|inventory_size|5|コンテナのスロット数|
|真偽値|can_be_siphoned_from|false|trueの場合、このインベントリの内容はホッパーによって吸い出せます|
|真偽値|private|false|trueの場合、そのエンティティのみがインベントリにアクセスできます|
|真偽値|restrict_to_owner|false|trueの場合、エンティティのインベントリには、所有者または自分自身のみがアクセスできます|
|整数|additional_slots_per_strength|0|このエンティティが追加の強度ごとに獲得できるスロットの数(?)|

#### minecraft:inventory_container
このコンポーネントは、エンティティのインベントリを表します。コンポーネントには、インベントリ内の各スロットに対応したItemStack JS APIオブジェクトの配列が格納されています。  
注意：現在、アイテムとコンテナは読み取り専用です。スロット0〜8はホットバー、9〜16はプレイヤーのインベントリの一番上の列、17〜24は中央の列、25〜32は一番下の列です  

```js
// この例では、プレーヤーがエンティティを攻撃した後、プレーヤーの3番目のインベントリスロットをチェックします。
system.listenForEvent("minecraft:player_attacked_entity", function(eventData) {
    // プレイヤーのインベントリを取得
    let playerInventory = system.getComponent(eventData.data.player, "minecraft:inventory_container");
    // 3番目のスロットを取得
    let thirdItemSlot = playerInventory.data[2];
    // 3番目のスロットのアイテムがりんごなら攻撃されたエンティティを破棄
    if (thirdItemSlot.item ==== "minecraft:apple") {
        system.destroyEntity(eventData.data.attacked_entity);
    }
});
```

#### minecraft:lookat
エンティティに別のエンティティをターゲットさせます。適用されると、指定されたタイプのエンティティが近くにあり、ターゲットにできる場合、エンティティはそれに向かって向きを変えます。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|真偽値|setTarget|true|trueの場合、このエンティティは発見したエンティティを攻撃ターゲットに設定します|
|小数|searchRadius|10.0|感知距離|
|真偽値|mAllowInvulnerable|false|trueの場合、不死身のエンティティ（クリエイティブモードのプレイヤーなど）も有効なターゲットと見なされます|
|Range [a, b]|look_cooldown|[0.0, 0.0]|エンティティが「クールダウン」し、敵対やターゲットを探さないランダムな時間の範囲|
|Minecraft Filter|filters|player|このコンポーネントを起動できるエンティティを定義します|
|文字列|look_event||フィルタで指定されたエンティティがこのエンティティを発見したときに実行するイベント識別子|

#### minecraft:nameable
名前タグを使用して名前を付けるエンティティの機能と、名前が表示されるかどうかを示します。さらに、スクリプトを使用すると、エンティティの名前をプロパティ「name」で直接設定できます。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|JSON Object|name_actions||このエンティティの特別な名前と、エンティティがこれらの名前を取得したときに呼び出すイベントについて(jeb_羊かな？)<br>下表参照|
|文字列|default_trigger||エンティティに名前が付けられたときに実行するトリガー|
|真偽値|alwaysShow|false|trueの場合、名前は常に表示されます|
|真偽値|allowNameTagRenaming|true|trueの場合、このエンティティは名前タグで名前を変更できます|
|文字列|name||エンティティの現在の名前。エンティティにまだ名前が付けられていない場合は空です。空ではなくなれば、エンティティに名前が適用されます|

##### name_actions
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|on_named||このエンティティが「name_filter」で指定された名前を取得したときに呼び出されるイベント|
|List|name_filter||「on_named」で定義されたイベントを発生させる特別な名前のリスト|

#### minecraft:position
このコンポーネントを使用すると、ワールドにおけるエンティティの現在の位置を制御できます。適用されると、エンティティは指定された新しい位置にテレポートされます。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|小数|x|0.0|エンティティのx座標(東西方向)|
|小数|y|0.0|エンティティのy座標(上下方向)|
|小数|y|0.0|エンティティのx座標(南北方向)|

#### minecraft:rotation
エンティティの頭の回転だけでなく、エンティティの現在の回転を制御できます。適用されると、エンティティは指定されたとおりに回転します。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|小数|x|0.0|頭の上下方向の向き|
|小数|y|0.0|体の水平方向の回転|

#### minecraft:shooter
エンティティの範囲攻撃を定義します。これは、エンティティが範囲攻撃を使用することを許可しません、発射する発射物の種類を定義するだけです。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|def||範囲攻撃の発射物として使用するエンティティ識別子。エンティティは、発射物として発射できるようにprojectileコンポーネントを持っている必要があります|
|整数|auxVal|-1|ヒット時に適用されるポーション効果のID|

#### minecraft:spawn_entity
エンティティまたはアイテムを生成するエンティティの機能を制御します。これは、鶏が一定時間後に卵を産む能力に似ています。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|真偽値|should_leash|false|trueの場合、生成されたエンティティが親に紐づけされます|
|整数|num_to_spawn|1|生成するエンティティの数|
|整数|min_wait_time|300|次にエンティティを生成するための最小時間|
|整数|max_wait_time|600|次にエンティティを生成するための最大時間|
|文字列|spawn_sound|plop|エンティティが生成されたときに再生する効果音の識別子|
|文字列|spawn_item|egg|生成するアイテムのアイテム識別子|
|文字列|spawn_entity||生成するエンティティの識別子。空のままにした場合、代わりに上記で定義したアイテムを生成します|
|文字列|spawn_method|born|エンティティの生成に使用するメソッド|
|文字列|spawn_event|minecraft:entity_born|エンティティが生成されたときに呼び出すイベント|
|Minecraft Filter|filters||指定した場合、指定されたエンティティは、フィルターがtrueと評価された場合にのみ生成されます|
|真偽値|single_use|false|trueの場合、このコンポーネントは指定されたエンティティを一度だけ生成します|

#### minecraft:teleport
これは、エンティティのテレポート能力を制御します（エンダーマンと同様）。エンティティをテレポートしたい場合は、代わりにPositionコンポーネントを使用します。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|真偽値|randomTeleports|true|trueの場合、エンティティはランダムにテレポートします|
|小数|minRandomTeleportTime|0.0|ランダムテレポート間の最小秒数|
|小数|maxRandomTeleportTime|20.0|ランダムテレポート間の最大秒数|
|Vector [a, b, c]|randomTeleportCube|[32.0, 16.0, 32.0]|エンティティは、このベクトルで定義されたエリア内のランダムな位置にテレポートします|
|小数|targetDistance|16.0|ターゲットを追跡するときにエンティティがテレポートする最大距離|
|小数|target_teleport_chance|1.0|エンティティがテレポートする可能性<br>0.0~1.0|
|小数|lightTeleportChance|0.01|エンティティが日光の下にいる場合、エンティティがテレポートする可能性|
|小数|darkTeleportChance|0.01|エンティティが暗闇の中にいる場合、エンティティがテレポートする可能性|

#### minecraft:tick_world
tick_worldコンポーネントは読み取り専用コンポーネントであり、ユーザーはエンティティのticking areaおよびticking areaのデータにアクセスできます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|整数|radius|ticking areaのチャンク半径|
|小数|distance_to_players|プレイヤーとの距離|
|真偽値|never_despawn|プレイヤーが範囲外になったときに、このticking areaが消滅するかどうか|
|Entity Ticking Area JS API Object|ticking_area|このエンティティに紐づけられたticking area|

### Clientコンポーネント
これらのコンポーネントは、スクリプトが実行されたクライアントでのみ実行されクライアントスクリプトからのみ使用できます。  

#### minecraft:molang
MoLangコンポーネントは、エンティティ内のMoLang変数へのアクセスを提供します。 MoLang変数の詳細については、アドオンのドキュメントをご覧ください。スクリプトでは、エンティティのJSONファイルで定義されているこれらの変数を取得および設定できます。フォーマットされたMoLang変数(たとえば、variable.isgrazing)にアクセスするには、オブジェクトに [] 演算子を使用する必要があります。以下の例は [] 演算子を使用して変数にアクセスする方法を示しています。  

##### サンプルコード
```js
let molangComponent = this.createComponent(entity, "minecraft:molang");
molangComponent["variable.molangexample"] = 1.0;
this.applyComponentChanges(molangComponent);
```

### Blockコンポーネント
これらのコンポーネントはブロックオブジェクト上にのみ存在できます。  

#### minecraft:blockstate
このコンポーネントには、ブロックオブジェクトのすべてのブロックステートが含まれます。ブロックステートは、ブロックの向きから木材の種類まで、ブロックのすべての様相を制御します。ブロックステートは、数字、Boolean、または文字列で表されます。各状態の有効な値を確認するには、Blockstates Documentationを参照してください。このコンポーネントは、これらの状態の取得と設定を可能にします。  

##### サンプルコード
```js
let blockstateComponent = this.getComponent(block, "minecraft:blockstate");
blockstateComponent.data.coral_color = "blue";
this.applyComponentChanges(block, blockstateComponent);
```

## ユーザー定義 コンポーネント
ユーザー定義のコンポーネントは、スクリプトで定義できる特別な種類のコンポーネントであり、組み込みのゲームシステムは機能しません。  
コンポーネントは、name：valueという形式で名前とフィールドを指定して、Script Engineに登録する必要があります。適用されると、コンポーネントは組み込みコンポーネントと同様に動作します。エンティティから値を取得し、変更し、変更を適用できます。  
現在、ユーザー定義コンポーネントは、スクリプトを使用してエンティティに動的に追加および削除できる唯一のコンポーネントです。エンティティのJSONファイルで事前に定義する必要はありません。現在のバージョンでは、これらのコンポーネントは保存またはロードされません。エンティティが存在する間のみ存在し、レベルをリロードするときに追加し直す必要があります。  

##### サンプルコード
コンポーネント登録
```js
this.registerComponent("myNamespace:myComponent", { myString: "TamerJeison", myInt: 42, myFloat: 1.0, myArray: [1, 2, 3] });
```

## スクリプトイベント
スクリプトで監視して応答できるイベントのリスト  

### クライアントイベント
#### 監視イベント
次のイベントは、スクリプトエンジンが監視しているイベントであり、スクリプトで反応することができます。  

#### minecraft:client_entered_world
このイベントは、プレイヤーがワールドに参加するたびに発生します。イベントデータには、プレイヤーエンティティオブジェクトが含まれます。  

#### minecraft:hit_result_changed
このイベントは、レティクルがブロックまたは空気を指した状態からエンティティを指した状態に変化するたびにトリガーされます。最大1000ブロック離れています。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|指したエンティティ、レティクルをエンティティから外して発生した場合はnull|
|Vector [a, b, c]|position|指したエンティティの位置、レティクルをエンティティから外して発生した場合はnull|

##### サンプルコード
```js
system.initialize = function() {
	this.listenForEvent("minecraft:hit_result_changed", (eventData) => this.onPick(eventData)); 
};
```
```js
system.onPick = function(eventData) { 
	if (eventData.position !=== null) { 
		this.broadcastEvent("minecraft:display_chat_event", "Pick at x:" + eventData.position.x + " y:" + eventData.position.y + " z:" + eventData.position.z); 
	} 
};
```

#### minecraft:hit_result_continuous
このイベントは更新のたびにトリガーされ、レティクルが最大1000ブロック先までで指しているエンティティを通知します。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|指したエンティティ、エンティティを指していない場合null|
|Vector [a, b, c]|position|指したエンティティまたはブロックの位置|

##### サンプルコード
```js
system.initialize = function() {
	this.listenForEvent("minecraft:hit_result_continuous", (eventData) => this.onPick(eventData)); 
};
```
```js
system.onPick = function(eventData) { 
	if (eventData.position !=== null) { 
		this.broadcastEvent("minecraft:display_chat_event", "Pick at x:" + eventData.position.x + " y:" + eventData.position.y + " z:" + eventData.position.z); 
	} 
};
```

#### minecraft:pick_hit_result_changed
このイベントは、マウスポインターがブロックまたは空を指し示すものからエンティティを指すように変化するたびにトリガーされます。最大1000ブロック離れています。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|指したエンティティ、レティクルをエンティティから外して発生した場合はnull|
|Vector [a, b, c]|position|指したエンティティの位置、レティクルをエンティティから外して発生した場合はnull|

##### サンプルコード
```js
system.initialize = function() {
	this.listenForEvent("minecraft:pick_hit_result_changed", (eventData) => this.onPick(eventData)); 
};
```
```js
system.onPick = function(eventData) { 
	if (eventData.position !=== null) { 
		this.broadcastEvent("minecraft:display_chat_event", "Pick at x:" + eventData.position.x + " y:" + eventData.position.y + " z:" + eventData.position.z); 
	} 
};
```

#### minecraft:pick_hit_result_continuous
このイベントは更新のたびにトリガーされ、最大1000ブロック先のマウスポインターが指しているエンティティを通知します。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|指したエンティティ、エンティティを指していない場合null|
|Vector [a, b, c]|position|指したエンティティまたはブロックの位置|

##### サンプルコード
```js
system.initialize = function() {
	this.listenForEvent("minecraft:pick_hit_result_continuous", (eventData) => this.onPick(eventData)); 
}; 
```
```js
system.onPick = function(eventData) { 
	if (eventData.position !=== null) { 
		this.broadcastEvent("minecraft:display_chat_event", "Pick at x:" + eventData.position.x + " y:" + eventData.position.y + " z:" + eventData.position.z); 
	} 
};
```

#### 呼び出し可能イベント
次のイベントは、スクリプトから発生させ、ゲームを応答させることができます。  

#### minecraft:display_chat_event
このイベントは、クライアントスクリプトを実行している特定のプレーヤーにチャットメッセージを表示するために使用されます。イベントデータは、プレーンテキストで表示されるメッセージです。特別なフォーマットは、プレイヤーがメッセージを送信する場合と同じ方法でサポートされます。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|message||表示されるチャットメッセージ|

#### minecraft:load_ui
このイベントは、クライアントスクリプトを実行している特定のプレーヤーにUI画面を表示するために使用されます。このイベントは、UI画面をスタックの一番上に追加します。画面は、イベントがトリガーされた直後に表示されます。このイベントを使用して表示できるのは、HTMLファイルで定義された画面のみです。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|JSON Object|options||値をtrueまたはfalseに設定することにより、画面に次のオプションを定義できます<br>`always_accepts_input`<br>trueの場合、他のカスタムUI画面が画面上に表示されていても、入力を受け入れて処理します<br>`render_game_behind`<br>trueの場合、ゲーム画面はこの画面の下にレンダリングされ続けます<br>`absorbs_input`<br>trueの場合、入力はその下の他の画面に渡されません<br>`is_showing_menu`<br>trueの場合、画面は一時停止メニューとして扱われ、一時停止メニューはこの画面の上部に表示されません<br>`should_steal_mouse`<br>trueの場合、画面はマウスポインターをキャプチャし、UI画面への移動を制限します<br>`force_render_below`<br>trueの場合、別の画面がその上にある場合でも、この画面がHUDを含むそれらの上にレンダリングされます<br>`render_only_when_topmost`<br>trueの場合、この画面は、スタックの一番上の画面である場合にのみレンダリングされます|

#### minecraft:send_ui_event
このイベントは、スクリプトを実行している特定のプレーヤーのUIエンジンにUIイベントを送信するために使用されます。イベントがトリガーされると、UIイベントがすぐに送信されます。  
カスタムUIはHTML 5に基づいています。カスタムUIファイルの例については、スクリプトのデモを確認してください。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|文字列|eventIdentifier|UIイベントの識別子|
|文字列|data|トリガーされるUIイベントのデータ|

#### minecraft:spawn_particle_attached_entity
このイベントを使用して、エンティティの周りを追従するパーティクルを作成します。このパーティクルは、イベントを発生させたクライアントスクリプトを実行している特定のプレーヤーにのみ表示されます。  
ここでは、JSONファイル（リソースパックとMinecraftの両方）で定義されているエフェクトを使用できます。  
エフェクトのJSONで定義されたMoLang変数は、アタッチされたエンティティで変更することにより、そのエフェクトを制御するために使用できます。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|effect||エンティティに追従するパーティクルエフェクトの識別子。これは、JSONファイルで指定した名前と同じです|
|Vector [a, b, c]|offset|[0, 0, 0]|エフェクトをスポーンするエンティティの「中心」からのオフセット|
|Entity JS API Object|entity||エフェクトを追従するエンティティオブジェクト|

#### minecraft:spawn_particle_in_world
このイベントは、世界で静的なパーティクルを作成するために使用されます。このパーティクルは、イベントを発生させたクライアントスクリプトを実行している特定のプレーヤーにのみ表示されます。  
ここでは、JSONファイル（リソースパックとMinecraftの両方）で定義されているエフェクトを使用できます。エフェクトが生成されると、それ以上制御できなくなります。  
サーバーバージョンのイベントとは異なり、クライアントバージョンは、プレーヤーが現在いるディメンションにパーティクルを生成します。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|effect||パーティクルの識別子。これは、JSONファイルで効果を指定した名前と同じです|
|Vector [a, b, c]|position|[0, 0, 0]|パーティクルを出現させる座標|

#### minecraft:unload_ui
このイベントは、クライアントスクリプトを実行している特定のプレーヤーのスタックからUI画面を削除するために使用されます。イベントデータには、削除する画面の名前が文字列として格納されています。イベントがトリガーされた後、画面は、UIエンジンが次回削除できるときにスタックから削除されるようにスケジュールされます。このイベントを使用して削除できるのは、HTMLファイルで定義された画面のみです。  

#### minecraft:script_logger_config
このイベントは、クライアントスクリプトのさまざまなレベルのログをオンまたはオフにするために使用されます。ログのオン/オフの切り替えは、イベントをブロードキャストしたスクリプトに限定されないことに注意してください。これは、世界に適用される他のビエイビアパックのスクリプトを含むすべてのクライアントスクリプトに影響します。ログの詳細については、「デバッグ」セクションを参照してください。  

### サーバーイベント
#### 監視イベント
次のイベントは、スクリプトエンジンが監視しているイベントであり、スクリプトで反応することができます。  

#### minecraft:player_attacked_entity
このイベントは、プレイヤーがエンティティを攻撃するたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|player|エンティティを攻撃したプレイヤー|
|Entity JS API Object|attacked_entity|プレイヤーによって攻撃されたエンティティ|

#### minecraft:entity_acquired_item
このイベントは、エンティティがアイテムを取得するたびにトリガーされます  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|アイテムを取得したエンティティ|
|ItemStack JS API Object|item_stack|取得したアイテム|
|文字列|acquisition_method|エンティティがアイテムを取得した方法|
|整数|acquired_amount|このイベント中にエンティティが取得したアイテムの総数|
|Entity JS API Object|secondary_entity|存在する場合、アイテムが取得される前にアイテムに影響を与えたエンティティ<br>例：プレイヤーが村人との取引をした場合、 「entity」プロパティがプレイヤーになり、「secondary_entity」が村人になります|

#### minecraft:entity_carried_item_changed
このイベントは、エンティティが手に持っているアイテムを変更するたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|手持ちを変更したエンティティ|
|ItemStack JS API Object|previous_carried_item|前にエンティティの手にあったアイテム|
|ItemStack JS API Object|carried_item|現在エンティティの手にあるアイテム|

#### minecraft:entity_created
このイベントは、エンティティがワールドに追加されるたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|作成されたばかりのエンティティ|

#### minecraft:entity_death
このイベントは、エンティティが消滅するたびにトリガーされます。エンティティが削除されたとき(destroyEntityを使用するときなど)これはトリガーされません。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|死んだエンティティ|

#### minecraft:entity_dropped_item
このイベントは、エンティティがアイテムをドロップするたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|アイテムをドロップしたエンティティ|
|ItemStack JS API Object|item_stack|ドロップされたアイテム|

#### minecraft:entity_equipped_armor
このイベントは、エンティティがアーマースロットにアイテムを装備するたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|アーマーを装備したエンティティ|
|ItemStack JS API Object|item_stack|装備した鎧|

#### minecraft:entity_start_riding
このイベントは、エンティティが別のエンティティにのるたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|乗った方のエンティティ|
|Entity JS API Object|ride|乗られた方のエンティティ|

#### minecraft:entity_stop_riding
このイベントは、エンティティが別のエンティティの乗車を停止するたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|別のエンティティに乗っていたエンティティ|
|真偽値|exit_from_rider|trueの場合、ライダーは自分の判断で乗りを停止した|
|真偽値|entity_is_being_destroyed|trueの場合、エンティティが死んだため、乗りを停止した|
|真偽値|switching_rides|trueの場合、ライダーは別のエンティティに乗りかえたため、乗りを停止した|

#### minecraft:entity_tick
このイベントは、エンティティに対し1tickごとにトリガーされます。このイベントは、プレイヤーには発生しません。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|エンティティ|

#### minecraft:entity_use_item
このイベントは、エンティティがアイテムを使用するたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|entity|アイテムを使用したエンティティ|
|ItemStack JS API Object|item_stack|使用したアイテム|
|文字列|use_method|アイテムの使用法|

#### minecraft:block_destruction_started
このイベントは、プレイヤーがブロックの破壊を開始するたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|player|ブロックを破壊し始めたプレイヤー|
|JavaScript Object|block_position|ブロックの座標|

#### minecraft:block_destruction_stopped
このイベントは、プレイヤーがブロックの破壊を停止するたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|player|ブロックの破壊をやめたプレイヤー|
|JavaScript Object|block_position|ブロックの座標|
|小数|destruction_progress|破壊を停止するまでの時間<br>0~1の範囲|

#### minecraft:block_interacted_with
このイベントは、プレーヤーがブロックに手で触るたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|player|ブロックに接触したプレイヤー|
|JavaScript Object|block_position|触ったブロックの座標|

#### minecraft:piston_moved_block
このイベントは、ピストンがブロックを移動するたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|JavaScript Object|piston_position|ブロックを動かしたピストンの座標|
|JavaScript Object|block_position|動かされたブロックの座標|
|文字列|piston_action|ピストンの動き、"extended"か"retracted"|

#### minecraft:player_destroyed_block
このイベントは、プレイヤーがブロックを破壊するたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|player|ブロックを破壊したプレイヤー|
|JavaScript Object|block_position|ブロックの座標|
|文字列|block_identifier|ブロックの識別子|

#### minecraft:player_placed_block
このイベントは、プレイヤーがブロックを設置するたびにトリガーされます。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|Entity JS API Object|player|ブロックを設置したプレイヤー|
|JavaScript Object|block_position|ブロックの座標|

#### minecraft:play_sound
このイベントは、効果音を再生するために使用されます。現在、サウンドは固定位置でのみ再生できます。グローバルサウンドとエンティティが再生するサウンドは、後のアップデートで実装される予定です。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|sound||再生したいサウンドの識別子。適用されたリソースパックで定義されたサウンドのみを再生できます|
|小数|volume|1.0|効果音の音量。 1.0の場合、記録された音量で効果音を再生します|
|小数|pitch|1.0|効果音のピッチ。 1.0の場合、通常のピッチで効果音を再生します|
|Vector [a, b, c]|position|[0, 0, 0]|サウンドを再生したい座標|

#### minecraft:weather_changed
このイベントは、天気が変わるたびにトリガーされます。変化する天気に関する情報が含まれています。  

##### パラメータ
|型|名前|説明|
|:--:|:--:|:--:|
|文字列|dimension|天気の変化が発生したディメンションの名前|
|真偽値|raining|新しい天気に雨が降っているかどうか|
|真偽値|lightning|新しい天気に雷が鳴っているかどうか|

#### 呼び出し可能イベント
次のイベントはスクリプトからトリガーでき、ゲームはそれに応じて応答します。  

#### minecraft:display_chat_event
このイベントは、サーバーからプレイヤーにチャットメッセージを送信するために使用されます。イベントデータは、文字列として送信されるメッセージです。特別なフォーマットは、プレイヤーがメッセージを送信する場合と同じ方法でサポートされます。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|message||表示されるメッセージ|

#### minecraft:execute_command
このイベントは、ホストの権限レベルでサーバー上でコマンドを実行するために使用されます。イベントデータには、コマンドが文字列として含まれています。コマンドが処理され、イベントの送信後に実行されます  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|command||実行されるコマンド|

#### minecraft:play_sound
このイベントは、効果音を再生するために使用されます。現在、サウンドは固定位置でのみ再生できます。グローバルサウンドとエンティティが再生するサウンドは、後のアップデートで実装される予定です。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|sound||再生したいサウンドの識別子。適用されたリソースパックで定義されたサウンドのみを再生できます|
|小数|volume|1.0|効果音の音量。 1.0の場合、記録された音量で効果音を再生します|
|小数|pitch|1.0|効果音のピッチ。 1.0の場合、通常のピッチで効果音を再生します|
|Vector [a, b, c]|position|[0, 0, 0]|サウンドを再生したい座標|

#### minecraft:spawn_particle_attached_entity
このイベントを使用して、エンティティの周りを追従するパーティクルを作成します。このパーティクルは、全てのプレイヤーに表示されます。  
ここでは、JSONファイル（リソースパックとMinecraftの両方）で定義されているエフェクトを使用できます。  
エフェクトのJSONで定義されたMoLang変数は、アタッチされたエンティティで変更することにより、そのエフェクトを制御するために使用できます。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|effect||エンティティに追従するパーティクルエフェクトの識別子。これは、JSONファイルで指定した名前と同じです|
|Vector [a, b, c]|offset|[0, 0, 0]|エフェクトをスポーンするエンティティの「中心」からのオフセット|
|Entity JS API Object|entity||エフェクトを追従するエンティティオブジェクト|

#### minecraft:spawn_particle_in_world
このイベントは、世界で静的なパーティクルを作成するために使用されます。このパーティクルは、すべてのプレーヤーに表示されます。  
ここでは、JSONファイル（リソースパックとMinecraftの両方）で定義されているエフェクトを使用できます。エフェクトが生成されると、それ以上制御できなくなります。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|文字列|effect||エンティティに追従するパーティクルエフェクトの識別子。これは、JSONファイルで指定した名前と同じです|
|Vector [a, b, c]|position|[0, 0, 0]|パーティクルを出現させる座標|
|文字列|dimension|overworld|エフェクトをスポーンするディメンション。"overworld", "nether", または"the end"|

#### minecraft:script_logger_config
このイベントは、サーバースクリプトのさまざまなレベルのログをオンまたはオフにするために使用されます。ログのオン/オフの切り替えは、イベントをブロードキャストしたスクリプトに限定されないことに注意してください。ワールドに適用された他のBehavior Packのスクリプトを含むすべてのサーバースクリプトに影響します。ログの詳細については、「デバッグ」セクションを参照してください。  

##### パラメータ
|型|名前|デフォルト値|説明|
|:--:|:--:|:--:|:--:|
|真偽値|log_errors|false|サーバーで発生したスクリプトエラーをログに記録するには、trueに設定します|
|真偽値|log_warnings|false|trueに設定すると、サーバーで発生するスクリプト警告をログに記録します|
|真偽値|log_information|false|サーバーで発生する一般的なスクリプト情報を記録するには、trueに設定します。これには、server.log()で行われたログが含まれます|