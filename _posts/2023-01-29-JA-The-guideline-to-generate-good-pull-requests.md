# 8. ナマケモノの為のPull Request作成ガイドライン

## このドキュメントを作成した経緯

ナマケモノである私にはPull Requestのレビューは骨が折れるものです。
コードだけでなく実装の経緯や既存コードへの影響、動作確認などチェックしないといけないことが多いので。

そんな中、最近チームのメンバ－のPull Request（以下PR）のレビューでフラストレーションが溜まることがありました。
では自分は他のメンバーが快適にレビューできるようなPRが作成できているかと言われると必ずしもそうでないです。
前職の上司に「PRこそエンジニアの成果であり、優れたPRこそ正義！」と言われていたが、今更になってその言葉が心にしみています。

ということで、先ず自分がPRの改善を実践し周りに良い影響を与えたいと考え、今回ガイドラインを作ってみました。
これは取り敢えずは自分用のガイドラインである事を理解していただきたいです。
勿論、良いと思うものは実践していただいて、可能であればフィードバックをいただければと思います。

## 基本的な考え方

PRを作るとき、コードのDiff（ `Files Changed` ）が重視されがちだと思います。
確かにコーディングこそエンジニアの腕の見せ所だし、リポジトリに変更をもたらしてくれるものなので重要なのは間違いないです。
ただここに書かれている内容はコーディングの改善ではなく、それ以外で部分の改善についてです。
コーディング以外の部分の改善でレビュアーの負担を減らし業務効率化、そしてバグを減らすことができればと考えています。

## Pull Requestを改善する目的

Pull Requestを作成すること自体の目的は、リポジトリに加える変更を他者に確認してもらいメインブランチに反映させる許可をもらうことだと考えています。
また、過去にどのような修正が行われたか確認するドキュメントの役割も果たしていると考えています。

では何故改善するのか。
多くの人に同意していただけると思うのですが、PRのレビューはとても大変な作業だと思っています。

> First, let’s admit it: reviewing pull requests is really hard.

