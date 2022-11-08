# Spring Boot で軽量 DB を使った REST API の作成を体験するチュートリアル

Spring Boot は、API の作成が、すばやくできるフレームワークである。

しかし、DB を使った REST API を作ろうとなると、
DB のインストールが最初は大変に感じたりする。

そこで、REST API の作成ってこんなもんだよ、というのを体験してもらうために、
インストール不要で軽量な DB を使ったチュートリアルを用意した。

今回は、H2 というデータベースを利用する。

以下は、スムーズにこなせれば、30分ほどで実施できる内容となる。

## 環境

- Windows 11 Home
- Java 17 (Microsoft.OpenJDK.17)
- VSCode

拡張機能として、VSCode に以下をインストールしておく。

- Extension Pack for Java
- Spring Boot Extension Pack

## 手順

1. プロジェクトの作成
2. 作成されたプロジェクト内容の確認
3. application 設定ファイルへの DB 設定
4. モデルの作成
5. サービスの作成
6. Rest API コントローラの作成
7. 拡張機能 REST Client のインストール
8. http ファイルの作成
9. 一度動かしてみる
10. API を叩く
11. H2 コンソールで確認
12. サービスクラスへのメソッド追加
13. コントローラへの API 追加
14. 最後に REST Client で動作確認

## 1. プロジェクトの作成

VSCode を開き、Ctrl + Shift + P キーを押す。

コマンドパレットが開いたら、`Spring Initializr` と入力して、
コマンド「Spring Initializr: Generate a Gradle Project」を選択する。

次に `Specify Spring Boot version.` Spring Boot のバージョンを指定してね、と言われるので、使いたい Spring Boot のバージョンを指定する。

ここでは `2.6.14(SNAPSHOT)` を選択した。

次に、`Specify project language` と問われるので、プロジェクトに使用する言語を選択する。

ここでは、`Java` を選択。

次に `Input Group Id` グループ ID の入力を求められる。

ここでは、デフォルトのまま、「com.example」として、Enter キーを押下。

次に `Input Artifact Id` アーティファクト ID の入力を求められる。

アーティファクト ID と言われると何のことか分からないが、「プロジェクト名」的な役割として使われる ID となる。

ここでは、`vegi-h2-api` （入出力テストアプリ）とした。

次に `Specify pacakaging type.` パッケージタイプの指定を求められる。

ここでは、「Jar」を選択した。

次に `Specify Java version.` と、使用する Java のバージョンを求められる。

ここでは、安定版の Java バージョンである 17 を指定した。

次に `Choose Dependencies.` 依存ライブラリを選択してね、と問われる

ここでは、以下の2つのライブラリを選択した。

- Spring Web（Web API を作るため）
- Spring Data JPA（Java クラスとテーブルを紐づけるため）
- Lombok（Getter, Setter のコーディングを省略するため）
- Spring DevTools（ホットリロードを可能とするため）
- H2 Database（データベース）

この5つを選んで、「Select 5 Dependencies」となった状態で Enter キーを押下。

次に、プロジェクトを作成する場所を聞かれるので、
プロジェクトを作成するフォルダを選択する。

ファイルエクスプローラから、フォルダを選択したら、「Generate into this folder」ボタンを押下する。

Spring Boot プロジェクトの作成に成功すると `Successfully generated.` と
メッセージが表示される。

そのまま、プロジェクトを VSCode で開きたい場合は、「Open」ボタンをクリックする。

## 2. 作成されたプロジェクト内容の確認

プロジェクトを確認すると、プロジェクトにある Java クラスは、
`VegiH2ApiApplication.java` のみとなっている。

このクラスが、この Spring Boot アプリケーションの起点となる。

`src/main/java/com/example/vegiapi/VegiH2ApiApplication.java`
``` java
@SpringBootApplication
public class VegiH2ApiApplication {

	public static void main(String[] args) {
		SpringApplication.run(VegiH2ApiApplication.class, args);
	}
}
```

## 3. application 設定ファイルへの DB 設定

プロジェクトには、アプリケーション設定ファイルとして、
標準で `application.properties` が作られているが、
このファイルは削除しておく。

