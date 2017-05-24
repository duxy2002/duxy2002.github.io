# LiquiBase
LiquiBaseを使用してDBスキーマの定義を行うには、スキーマの「変更ログ」を記述したXMLファイルを用意する必要がある。 

Each change set is uniquely identified by an “id” attribute and an “author” attribute. 
各changeSetは、id属性とauthor属性の組み合わせによってファイル内で一意に特定されるため、これらの属性は必須だ。
テーブル(databasechangelogとdatabasechangeloglock)は、LiquiBaseがスキーマの変更管理情報を格納するために利用するものだ。
The databasechangelog table contains a list of all the statements that have been run against the database.  
The databasechangeloglock table is used to make sure two machines don’t attempt to modify the database at the same time.  
changeSetタグで指定したidとauthor、そしてチェンジログファイルの名前(FILENAME列)によって行が一意に特定されていることがおわかりだろう。

## Get Started
1. Download Liquibase
2. Create new changelog file in XML, YAML, JSON or SQLformat 
3. Add changeset to changelog file
4. Run liquibase update
5. Commit changelog file to source control
6. GOTO 3

## スキーマへのタグ付け

開発者shiraishiは、データベースの変更を伴うコード修正を行う。開発時は何が起きるかわからないため、いつでもスキーマを元に戻せるように、現在のスキーマにタグを付けておくことにする。

タグをつけるために使用するサブコマンドは「tag」だ。以下のように、「liquibase tag ＜タグ名＞」と指定することで、現在のスキーマの状態にタグを付けておき、後で参照することができる (JDBC関連のオプションは省略)。
```
liquibase tag before_update_person
```

ここでは、「before_update_person」というタグを指定している。

一旦自分のスキーマをタグ「before_update_person」までロールバックする。

ロールバックするためのサブコマンドは「rollback」だ。以下のようにして、タグ名を指定してコマンドを実行すれば、スキーマを以前の状態に戻すことができる。
```
> liquibase rollback before_update_person
```
とはいえ、実際にはあらゆるスキーマ操作がロールバックできるわけではない。例えばDROP TABLEやMODIFY COLUMNなどは元に戻せない。

「rollback」以外にも、ロールバックを行うためのサブコマンドが存在し、タグ以外の方法で「どこまで戻すか」を指定することもできる。
rollbackToDate - 指定した時点のスキーマまでロールバックする
rollbackCount - 指定した数ぶん、changeSetをロールバックする


## シナリオ2 (Diffとドキュメント出力)

さて、開発も順調に進み、そろそろ本番環境へのDBマイグレーションを考慮しなければならなくなったとしよう。

その際に必要なのは、開発用DBと本番用DBに現在どれだけの差異があるか、を確認することだ。LiquiBaseはそのための機能を「diff」サブコマンドで提供している。 diffサブコマンドの使用法は、以下の通りだ。
liquibase diff --baseUrl=＜比較元となるDBのURL＞
               --baseUsername=＜比較元となるDBのユーザ名＞
               --basePassword=＜比較元となるDBのパスワード＞


本番環境のデータベース名が「production」だとすると、実行するコマンドとその出力結果は以下のようになる。
```
> liquibase diff --baseUrl=jdbc:mysql://localhost/production --baseUsername=shumpei --basePassword=*****
Diff Results:
Base Database: shumpei jdbc:mysql://localhost/dev
Target Database: shumpei jdbc:mysql://localhost/production
Product Name: EQUAL
Product Version: EQUAL
Missing Tables: NONE
Unexpected Tables: NONE

…略…

Missing Columns:
     person.is_deleted

…略…
```

「…略…」に挟まれた部分に、「productionデータベースにはpersonテーブルのis_deleted列がない」と表示されているのがお解りだろう。このように、diffサブコマンドを用いれば出力されたデータベース間の差異を確認することができる。


またdiffChangeLogというサブコマンドを用いれば、DB間の差分を埋めるためのコードをチェンジログの形で出力させることができる。
```
> liquibase diffChangeLog --baseUrl=jdbc:mysql://localhost/production --baseUsername=shumpei --basePassword=*****

Reading tables for shumpei @ jdbc:mysql://localhost/dev ...

…略…

Reading sequences for shumpei @ jdbc:mysql://localhost/production ...
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.3"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="ht
tp://www.liquibase.org/xml/ns/dbchangelog/1.3 http://www.liquibase.org/xml/ns/db
changelog/dbchangelog-1.3.xsd">
    <changeSet author="diff-generated" id="1192530329578-1">
        <addColumn tableName="person">
            <column defaultValue="0" name="is_deleted" type="BIT"/>
        </addColumn>
    </changeSet>
</databaseChangeLog>

```
こうして生成したチェンジログを直接、もしくは一部利用してマイグレーションを行うことも可能だ。

