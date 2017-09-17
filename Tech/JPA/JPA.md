#Spring DATA JPA

Spring Data JPAを利用するには、プロジェクトにいくつか用意しておかなければいけないものがあります。  
その１つが、「persistence.xml」と呼ばれるファイルです。これは、「パーシスタンスユニット」というものの情報を記述したファイルです。  
JPAでは、データベースアクセスを行うのに「パーシスタンス」というものを用意します。要するに、このpersistence.xmlの中に、アクセスするデータベースに関する情報を記述しておく、と考えて下さい。 

## query-lookup-strategyについて。
Spring Data は リポジトリに定義されたメソッドから自動的にクエリを生成することができますが、その戦略です。設定は3つあります。

* CREATE
* USE_DECLARED_QUERY
* CREATE_IF_NOT_FOUND
CREATEはメソッドからクエリを生成します。USE_DECLARED_QUERYはユーザ定義されたクエリを使用します。これは見つからなかったらエラーになります。

CREATE_IF_NOT_FOUNDはこの両方を合わせたやつで、もしユーザ定義のクエリが見つからない場合は、メソッドからクエリを生成します。これがdefaultです。


Spring Data JPA はメソッドからクエリを自動生成しますが、そのメソッド名には命名規則があります。

まず戻り値はRepositryに指定している総称型のエンティティのListか、もしくは、そのエンティティにします。
Listの場合はJPAのgetResultListが、エンティティの場合はgetSingleResultが呼ばれるという感じですかね？中身見てないのでわかりませんが。定義はこんなんです。
```
    Emp findByName(String name);
    List<Emp> findByDept(Dept dept);
```
次にメソッド名のprefixは、findBy、readBy、getByが使用出来ます。単にfind、read、getとか使えるような記述がマニュアルに書いてあったのですが、エラーになっちゃいました。ようわからん。次のメソッドはすべて同じ意味です。
```
    List<Emp> findByDept(Dept dept);
    List<Emp> readByDept(Dept dept);
    List<Emp> getByDept(Dept dept);
    List<Emp> dept(Dept dept);
```

よくわからんのですがプレフィクスはなくてもいいみたいです。いろいろ使えるみたいですが、
**findByが個人的には一般的な気がするので、他のはわりとどうでもいいやという**。

プレフィックに続いてエンティティのプロパティを指定してい行きます。
その指定に合わせて、引数でそのプロパティを受け取るようにします。
複数の条件を組み合わせる場合はAnd、Orで組み合わせられます。その他にもプロパティの後にキーワードを設定することで条件を設定できます(これはJPA限定みたいですけど)。

* And	findByLastnameAndFirstname	… where x.lastname = ?1 and x.firstname = ?2
* Or	findByLastnameOrFirstname	… where x.lastname = ?1 or x.firstname = ?2
* Between	findByStartDateBetween	… where x.startDate between 1? and ?2
* LessThan	findByAgeLessThan	… where x.age < ?1
* GreaterThan	findByAgeGreaterThan	… where x.age > ?1
* IsNull	findByAgeIsNull	… where x.age is null
* IsNotNull,NotNull	findByAge(Is)NotNull	… where x.age not null
* Like	findByFirstnameLike	… where x.firstname like ?1
* NotLike	findByFirstnameNotLike	… where x.firstname not like ?1
* OrderBy	findByAgeOrderByLastnameDesc	… where x.age = ?1 order by x.lastname desc
* Not	findByLastnameNot	… where x.lastname <> ?1
* In	findByAgeIn(Collection ages)	… where x.age in ?1
* NotIn	findByAgeNotIn(Collection age)	… where x.age not in ?1

例えばこんな感じで定義できます。
```java
List<Emp> findByDeptAndIdGreaterThan(Dept dept, Long id);
```

基本はメソッド名で指定したプロパティを引数に取るようにしますが、特別な引数があります。それが、PageableとSortです。
```java
    Page<Emp> findByDeptAndIdGreaterThan(Dept dept, Long id, Pageable pageable);
    List<Emp> findByDeptAndIdGreaterThan(Dept dept, Long id, Sort sort);
```
Pageableを使用する場合は戻り値の型にPageを指定できますが、Listでも問題ありません。Pageableを使用する場合は必ずentityManagerFactoryのBeanの定義の中にjpaVendorAdapterを定義しないと動きません(ちょっとはまった)。
```xml
<bean class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean" id="entityManagerFactory">
    <property name="persistenceUnitName" value="persistenceUnit" />
    <property name="dataSource" ref="dataSource" />
    <property name="jpaVendorAdapter">
      <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter" />
    </property>
  </bean>
```

あとProperty expressionsというのもあります。これは関連エンティティをたぐってプロパティを指定するような場合に使用出来ます。例えば
```
List<Emp> findByDeptNameLike(String name);
```
これはx.dept.nameのプロパティを条件にしています。引数もこの場合はDeptの型ではなく、その関連をたぐったプロパティの型になります。DeptのnameはStringなので、ここではStringを指定しています。

場合によっては、このProperty expressionsはうまく行かない時があります。プロパティの名前がかぶっているようなときです。そんなときは**アンスコ**をプロパティの区切りに指定できます。
```
List<Emp> findByDept_NameLike(String name);
```
これは先程の定義とまったく同じ意味です。