代わりに、設定項目が見やすい YAML 形式のファイル、
`application.yml` を以下の内容で作成する。

`src/main/resource/application.yml`
``` yaml
spring:
  datasource:
    url: jdbc:h2:file:./h2db/vegitable
    driverClassName: org.h2.Driver
    username: sa
    password: pass
    
  jpa:
    show-sql: true
    database-platform: org.hibernate.dialect.H2Dialect
    hibernate:
      ddl-auto: update

  h2:
    console:
      enabled: true
      # default path: h2-console
      path: /h2-ui
```

### application.yml 内の設定値の説明

**spring.datasource.url:**  
`jdbc:h2:mem:{データベース名}` を指定した場合、インメモリにデータが読み書きされる。
つまり、アプリが終了した場合、保存していたデータはすべて消えることになる。

`jdbc:h2:file:{パス}` を指定した場合は、パスで指定された場所にファイルを作成し、データを保存する。

**spring.datasource.username:**  
**spring.datasource.password:**  
このプロパティは、後ほどインストール設定する H2 データベースと同じ値にしておく。

**spring.jpa.properties.hibernate.dialect:**  
Spring Boot は JPA の実装に Hibernate（ORマッパーのひとつ、DB のテーブル
と Java オブジェクトを紐づける仕組み）を使用しているので、H2Dialect を H2Database に設定する。

**spring.jpa.hibernate.ddl-auto:**  
データベースの初期化に使用する項目。
設定可能な値を以下の通り。

- **none** :何もしない
- **validate** :検証だけ行い、DB には変更を加えない。本番運用時にこの値を設定する。
- **update** :アプリ起動時に、Entity に対応するテーブルがなければ作成する。
- **create** :Entity に対応するテーブルがなければ作成、あればデータを削除する。
- **create-drop** :Entity に対応するテーブルがなければ作成、アプリ終了時にスキーマを削除する。

**spring.h2.console.enabled:** 
true を指定すると、Spring が H2 Database 管理ツールを起動するようになり、
ブラウザからこのツールに `http://localhost:8080/h2-console` でアクセスできるようになる。

**spring.h2.console.path:**  
この値は、H2 コンソールの URL を表すので、値を `/h2-ui` とした場合、デフォルトの URL 
`http://localhost:8080/h2-console` は `http://localhost:8080/h2-ui` に変更される。

## 4. モデルの作成

データモデルは Vegitable で、id, name, color, price, createdAt, updatedAt の
6つのフィールドを持つ。

model パッケージで、Vegitable クラスを定義する。

`model/Vegitable.java`
``` java
@Entity
@Data
@Table(name = "VEGITABLES")
public class Vegitable {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private long id;

	@Column(name = "NAME", nullable = false, unique=true)
	private String name;

    @Column(name = "COLOR")
	private String color;

	@Column(name = "PRICE")
	private int price;

    @Column(name = "CREATED_AT", nullable = false)
    private LocalDateTime createdAt;

    @Column(name = "UPDATED_AT", nullable = false)
    private LocalDateTime updatedAt;
}
```

**@Entity**  
そのクラスが永続的な Java クラスであることを示す。

**@Table**  
この Entity をマッピングするテーブルを指定する。

**@Id**  
主キーとする項目を指定する。

**@GeneratedValue**  
主キーの生成方法を定義する。
`GenerationType.IDENTITY` は、データベースの IDENTITY 列を利用して，プライマリキー値を生成することを意味する。

**@Column**  
マッピングするデータベース内のカラムを定義する。

## リポジトリインタフェースの作成

データベースから Vegitables テーブルを操作するためのリポジトリを作成する。

Repository パッケージで、JpaRepository を継承した VegitableRepository インターフェイスを作成する。

`repository/VegitableRepository.java`
``` java
public interface VegitableRepository extends JpaRepository<Vegitable, Long> {
    List<Vegitable> findByColor(String color);
    List<Vegitable> findByNameContaining(String name);
}
```

