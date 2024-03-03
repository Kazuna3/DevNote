出所：[#32 Spring AOPの基礎から実装まで #Java - Qiita](https://qiita.com/Jackoguro/items/ef80abd9b952e0824aaf)

取得年月日：2024(R6).03.03

![img](https://qiita-user-profile-images.imgix.net/https%3A%2F%2Fpbs.twimg.com%2Fprofile_images%2F1477318393099603970%2F0BhOv9os_bigger.jpg?ixlib=rb-4.0.0&auto=compress%2Cformat&lossless=0&w=48&s=bc6f8b4da76f51999a60324e1d4c62a0)

@Jackoguro

# #32 Spring AOPの基礎から実装まで

- [Java](https://qiita.com/tags/java)
- [spring](https://qiita.com/tags/spring)
- [AOP](https://qiita.com/tags/aop)
- [SpringBoot](https://qiita.com/tags/springboot)

最終更新日 2022年12月24日投稿日 2022年12月21日

# #32 Spring AOPの基礎から実装まで

今回はSpringの大きな特徴の1つと言われているAOPについてまとめて行きます。

# 前提条件

この記事はSpringの最低限の知識が必要になります。
また、なるべく分かりやすく書くつもりですが、この記事の目的は自分の勉強のアウトプットであるため所々説明は省略します。

# 構築環境

1. 各バージョン
   Spring Boot ver 2.7.5
   mybatis-spring-boot-starter ver 2.2.2
   Model Mapper ver 3.1.0
   jquery ver 3.6.1
   bootstrap ver 5.2.2
   webjars-locator ver 0.46
   thymeleaf-layout-dialect ver 3.0.0
2. 依存関係
   ![依存関係.avif](C:\Users\Kazunari\Documents\1000Develop\Note\Git管理対象\Qiita\SpringAOP\#32SpringAOPの基礎から実装まで\material\依存関係.avif)

# 今回行うこと

今回は以下の流れに沿って進めていきます。

1. AOPとは

2. AOPの用語

   1. AOPの専門用語
      1. Advice
      2. Pointcut
      3. JoinPoint
   2. JoinPointの種類

3. AOPの仕組み

4. AOPの実装

   1. Pointcutの指定方法

   2. 環境設定

      1. クラスの作成
      2. pox.xmlにコードを追加

   3. Beforeの実装

      1. executionの使用方法

   4. Afterの実装

   5. Aroundの実装

      1. bean

      2. [@annotation](https://qiita.com/annotation)

      3. [@within](https://qiita.com/within)

         

## 1. AOPとは

AOPとは、共通する処理を抜き出してまとめて管理することです。
例えば、データベースアクセス処理には例外発生時の対応処理を必ず含める必要がありますが、アクセス処理が多くなると必然的に例外発生時の対応も増やさなければなりません。そのため、プログラムコードは増え、煩雑になってしまいます。
AOPを用いることで実現したい内容(中心的関心事)と付随するプログラム(横断的関心事)を分離してプログラムを作成することができます。

## 2. AOPの用語

### 1. AOPの専門用語

#### 1. Advice

AOPで実行する処理内容(横断的関心事)。ログの出力やトランザクションの制御など

#### 2. Pointcut

処理を実行する対象(クラス、メソッド)。メソッド名がgetで始める時だけ処理するなど

#### 3. JoinPoint

処理を実行するタイミング。メソッドの実行前/後など

### 2. JoinPointの種類

JointPointの種類は大きく分けて5つあります。

| 実行タイミング |                     内容                     |
| :------------: | :------------------------------------------: |
|     Before     |    対象のメソッドが実行される前に処理する    |
|     After      |    対象のメソッドが実行された後に処理する    |
| AfterReturning | 対象のメソッドが正常処理した場合のみ処理する |
|     Around     |      対象のメソッド実行の前後に処理する      |
| AfterThrowing  | 対象のメソッドが異常終了した場合のみ処理する |

## 3. AOPの仕組み

AOPの仕組みは以下のようになっています。
DIコンテナーに登録されているBean([@Controller](https://qiita.com/Controller)や[@Service](https://qiita.com/Service)などのアノテーションが付けられているクラス)メソッドをSpringが呼び出そうとします。その際にSpringはProxy経由でBeanメソッドを呼び出しています。そして、ProxyがAOPの処理(横断的関心事)とBeanのメソッド(中心的関心事)を呼び出します。

![図解１.avif](C:\Users\Kazunari\Documents\1000Develop\Note\Git管理対象\Qiita\SpringAOP\#32SpringAOPの基礎から実装まで\material\図解１.avif)
(引用画像：https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F2448205%2F9ffceb0c-6393-ad36-d074-13a776ac292e.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&w=1400&fit=max&s=bd029454c25be48915b87c9400a16c32)

## 4. AOPの実装

### 1. Pointcutの指定方法

Pointcutの指定方法は4種類あります。

|                  Pointcut                   |                             内容                             |
| :-----------------------------------------: | :----------------------------------------------------------: |
|                  execution                  |       正規表現を使って任意のクラス、メソッドを指定する       |
|                    bean                     |         DIコンテナーに登録されているBean名を指定する         |
| [@annotation](https://qiita.com/annotation) | パッケージ名を含めたアノテーション名を使って実行対処を指定する。指定したアノテーションが付いているメソッドが対象 |
|     [@within](https://qiita.com/within)     | パッケージ名を含めたアノテーション名を指定します。指定したアノテーションが付いているクラスの全てのメソッドがAOPの対象となる。 |

[@within](https://qiita.com/within)と[@annotation](https://qiita.com/annotation)の違いとして、[@annotation](https://qiita.com/annotation)は指定したアノテーションが付いている`メソッド`が対象となりますが、[@within](https://qiita.com/within)は指定したアノテーションが付いているクラス全てのメソッドがAOPの対象になります。

### 2. 環境設定

#### 1. クラスの作成

src/main/java/com/example/aspect/LogAspect.javaを作成します。
![図解２.avif](C:\Users\Kazunari\Documents\1000Develop\Note\Git管理対象\Qiita\SpringAOP\#32SpringAOPの基礎から実装まで\material\図解２.avif)

#### 2. pox.xmlにコードを追加

pom.xml

```
<!-- 省略 -->
		<!-- Spring AOP --> 
		<dependency>
			<groupId>org.springframework</groupId> 
			<artifactId>spring-aop</artifactId>
		</dependency> 
		<!-- AspectJ -->
		<dependency> 
			<groupId>org.aspectj</groupId> 
			<artifactId>aspectjweaver</artifactId>
		</dependency>
<!-- 省略 -->
```

### 3. Beforeの実装

AOPクラスを実装するにはクラスに[@Aspect](https://qiita.com/Aspect), [@Component](https://qiita.com/Component)アノテーションを付けます。

JointPointを指定するためには、メソッドにJointPointと同じ名前のアノテーションを付けます。
今回はBeforeを実装するために[@Before](https://qiita.com/Before)アノテーションをメソッドを付けます。

LogAspect.java

```
package com.example.aspect;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

import lombok.extern.slf4j.Slf4j;

@Aspect
@Component
@Slf4j
public class LogAspect {

	/**
	 * サービスの実行前にログを出力
	 * 対象:[UserService]をクラス名に含んでいる
	 */
	 @Before("execution(* *..*.*UserService.*(..))")
	 public void startLog(JoinPoint jp) {
		 log.info("メソッド開始：" + jp.getSignature());
	 }
}
```

どのクラスやメソッドをAOPの対象にするか指定するためには、[@Before](https://qiita.com/Before)などのアノテーション内に実行対象(Pointcut)を指定します。
今回はexecutionを使用しています。

```
 @Before("execution(* *..*.*UserService.*(..))")
```

#### 1. executionの使用方法

executionは以下の構文で構成されます。
`execution(戻り値 パッケージ名.クラス名.メソッド名(引数))`

- ＊(アスタリスク)：アスタリスクは任意の文字列を表します。パッケージ部分では、アスタリスクが1個のパッケージ名を表します。メソッドの引数部分では、アスタリスクが1個の引数を表します。
- ..(ドット2文字)：ドットを2個続けると、0個以上の任意の値を表します。パッケージ部分では、ドット2文字が0個以上のパッケージを表します。メソッドの引数部分では、ドット2文字が0個以上の引数を表します。
- ＋(プラス)：クラス名の後にプラスを指定すると、指定クラスのサブクラスが含まれます。

今回の場合は、
引数(*)：任意の引数 → どんな引数でも良い
パッケージ(*..*)：任意の個数、名前を持つパッケージ → どんなパッケージでも良い
クラス(.*UserService)：最後が「UserService」で終わるクラス
メソッド(.*(..))：任意の引数を持つ任意のメソッド→どんな引数を持つメソッドでも良い

### 4. Afterの実装

次にAfterを実装していきます。
[@Before](https://qiita.com/Before)→[@After](https://qiita.com/After)から変更した以外に変更点はありません。

LogAspect.java

```
package com.example.aspect;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

import lombok.extern.slf4j.Slf4j;

@Aspect
@Component
@Slf4j
public class LogAspect {

	/**
	 * サービスの実行前にログを出力
	 * 対象:[UserService]をクラス名に含んでいる
	 */
	 @After("execution(* *..*.*UserService.*(..))")
	 public void startLog(JoinPoint jp) {
		 log.info("メソッド開始：" + jp.getSignature());
	 }
}
```

実際にBefore, Afterを実装後、UserService.java(「UserService」という文字列を含んでいるクラス)の処理を実行すると以下のようにAOPが出力されていることが分かります。

```
2022-12-21 19:56:31.799  INFO 8180 --- [nio-8080-exec-1] com.example.aspect.LogAspect             : メソッド開始：List com.example.service.impl.UserServiceImpl.getAllMUser(MUser)
2022-12-21 19:56:31.807 DEBUG 8180 --- [nio-8080-exec-1] c.e.repository.UserMapper.findAllMUser   : ==>  Preparing: SELECT * FROM M_USER
2022-12-21 19:56:31.807 DEBUG 8180 --- [nio-8080-exec-1] c.e.repository.UserMapper.findAllMUser   : ==> Parameters: 
2022-12-21 19:56:31.810 DEBUG 8180 --- [nio-8080-exec-1] c.e.repository.UserMapper.findAllMUser   : <==      Total: 4
2022-12-21 19:56:31.811  INFO 8180 --- [nio-8080-exec-1] com.example.aspect.LogAspect             : メソッドの終了：List com.example.service.impl.UserServiceImpl.getAllMUser(MUser)
```

### 5. Aroundの実装

最後にAroundの実装を行います。
[@Around](https://qiita.com/Around)アノテーションをメソッドを付けることで、AOP実行対象の前後に処理を入れることができます。

LogAspect.java

```
package com.example.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

import lombok.extern.slf4j.Slf4j;

@Aspect
@Component
@Slf4j
public class LogAspect {

	 /** コントローラーの実行前後にログを出力する */
	 @Around("@within(org.springframework.stereotype.Controller)")
	 public Object startLog(ProceedingJoinPoint jp) throws Throwable {
		 
		 // 開始ログの出力
		 log.info("メソッドの開始：" + jp.getSignature());
		 
		 try {
			 //メソッドの実行
			 Object result = jp.proceed();
			 
			 // メソッドの実行
			 log.info("メソッドの終了：" + jp.getSignature());
			 
			 // 実行結果を呼び出し元に返却
			 return result;
			 
		 } catch(Exception e) {
			 // エラーログ出力
			 log.error("メソッド異常終了：" + jp.getSignature());

			 
			 // エラーの再スロー
			 throw e;
		 }
	 }
}
```

メソッドを正常に実装するためには以下の処理を加える必要があります。

```
//メソッドの実行
Object result = jp.proceed();

// 実行結果を呼び出し元に返却
return result;
```

#### 1. bean

Pointcut(実行対象)に`bean`を使用すると、DIコンテナーに登録されているBean名でAOPの対象を指定できます。
以下の場合、Bean名の最後に「Controller」が付いているクラスをAOPの対象にしています。

```
@Around("bean(*Controller)")
```

#### 2. [@annotation](https://qiita.com/annotation)

Pointcut(実行対象)に`@annotation`を使用すると、指定したアノテーションが付いているメソッドがAOPの対象になります。
なお、指定するアノテーションはパッケージ名を含めます。
以下の場合、[@GetMapping](https://qiita.com/GetMapping)が付いているメソッドをAOPの対象にしています。

```
@Around("@annotation(org.springframework.web.bind.annotation.GetMapping)")
```

#### 3. [@within](https://qiita.com/within)

Pointcut(実行対象)に`@within`を使用すると、指定したアノテーションが付いているクラスの全てのメソッドがAOPの対象となります。
なお、指定するアノテーションはパッケージ名を含めます。
以下の場合、[@Controller](https://qiita.com/Controller)アノテーションが付けられているクラスの全メソッドをAOPの対象にしています。

```
@within(org.springframework.stereotype.Controller)
```

正常終了した場合は以下のようなログが出力されます

```
2022-12-21 20:26:21.476  INFO 8180 --- [nio-8080-exec-6] com.example.aspect.LogAspect             : メソッドの開始：String com.example.controller.UserListController.getUserList(UserListForm,Model)
2022-12-21 20:26:21.482 DEBUG 8180 --- [nio-8080-exec-6] c.e.repository.UserMapper.findAllMUser   : ==>  Preparing: SELECT * FROM M_USER
2022-12-21 20:26:21.482 DEBUG 8180 --- [nio-8080-exec-6] c.e.repository.UserMapper.findAllMUser   : ==> Parameters: 
2022-12-21 20:26:21.485 DEBUG 8180 --- [nio-8080-exec-6] c.e.repository.UserMapper.findAllMUser   : <==      Total: 4
2022-12-21 20:26:21.485  INFO 8180 --- [nio-8080-exec-6] com.example.aspect.LogAspect             : メソッドの終了：String com.example.controller.UserListController.getUserList(UserListForm,Model)
```

異常終了した場合は以下のようなログが出力されます

```
2022-12-21 20:28:29.437  INFO 8180 --- [nio-8080-exec-7] com.example.aspect.LogAspect             : メソッドの開始：String com.example.controller.UserListController.getUserList(UserListForm,Model)
2022-12-21 20:28:29.447 DEBUG 8180 --- [nio-8080-exec-7] c.e.repository.UserMapper.findAllMUser   : ==>  Preparing: SELECT * FROM M_USER
2022-12-21 20:28:29.447 DEBUG 8180 --- [nio-8080-exec-7] c.e.repository.UserMapper.findAllMUser   : ==> Parameters: 
2022-12-21 20:28:29.450 DEBUG 8180 --- [nio-8080-exec-7] c.e.repository.UserMapper.findAllMUser   : <==      Total: 4
2022-12-21 20:28:29.451 ERROR 8180 --- [nio-8080-exec-7] com.example.aspect.LogAspect             : メソッド異常終了：String com.example.controller.UserListController.getUserList(UserListForm,Model)
```

# 最後に

今回はAOPの使用方法をまとめました。
実務では正しく出力されているか確かめることが非常に重要なので皆さんもAOPを学んでみてください