### Spring Boot の Web アプリを Eclipse で起動した時に「Cannot find template location:・・・」とコンソールに出力された場合の対処法

コンソールで次の WARN が出力されました。

Cannot find template location: classpath:/templates/ (please add some templates, check your Thymeleaf configuration, or set spring.thymeleaf.check-template-location=false)

WARN は、下図の青色ハイライトの行です。

![console1](C:\Users\Kazunari\Documents\1000Develop\Note\Git管理対象\Qiita\投稿原稿\20231215\material\console1.png)

##### WARN を消すための操作手順

※手順の（Hello）は、適宜読み替えてください。

1. （Hello）プロジェクトのプロパティーを開く。
2. ソースタブを開く。
3. （Hello）/src/main/resources の下の［除外:**］を選択する。
4. ボタン［除去］を押下する。
5. ボタン［適用］を押下する。

![projectProperty1](C:\Users\Kazunari\Documents\1000Develop\Note\Git管理対象\Qiita\投稿原稿\20231215\material\projectProperty1.png)

以上の操作手順を行うと、下図のように［除外:**］だった箇所が、［除外:なし］に変わります。

![projectProperty2](C:\Users\Kazunari\Documents\1000Develop\Note\Git管理対象\Qiita\投稿原稿\20231215\material\projectProperty2.png)

変更の確認が済んだらプロパティー画面を閉じて、再度プロジェクトを起動します。で、下図のように WARN の行が出なければ、作業は成功です。

![console2](C:\Users\Kazunari\Documents\1000Develop\Note\Git管理対象\Qiita\投稿原稿\20231215\material\console2.png)

------

投稿年月日：2023(R5).12.15

IDE：Eclipse 2022

Spring Boot Version 3.1.5