これで、JpaRepository のメソッドである 
save(), findOne(), findById(), findAll(), count(), delete(), deleteById()... 
などを実装せずに使用できるようになった。

また、カスタムの finder メソッドも定義している。

- findByColor()：指定された color を持つ全ての Vegitable を返す
- findByNameContaining(): 指定された名前が含まれる Vegitable をすべて返す

この実装は、Spring Data JPAによって自動的にプラグインされる。

## 5. サービスの作成

サービスクラスの役割は、対象データの処理（ビジネスロジック）を管理すること。

Vegitable のサービスに、まずは以下の２つのメソッドを作っていく。

- 対象 ID の Vegitable を取得するメソッド
- 受け取った Vegitable を登録するメソッド

`service/VegitableService.java`
``` java
@Service
public class VegitableService {

    @Autowired
    VegitableRepository vegitableRepository;

    public Vegitable getById(long id) {
        Optional<Vegitable> vegitableData = this.vegitableRepository.findById(id);

        if (vegitableData.isPresent()) return vegitableData.get();
        return null;
    }

    public Vegitable create(Vegitable vegitable) {
        LocalDateTime now = LocalDateTime.now();

        vegitable.setCreatedAt(now);
        vegitable.setUpdatedAt(now);
        return this.vegitableRepository.save(vegitable);
    }
}
```

## 6. Rest API コントローラの作成

サービスにビジネスロジックを実装したら、
次は、API の問い合わせ窓口となるコントローラを作成する。

`controller/VegitableController.java`
``` java
@RestController
@RequestMapping("/vegitable")
public class VegitableController {

	@Autowired
    VegitableService vegitableService;

    @GetMapping("/{id}")
    public ResponseEntity<Vegitable> getById(@PathVariable("id") long id) {
        Vegitable vegitable = this.vegitableService.getById(id);

        if (vegitable != null) return new ResponseEntity<>(vegitable, HttpStatus.OK);
        return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }

    @PostMapping("/")
    public ResponseEntity<Vegitable> createVegitable(@RequestBody Vegitable vegitable) {
        try {
            Vegitable resultVegitable = this.vegitableService.create(vegitable);
            return new ResponseEntity<>(resultVegitable, HttpStatus.CREATED);
        } catch (Exception e) {
            return new ResponseEntity<>(null, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}
```

- RestController アノテーションは、コントローラを定義し、メソッドの戻り値をレスポンスボディにバインドすることを示す
- `@RequestMapping("/vegitable")` は、コントローラ内の全ての API の URL が `/vegitable` で始まることを宣言している
- `@Autowired` は、ローカル変数 vegitableService に `VegitableService` のインスタンスを注入する

## 7. 拡張機能 REST Client のインストール

VSCode に REST Client という拡張機能をインストールする。

## 8. http ファイルの作成

REST Client を使うと、http ファイルに記載した　URL をクリックして API への問い合わせができるようになる。

コントローラに実装した API を試すために、以下の内容の http ファイルを作成する。

`rest-client/vegi-h2-api.http`
``` md
### getById(id)
GET http://localhost:8080/vegitable/1

### create(vegitable)
POST http://localhost:8080/vegitable/
content-type: application/json

{
  "name": "赤パプリカ",
  "color": "赤",
  "price": "100"
}
```

## 9. 一度動かしてみる

ここまでできたら、アプリを一度動かしてみる。

アプリを動かすには、アプリの起点である `VegiH2ApiApplication.java` クラスを確認する。

`VegiH2ApiApplication.java` クラスを
VSCode のエディタで確認すると、main メソッドの上に「Run」ボタンが表示されているはずだ。

この「Run」ボタンをクリックすると、開発用サーバが起動し、
コンソールに起動ログが表示されるはずだ。

ログの出力が一旦止まれば、起動が完了している

## 10. API を叩く

作成した vegi-h2-api.http ファイルを開いて、
URL を見ると、URL の上部に「Send Request」とリンクが表示される。

まずは、`GET http://localhost:8080/vegitable/1` を叩いてみる。

当然、まだ何も登録されていないので、データは取得できない。