さらに、LiquiBaseはデータベースとチェンジログの情報を、JavaDocに似た形式のドキュメントとして出力することも可能だ。そのためには「dbDoc」サブコマンドに、出力先のディレクトリパスを指定してコマンドを実行する。

```
>liquibase dbDoc .\docs\db
```
このコマンドを実行すると、現在のディレクトリ以下にdocs\dbというディレクトリツリーが生成され、その中にHTMLドキュメントが出力される。
このドキュメントには、データベース内のオブジェクトやチェンジログにエントリを記述した開発者 (author属性の値)などが表示される。さらに、DBに反映されていないチェンジログの状況も表示されるので、チェンジログ⇔DBの間の差分を確認することも可能だ。これを利用すれば、本番環境のDBと、開発用DBのチェンジログを比較して、ドキュメント化することもできる。


## 現在使用可能なサブコマンド一覧

以下に、現在のLiquiBaseで使用可能なサブコマンドの一覧を示す。これを見れば、LiquiBaseがまさに「かゆい所に手が届く」ツールであるかがお解りだろう。
* migrate DBスキーマをチェンジログに合わせて最新にする 
* migrateSQL migrateコマンドによって発行されるSQLを出力する 
* rollback <tag> タグが指定された時点までDBの状態をロールバックする 
* rollbackToDate <date/time> 指定した日時までDBの状態をロールバックする 
* rollbackCount <value> 指定した回数分の変更をロールバックする 
* rollbackSQL <tag> タグが指定された時点までDBの状態をロールバックするためのSQLを、標準出力に表示する 
* rollbackToDateSQL <date/time> 指定した日時までDBの状態をロールバックするためのSQLを、標準出力に表示する 
* rollbackCountSQL <value> 指定した回数分の変更をロールバックするためのSQLを、標準出力に表示する 
* futureRollbackSQL チェンジログを適用した状態から、現在のDBの状態までロールバックするためのSQLを出力する 
* generateChangeLog データベースの状態から、チェンジログを生成して表示する 
* diff [diff parameters] DBの比較結果を表示する 
* diffChangeLog [diff parameters] diffの結果をチェンジログとして表示する 
* dbDoc <outputDirectory> DBの状態とチェンジログを、Javadocライクなドキュメントとして出力する 
* tag 現在の状態にタグを付ける 
* status まだ適用されていない変更セットの数を表示する 
* validate チェンジログにエラーがないかチェックする 
* changelogSyncSQL チェンジログを同期するためのSQL (databasechangelogテーブルへのクエリ) を出力する 
* listLocks データベースのチェンジログをロックしているユーザを表示する 
* releaseLocks データベースのチェンジログに対するロックを解放する 
* dropAll ユーザが保持する全てのDBオブジェクトを削除する 


##  include tag
The include tag allows you to break up your change-logs into more manageable pieces. To easily include multiple files, use the includeAll tag.

## Preconditions tag
Preconditions can be attached to change logs or changesets to control the execution of an update based on the state of the database.

## Using Contexts for Test Data
When you run the migrator though any of the available methods, you can pass in a set of contexts to run. Only changeSets marked with the passed contexts will be run.

## Best Practise


1. Liquibase attempts to execute each changeSet in a transaction that is committed at the end, 
or rolled back if there is an error. 
Some databases will auto-commit statements which interferes with this transaction setup and could lead to an unexpected database state. 
Therefore, it is usually best to have just one change per changeSet unless there is a group of non-auto-committing changes that 
you want applied as a transaction such as inserting data.


2. When Liquibase reaches a changeSet, it computes a check sum and stores it in the “databasechangelog”.   
The value of storing the check sum is so that Liquibase can know if someone changed the changes in the changeSet since it was run.   
If the changeSet was changed since it was run, Liquibase will exit the migration with an error because it cannot know what was changed and the database may be in a state different than what the changeLog is expecting.   
If there was a valid reason for the changeSet to have been changed and you want to ignore this error,   
**update the databasechangelog table so that the row with the corresponding id/author/filepath has a null value for the check sum**.  
The next time Liquibase runs, it will update the check sum value to the new correct value.


3. If you are managing your test data with Liquibase, the best way to include it is in-line with all your other changeSets, 
    but marked with a “test” context. That way, when you want your test data inserted you can run the migrator with the “test” context. 
    When it comes time to migrate your production database, don’t include the “test” context, and your test data not be included. 
    If you have multiple test environments or test data sets, simply tag them with different contexts such as “min-test”, “integration-test”, etc.

