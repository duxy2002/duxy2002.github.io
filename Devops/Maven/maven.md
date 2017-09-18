# maven-enforcer-plugin
https://maven.apache.org/enforcer/maven-enforcer-plugin/usage.html

The Enforcer plugin provides goals to control certain environmental constraints such as Maven version, JDK version and OS family along with many more built-in rules and user created rules.

The enforcer:enforce mojo
This goal is meant to be bound to a lifecycle phase and configured in your pom.xml. The enforcers execute the configured rules to check for certain constraints. The available built-in rules are described here. Besides the rules to execute, these goals support three options:

skip - a quick way to skip checks via a profile or using -Denforcer.skip=true from the command line.
fail - if the goal should fail the build when a rule fails. The default is true. If false, the errors will be logged as warnings.
failFast - if the goal should stop checking after the first failure. The default is false.
Each rule to be executed should be added to the rules element along with the specific configuration for that rule.

As of version 1.4, you may add a level element to the rules. Valid values are WARN and ERROR. When level WARN is specified, the rule will only spit out a warning but will not fail the build.

Sample Plugin Configuration:

````xml
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-enforcer-plugin</artifactId>
        <version>3.0.0-M1</version>
        <executions>
          <execution>
            <id>enforce-versions</id>
            <goals>
              <goal>enforce</goal>
            </goals>
            <configuration>
              <rules>
                <bannedPlugins>
                  <!-- will only display a warning but does not fail the build. -->
                  <level>WARN</level>
                  <excludes>
                    <exclude>org.apache.maven.plugins:maven-verifier-plugin</exclude>
                  </excludes>
                  <message>Please consider using the maven-invoker-plugin (http://maven.apache.org/plugins/maven-invoker-plugin/)!</message>
                </bannedPlugins>
                <requireMavenVersion>
                  <version>2.0.6</version>
                </requireMavenVersion>
                <requireJavaVersion>
                  <version>1.5</version>
                </requireJavaVersion>
                <requireOS>
                  <family>unix</family>
                </requireOS>
              </rules>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
````

# maven-resources-plugin
http://qiita.com/kozy4324/items/9fa17a98203761012fd9

pom.xml内で${...}と記述することで変数置換ができることと同様に、maven-resources-pluginのフィルタリングを利用することで設定ファイル(e.g., system.properties)内でも変数置換が行える。

## よくありそうなユースケース

* pom.xml内のproject.versionをプロパティファイル経由でプログラムが利用する
* 設定ファイルの各種パスでproject.basedirを利用してテスト実行などではプロジェクト内で入出力を完結させる
* etc...

## そもそもmaven-resources-pluginって何ですか？

ソース以外の設定ファイルリソースなどをコピーするためのプラグイン。
http://maven.apache.org/plugins/maven-resources-plugin/

## 基本的な挙動
このプラグインが利用する項目の一部を抜粋。
````xml
<build>
  <outputDirectory>${project.build.directory}/classes</outputDirectory>
  <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
  <resources>
    <resource>
      <directory>${project.basedir}/src/main/resources</directory>
    </resource>
  </resources>
  <testResources>
    <testResource>
      <directory>${project.basedir}/src/test/resources</directory>
    </testResource>
  </testResources>
</build>
````
resources:resourcesゴールでresourcesで指定しているディレクトリ以下リソースをoutputDirectoryへコピーする
resources:testResourcesゴールでtestResourcesで指定しているディレクトリ以下リソースをtestOutputDirectoryへコピーする
単純に言えばコピーしているだけですね。

特定ファイルのみコピー/除外

例えば拡張子が.txtと.rtfのみコピーする場合はincludesでパターンを指定。
````xml
<resources>
  <resource>
    <directory>src/my-resources</directory>
    <includes>
      <include>**/*.txt</include>
      <include>**/*.rtf</include>
    </includes>
  </resource>
</resources>
````
逆に除外する場合はexcludesでパターンを指定する。
````xml
<resources>
  <resource>
    <directory>src/my-resources</directory>
    <excludes>
      <exclude>**/*.bmp</exclude>
      <exclude>**/*.jpg</exclude>
      <exclude>**/*.jpeg</exclude>
      <exclude>**/*.gif</exclude>
    </excludes>
  </resource>
</resources>
````

## 本題：フィルタリング
フィルタリングのExamplesはこれ。
http://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html

filteringにtrueを設定するとコピー時にフィルタリングが有効になる。

````xml
<resources>
  <resource>
    <directory>src/main/resources</directory>
    <filtering>true</filtering>
  </resource>
</resources>
````
system.propertiesにこう書き、
````properties
version=${project.version}
````
pom.xml内で<version>1</version>となっているならば、
````properties
version=1
````
のように変換されて出力される。使える変数はpom.xml内と同様（のはず、明示的な言及は未確認...）。

${project.basedir}などももちろん利用できる。例えばログ出力パスが設定にあり、ローカル実行時は出力をoutputDirectory以下としたければ、

log4j.appender.C.File=${project.build.outputDirectory}/temp/a.log
などと書いておけばいいんじゃないでしょうか。

# Maven Surefire Plugin
http://maven.apache.org/surefire/maven-surefire-plugin/

The Surefire Plugin is used during the test phase of the build lifecycle to execute the unit tests of an application. It generates reports in two different file formats:
* Plain text files (*.txt)
* XML files (*.xml)
By default, these files are generated in ${basedir}/target/surefire-reports/TEST-*.xml.

# jacoco-maven-plugin
http://qiita.com/umezo@github/items/e750d8e94663f36d9500

## 前提
### javaのバージョン

> java version "1.8.0_74"
> Java(TM) SE Runtime Environment (build 1.8.0_74-b02)
> Java HotSpot(TM) 64-Bit Server VM (build 25.74-b02, mixed mode)
### mavenのバージョン

> Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T01:41:47+09:00)
> Maven home: /usr/local/apache-maven-3.3.9
> Java version: 1.8.0_74, vendor: Oracle Corporation
> Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_74.jdk/Contents/Home/jre
> Default locale: ja_JP, platform encoding: UTF-8
> OS name: "mac os x", version: "10.10.3", arch: "x86_64", family: "mac"
CIサーバ上でも同様だったのでOSは関係ないだろう。

## 関連技術
* jacoco
  * コードカバレッジを計測してくれるライブラリ
  * maven用のプラグインも提供している
  * 本記事ではこのmaven pluginを使う
## jacocoの導入
jacocoのusageを読もう。
サンプルのpom.xmlもある。
Qiitaにいろんな記事があるので、タグから探るのもいいでしょう。

コレでうまく行った人はすみやかにこの記事を閉じましょう。

ちなみに自分は以下の様なエラーから無限に抜け出せなかった。


$ mvn jacoco:report
[ERROR] No plugin found for prefix 'jacoco' in the current project and in the plugin groups

ちょくせつmvnのコマンドライン引数でゴールを指定する
最終的にgithubでコード検索したところ。かなりのリポジトリでpom.xmlに書いてないということがわかった。

何の事はないしかるべきタイミングでorg.jacoco:jacoco-maven-pluginというprefixをつかって指定すればいいのだ。
つまりこんな感じ

mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent test
mvn org.jacoco:jacoco-maven-plugin:report
まとめ
素直にやってダメなら、org.jacoco:jacoco-maven-pluginをコマンドライン引数で指定する。

# sonar-maven-plugin
https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Maven

# frontend-maven-plugin
https://github.com/eirslett/frontend-maven-plugin#installing-node-and-yarn