次に、`POST http://localhost:8080/vegitable/` を叩く。これで JSON に記載したデータが登録される。

登録後、再度 `GET http://localhost:8080/vegitable/1` を叩いてみる。今度は登録されたデータが取得できるはずだ。

## 11. H2 コンソールで確認

API からだけではなく、データベースの内容は、H2 の管理画面からも確認できる。

H2 の管理画面を見るには、
サーバを起動させた状態で、`http://localhost:8080/h2-ui` にアクセスする。

ログイン画面が表示されたら、
`JDBC URL:` の欄に DB ファイルのパスを入力する。

アプリが起動した状態であれば、application.yml で設定した通り、
プロジェクト直下の h2db フォルダ内に DB ファイルが作られている。

VSCode でこのファイルを右クリックして、「Copy Path」を選択すれば、
DB ファイルのパスを取得できるので、この情報をログインに利用する。

`JDBC URL:` の入力欄には、DB ファイルのパスから、`.mv.db` の拡張子を削除して、
先頭に `jdbc:h2:` を付けた文字列を入力する

あとは、application.yml に設定した内容にしたがって、パスワードを入力する。

`TestConnection` ボタンをクリックして、
「Test successful」となれば、ログイン情報は正しいことが確認できるので、
確認できれば、「Connect」ボタンを押してログインする。

サイドバーからテーブルをクリックすると、
エディタに SELECT 文が表示されるので、この状態で「Run」ボタンをクリックすると、
SQL 文が実行され、テーブルのデータが表示される。

これで一旦、動作確認を中断したいので、アプリを停止させる。

## 12. サービスクラスへのメソッド追加

動作確認で、１件の登録、１件の取得は

今度は全件取得、更新、１件削除、全件削除、野菜名による取得、色指定による取得を行うためのメソッドを
サービスに実装していく。

`service/VegitableService.java`
``` java
@Service
public class VegitableService {

    @Autowired
    VegitableRepository vegitableRepository;

    public List<Vegitable> getAll(String name) {
        List<Vegitable> vegitableList = new ArrayList<Vegitable>();
        
        if (name == null)
                this.vegitableRepository.findAll().forEach(vegitableList::add);
            else
                this.vegitableRepository.findByNameContaining(name).forEach(vegitableList::add);
        return vegitableList;
    }

    public Vegitable getById(long id) {
        Optional<Vegitable> vegitableData = this.vegitableRepository.findById(id);

        if (vegitableData.isPresent()) return vegitableData.get();
        return null;
    }

    public Vegitable create(Vegitable vegitable) {
        LocalDateTime now = LocalDateTime.now();

        vegitable.setCreatedAt(now);
        vegitable.setUpdatedAt(now);
        return this.vegitableRepository.save(vegitable);
    }

    public Vegitable update(long id, Vegitable vegitable) throws Exception {
        Optional<Vegitable> vegitableData = this.vegitableRepository.findById(id);

        if (vegitableData.isPresent()) {
            Vegitable resultVegitable = vegitableData.get();
            resultVegitable.setName(vegitable.getName());
            resultVegitable.setColor(vegitable.getColor());
            resultVegitable.setPrice(vegitable.getPrice());

            LocalDateTime now = LocalDateTime.now();
            resultVegitable.setUpdatedAt(now);
            return this.vegitableRepository.save(resultVegitable);
        } else {
            throw new Exception();
        }
    }

    public void deletedById(long id) throws Exception {
        Optional<Vegitable> vegitableData = this.vegitableRepository.findById(id);

        if (vegitableData.isPresent()) {
            this.vegitableRepository.deleteById(id);
            return;
        }
        throw new Exception();
        
    }

    public void deleteAllVegitables() throws Exception {
        this.vegitableRepository.deleteAll();
    }

    public List<Vegitable> findByColor(String color) {
        return this.vegitableRepository.findByColor(color);
    }
}
```

## 13. コントローラへの API 追加

サービスに実装したビジネスロジックを使用して、コントローラに API を追加していく。

