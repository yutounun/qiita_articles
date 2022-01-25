# 経緯
vscodeでgitをガッツリ使っていて、色々インストールしたりした結果、変更ファイルが数千個となった。

そこでこれらのファイルを一括で削除しようとしたのだが、これが何度やっても完了しない。

写真

なかなか記事が見つからず途方に暮れていたが、普通にgithub issue見るとあったので他に困った人ように執筆する。
# 解決策
GITのバージョンを3.35(latest)に変更した。
# 環境
git: 2.25
vscode: 1.61.1
# 具体的に
現在利用しているgitのバージョンを確認する。

[gitのissue](https://github.com/microsoft/vscode/issues/91080)によるとgit 2.25の場合は変更ファイルが削除できない問題が起きるようだ。

```
$ git --version
```

```
$ git version 2.25.0
```

```
$ brew unlink git
```

```
$ brew install git
```

# まとめ
日本語で検索しても全く記事が見つからず途方に暮れて英語で検索すると一瞬でgithubのissueにたどり着けたのでおすすめ。


# 参考

https://github.com/microsoft/vscode/issues/91080#issuecomment-593824961