[サンプル](https://github.com/yamkazu/springdata-jpa-example/tree/querycreation)


## 名前付きクエリの話。
JPAはもともとこの名前付きクエリをサポートしています。エンティティにつけるアレです。
```java
@Entity
@NamedQuery(name = "Emp.findByUseNamedQuery", query = "select e from Emp e where e.id > ?1")
public class Emp {
// ..
```
これをSpring Data JPAのリポジトリで利用するには、nameの指定にルールがあり、ドメインクラス名(エンティティ).メソッド名となるようにします。

この定義したクエリをリポジトリのメソッドで使うには以下のように、nameで指定したメソッド名で定義します。
```java
List<Emp> findByUseNamedQuery(Long id);
```

これでfindByUseNamedQueryを実行すると、定義されているクエリが使用されます。

## @Query
これ以外にもSpring Data JPA独自の @Query というアノテーションを使用出来ます。
通常JPAでの名前付きクエリはエンティティクラス、またはXMLに定義します(XMLとかあまり使わないけど)が、
@Query は リポジトリのメソッドに付与してクエリを定義できるものです。
```java
@Query("select e from Emp e where e.id > ?1")
List<Emp> findByUseQueryAnnotation(Long id);
```
上記の様に、クエリ内の変数は、メソッドの引数の順番で処理することができますが、この方法はリファクタリング等、変更に非常に弱いので、**名前付きでの指定**も可能になっています。

こんなん
```java
@Query("select e from Emp e where e.id > :id")
List<Emp> findByUseQueryAnnotationWithParam(@Param("id") Long id);
```
個々まで見てきたのでは、selectのみですが、updateもできます。

```java
@Query("update Emp e set e.name = :name where e.id = :id")
@Modifying
//    @Modifying(clearAutomatically = false)
int setName(@Param("id") Long id, @Param("name") String name);
```

メソッドの戻り値定義と@Modifyingが必要です。@Modifyingが指定されているとEntityManager#clearが自動的に呼びされますが、呼び出したくない場合はclearAutomatically = falseを指定してください。

@Modifyingが正直よくわからなくてJPAのexecuteUpdate()になるやつで必要なのかなぁーと思ったのですが

```java
@Query("delete from Emp e where e.id = :id")
int deleteById(@Param("id") Long id);
```
deleteは指定しなくても動きました。中身は覗いていない。


## Specificationの話です。

SpecificationはDDDのパターンの一つですが、JPA2から導入されたCriteriaを利用して、Spring Data JPAではSpecificationパターンみたいなことが出来ます。

Specificationを使用するにはリポジトリの定義でJpaSpecificationExecutorを継承する必要があります。
```java
public interface EmpRepository extends JpaRepository<Emp, Long>, JpaSpecificationExecutor<Emp> {
//..
```
JpaSpecificationExecutorに定義されているメソッドは以下のようなもの。
```java
T findOne(Specification<T> spec);
List<T> findAll(Specification<T> spec);
Page<T> findAll(Specification<T> spec, Pageable pageable);
List<T> findAll(Specification<T> spec, Sort sort);
long count(Specification<T> spec);
```
要はSpecificationを生成して引数として渡します。中身はCriteriaで条件を指定していきます。
こんなん
```java
    public class Specifications {
        public static Specification<Emp> idLessThanOrEqualTo(final Long id) {
            return new Specification<Emp>() {
                @Override
                public Predicate toPredicate(Root<Emp> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                    return cb.lessThanOrEqualTo(root.get(Emp_.id), id);
                }
            };
        }
    }
```

使うときは
```java
    @Test
    public void Specificationを使ってみる() throws Exception {
        List<Emp> emps = repository.findAll(idLessThanOrEqualTo(4L));
        assertThat(emps.size(), is(not(equalTo(0))));
    }
```
昨日のSpecificationの続きでSpecificationを複数組み合わせて使う際に便利な、Specificationsというヘルパークラスが用意されている。
```java
public class Specifications<T> implements Specification<T> {
  private final Specification<T> spec;
  private Specifications(Specification<T> spec) {...}
  public static <T> Specifications<T> where(Specification<T> spec) {...}
  public Specifications<T> and(final Specification<T> other) {...}
  public Specifications<T> or(final Specification<T> other) {...}
  public static <T> Specifications<T> not(final Specification<T> spec) {...}
  public Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder builder) {...}
}
```
幾つかメソッドが定義されていますがwhereを基点としてandとかorとかでつなげていく感じ。

例えば２つSpecがあるとして
```java
public class EmpSpecifications {
    public static Specification<Emp> idLessThanOrEqualTo(final Long id) {
        return new Specification<Emp>() {
            @Override
            public Predicate toPredicate(Root<Emp> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                return cb.lessThanOrEqualTo(root.get(Emp_.id), id);
            }
        };
    }

    public static Specification<Emp> hasDept(final Dept dept) {
        return new Specification<Emp>() {
            @Override
            public Predicate toPredicate(Root<Emp> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                return cb.equal(root.get(Emp_.dept), dept);
            }
        };
    }
}
```
これを組み合わせて使う場合は
```java
List<Emp> emps = repository.findAll(where(idLessThanOrEqualTo(9L)).and(hasDept(Dept.of(1L))));
```
とかいうふうに使えます。

## 结合
### 1:1
* エンティティ間の多重度が１対１で、関連の向きが単方向の場合は、特に何も指定することなくマッピングすることができる。
* ただし、結合に用いられるカラム名が、デフォルトでは フィールド名_相手のキーカラム名 で解決される（okazakiYumemi_id）ので、 @JoinColumn アノテーションを使って結合に用いるカラム名を明示している。
* デフォルトのフェッチ方法は EAGER になる。これを変更したい場合は、 @OneToOne(fetch = FetchType.LAZY) の指定を追加する。



## 参考网站
http://qiita.com/opengl-8080/items/265f9f66a65e966678cb