`controller/VegitableController.java`
``` java
@RestController
@RequestMapping("/vegitable")
public class VegitableController {

    @Autowired
    VegitableService vegitableService;

    @GetMapping("/list")
    public ResponseEntity<List<Vegitable>> getAll(@RequestParam(required = false) String name) {
        try {
            List<Vegitable> vegitableList = this.vegitableService.getAll(name);

            if (vegitableList.isEmpty()) {
                return new ResponseEntity<>(HttpStatus.NO_CONTENT);
            }

            return new ResponseEntity<>(vegitableList, HttpStatus.OK);
        } catch (Exception e) {
            return new ResponseEntity<>(null, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @GetMapping("/{id}")
    public ResponseEntity<Vegitable> getById(@PathVariable("id") long id) {
        Vegitable vegitable = this.vegitableService.getById(id);

        if (vegitable != null) return new ResponseEntity<>(vegitable, HttpStatus.OK);
        return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }

    @PostMapping("/")
    public ResponseEntity<Vegitable> create(@RequestBody Vegitable vegitable) {
        try {
            Vegitable resultVegitable = this.vegitableService.create(vegitable);
            return new ResponseEntity<>(resultVegitable, HttpStatus.CREATED);
        } catch (Exception e) {
            return new ResponseEntity<>(null, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @PutMapping("/{id}")
    public ResponseEntity<Vegitable> update(@PathVariable("id") long id, @RequestBody Vegitable vegitable) {
        try {
            Vegitable resultVegitable = this.vegitableService.update(id, vegitable);
            return new ResponseEntity<>(resultVegitable, HttpStatus.OK);
        } catch (Exception e) {
            e.printStackTrace();
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<HttpStatus> delete(@PathVariable("id") long id) {
        try {
            this.vegitableService.deletedById(id);
            return new ResponseEntity<>(HttpStatus.NO_CONTENT);
        } catch (Exception e) {
            return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @DeleteMapping("/")
    public ResponseEntity<HttpStatus> deleteAll() {
        try {
            this.vegitableService.deleteAllVegitables();
            return new ResponseEntity<>(HttpStatus.NO_CONTENT);
        } catch (Exception e) {
            return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
        }

    }

    @GetMapping("/")
    public ResponseEntity<List<Vegitable>> findByColor(@RequestParam(required = false) String color) {
        try {
            List<Vegitable> vegitableList = this.vegitableService.findByColor(color);
            
            if (vegitableList.isEmpty()) return new ResponseEntity<>(HttpStatus.NO_CONTENT);
            return new ResponseEntity<>(vegitableList, HttpStatus.OK);
        } catch (Exception e) {
            return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}
```

### 14. 最後に REST Client で動作確認

登録、取得、更新、削除・・・いわゆる CRUD 処理（Create, Read, Update, Delete）がひと通り実装できたので、
REST Client で動作確認をする。

`rest-client/vegitable-api.http`
``` md
### getAll()
GET http://localhost:8080/vegitable/list

### getAll(name)
GET http://localhost:8080/vegitable/list?name=パプリカ

### getById(id)
GET http://localhost:8080/vegitable/1

### create(vegitable)
POST http://localhost:8080/vegitable/
content-type: application/json

{
  "name": "赤パプリカ",
  "color": "赤",
  "price": "100"
}

### update(id, vegitable)
PUT http://localhost:8080/vegitable/1
content-type: application/json

{
  "name": "赤パプリカ",
  "color": "赤",
  "price": "777"
}

### delete(id)
DELETE http://localhost:8080/vegitable/1

### deleteAll()
DELETE http://localhost:8080/vegitable/

### findByColor(color)
GET http://localhost:8080/vegitable/?color=赤
```

いろいろなパターンで API に問い合わせをしてみたり、
処理の不明点を調べてみたり、改造してみたりして、REST API 開発に慣れ、知識や経験を広げてほしい。

ひと通り、今回の実装を体験してから、
ゼロから自分で同じ機能を実装してみると、力が付くと思う。

LocaleDateTime とは何か、Optional とは何か、ResponseEntity とは何か、
POST とは何か、PUT とは何か、というのを調べて理解しながら。

以上。