参考：[The (written) unwritten guide to pull requests - Work Life by Atlassian](https://www.atlassian.com/blog/git/written-unwritten-guide-pull-requests)

そのためPRの改善によって `レビュアーの負担軽減` と `適切なフィードバックを貰えること` 、
そして `数か月後の自分を含むすべての人が見返してもどのような修正が行われたか理解できるようにすること` が重要であると考えています。

## やること

### 1. 最初に空のコミットをプッシュしてPRを作成し説明を記入する

作業ブランチを切った後に空コミットを積んでプッシュすることで作業ブランチのPRを作成することができるようになります。
PRを先に作成し適切な説明を記述することで、実装者が何をしようとしているのかが他の人にもわかるようになります。
また、小まめにプッシュすることで実装者が途中で別の作業をして作業ができなくなった場合でも進捗状況がわかるようになります。

```
$ git checkout -b target_branch
$ git commit --allow-empty --m "Initial commit"
$ git push -u origin target_branch
```

### 2. merge masterや rebase masterでブランチを最新の状態に保つ

作業ブランチを作成してコミットを積んでいる間に、マージ先のブランチで新しい修正が入っている可能性は大いにあります。
この時マージ先の変更を小まめに取り込んでいないと、レビューでLGTMをもらった後にコンフリクト解消で修正が入り再度レビューが必要になってしまうことがあります。
勿論上記パターンは避けられない場合もありますがレビュアーの時間を余計に取らないように気を付ける必要はあると思います。

マージ先ブランチの変更を作業ブランチに取り込む方法には主に `merge` と `rebase` があります。
`merge` はGitHubの場合はPR画面上から行うことができるのでハードルは低いですが、余計なマージコミットが積まれ履歴が複雑になってしまいます。 
一方 `rebase` は奇麗な履歴を保てますがコミットの再書き込みが行われてしまいます。
作業ブランチの場合大抵は `rebase` で変更を取り込むほうがコストは低いと思います（私は `rebase` 派です）が、双方のメリットとデメリットを理解し適切に使い分けることが大切かと思います。

＊例）rebaseの手順

```
# How to rebase
$ git checkout main
$ git fetch
$ git reset --hard origin/main
$ git checkout target_branch
$ git rebase main
```

参考：[マージとリベース  Atlassian Git Tutorial](https://www.atlassian.com/ja/git/tutorials/merging-vs-rebasing)

### 3. 作業ブランチのコミット履歴を整える

こちらの項目の内容に関して、自分は今まで全く実践していませんでした。
ただ、今回の記事を作成するための調べもをしている最中に発見したMoneyforwardの開発ブログが素晴らしいと思い追加しました。
コミットの履歴を整える目的は、どの修正がどのコミット混ざっているか分かるようにすることです。
それによりレビュアーの負担を減らすだけでなく、過去の修正を遡る際に的確に探しているコミットを見つけられるようになると考えています。

記事にあるように、以下を実践しコミット履歴を整えるといいと思いました。

1. 同じ機能のコミットは一つのコミットに統合して冗長性を無くす
2. 分けられる機能が混ざっているコミットは分割して、それぞれの機能ごとの個別コミットを作る
3. コミットを議論性の低い順番に並び替える

参考：[レビューしやすいコミット履歴でバグ削減](https://moneyforward-dev.jp/entry/2015/11/30/reviewable-commit-log/)

### 4. 適切なコミットメッセージを残す

せっかく上記のコミット履歴を整えるを実践しても、コミットメッセージが適切でなければ効果が薄れてしまいます。
常に適切なコミットメッセージを残していればレビュアーも履歴を追いやすいし、履歴を整える際も役に立つと思うので効果は高いと思います。
コミットメッセージの残し方に関しては以下の記事が簡潔で良いと思いました。

参考：[僕が考える最強のコミットメッセージの書き方 - Qiita](https://qiita.com/konatsu_p/items/dfe199ebe3a7d2010b3e)

> Have on-point commit messages Good commit messages can also provide a nice bullet-point-like summary of the code changes as well, and it helps reviewers who read the commits in addition to the diff.

参考：[The (written) unwritten guide to pull requests - Work Life by Atlassian](https://www.atlassian.com/blog/git/written-unwritten-guide-pull-requests)

### 5. 一つのPRの修正範囲が大きくならないようにする

必要最低限を備えた小さなPRを心がけることも重要です。
一度に200~400の行数のコードを60~90分にわたってレビューすることで70~90％の欠陥を発見できる、という研究結果もあるようです。

参考：[Best Practices for Code Review](https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/)

ただ、レビュアーの負担やバグの見落としを減らす事を考えると、PRのLOC（変更コードの行数）は少なければ少ないほど良いと思います。
勿論大きなロジックを実装する場合、どうしてもPRが大きくなってしまう場合もあると思います。
その際はレイヤー毎に切り分けてPRに依存関係を記載するのが良いかもしれません。

例えばRailsで新規にCRUDの実装を行う場合、以下のようにより小さなタスクへ切り分けることができます。

タスク
- 新規CRUD実装

切り分けた後のタスク
- データベースの修正
- モデルの追加
- ルーティングの設定
- コントローラーとビューの実装（それぞれ分離させる、若しくはアクション毎に実装する）
- テストの実装 - etc...

### 6. コーディングを行い、動作確認が完了した段階でPRのWIPを外す

こちらは運用上のルールになりますが、実装と動作確認が完了してからＷＩＰを外しましょう。
例えば、動作確認の完了を待たずにＷＩＰを外すとレビュアーは全ての作業が完了したと勘違いしコードレビューを始めてしまうかもしれません。
そのコードは動かないのに。。。
もし動作確認により先行してレビューをして欲しい場合は、ＷＩＰを付けたまま事情説明を添えてレビュー依頼をするなど工夫が必要です。

### 7. PRの説明をしっかり書く

PRの説明に気を配ることは、PRを他者に快適にレビューしてもらう上で重要な要素だと思います。
また、過去の修正を遡る際にどのような修正が行われたか、コードそのものを確認せずとも理解できるようになります。

具体的に説明欄に何を書けばよいかは以下の記事がとても参考になりました。

参考：[【GitHub】プルリクエストの書き方とテンプレート](https://applis.io/posts/how-to-write-git-pull-request#%E3%83%97%E3%83%AB%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%81%AE%E6%9B%B8%E3%81%8D%E6%96%B9)

|順番|見出し|備考|
|----|------|----|
|1|変更の概要|概要、関連するIssueやプルリクエスト|
|2|なぜこの変更をするのか|
|3|やったこと|チェックボックスで進捗を表す|
|4|変更内容|UIのスクリーンショット、APIのリクエスト/レスポンスなど|
|5|やらないこと|プルリクエストのスコープ外とすること|
|6|影響範囲|ユーザーやメンバー、システムに影響すること|
|7|どうやるのか|変更したものの使い方や再現手順|
|8|課題|悩んでいるところ、とくにレビューしてほしいところ|
|9|備考||

記事にもある通り、本文は長ければよいというものではないと思います。
シンプルであることを心掛け必要だと思う項目を記載することが重要です。

また、上記テーブルはブログから抜粋しておりますが、4のスクリーンショットに関しては[Github プルリクの添付画像はprivateでも認証なしで誰でも参照可能](https://ksakae1216.com/entry/2022/01/17/073000)なので注意が必要です。


### 8. コードの説明をPRのコメントで残す

PRの説明をしっかり書けば、そのPRの概要は概ね理解できるようになります。
その上で実装者がコードにコメントを残すことでレビュアーがどの部分を注意して確認すればよいか分かるようになり負担を減らすことができます。
変更の肝となる部分や止むを得ず可読性が低くなっている部分に対してコメントで説明があるとレビューのハードルはかなり低くなるかと思います。

## 最後に

最後まで読んでくださった方、ありがとうございます！
色々詰め込んだらかなりの量になってしまいました。
大変そうですが実践あるのみ！チームに良い影響を与えられるといいな。。。
（ちなみに私は最近あまり業務でコードを書いていませんｗｗｗ）
記事の内容に関して何かご意見やご感想、若しくは実践してみてフィードバックなどあればコメントしていただければと思います！

