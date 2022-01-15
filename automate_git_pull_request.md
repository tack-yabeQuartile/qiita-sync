<!--
title:   Pull Requestの作成を自動化したい
tags:    shell,github,自動化,gh
private: false
-->
# 背景
チーム開発で有用なPR機能だが、定型作業なのに手動で進めるのは地味に時間の浪費となっている。
開発がひと段落ついたらチャチャっとPR作成してしまいたい

# 利用する便利ツール
githubのコマンドラインインターフェース:[gh](https://cli.github.com/)


# 準備
* ghコマンドの有効化→[参考(Qiita)](https://qiita.com/KEINOS/items/d5da8c154cfb97336e3c#tl-dr-%E4%BB%8A%E5%8C%97%E7%94%A3%E6%A5%AD)

# 自動でPull Requestを立てる
<b>前提</b>
* ghコマンドを使用できる状態
* 開発はマージ先と別のブランチで行われている
* 開発が終わってコミットまで済んでいる

パラメータを対話型で入力する場合
``` bash
#!/bin/bash

# ログイン状態のチェック
gh auth status 2> temp_github_auth
check_auth=`wc -l temp_github_auth |awk '{print $1}'`
rm temp_github_auth

if [ $check_auth = "1" ];then
    trap "stty echo; exit 1" 2
    stty -echo
    echo "Insert your github Token. (Now you are not logged in to github.)"
    read -p "Github Token: " pass
    echo $pass > tmp_token_github
    gh auth login -h GitHub.com --with-token < tmp_token_github
    rm tmp_token_github
    stty echo
fi

# create Pull Request
echo "Set reviewer?"
read reviewers
echo "which branch do you merge into?"
read target_branch
echo "input description if you have"
read body
echo "input the title of your PR"
read pr_title
gh pr create -r $reviewers -B $target_branch -b $body -t "${pr_title}"
```

このスクリプトを実行すると、ghコマンドの機能として対象のリポジトリに直接操作あるいはforkして操作などの選択肢が出る場合があります。
実際にPRを作成するときは、「続いて何をしますか？」という質問に"submit"(提出)を回答すると実際にPRが作成されます。

このスクリプトですが後半部分をハードコーディングした形にすると対話の手間を減らすこともできます。
``` bash
# create Pull Request
reviewers="hoge,fuga,hogehoge"
target_branch="xxxxxxx"
body="This PR aims ......"
pr_title="fix bug"
gh pr create -r $reviewers -B $target_branch -b $body -t "${pr_title}"
```
パラメタを別ファイルから読み出すのもいいかもしれません。

# 最後に
地味な自動化ですが、チリも積もればなんとやら。みんなで何回も使うものを自動化する価値は高いです。
他にもghコマンドを使ってマージやラベル付与も自動化できるのでエンジニアが開発とレビューに専念出来る環境作りに有用です
