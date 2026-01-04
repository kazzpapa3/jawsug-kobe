# 前提条件

- AWS CloudShell での実行を想定しています
- AWS Builder ID を保有していることを想定しています
- 

## AWS CloudShell での事前準備

### Kiro-CLI の存在確認

2026年1月4日 現在、AWS CloudShell では kiro-cli が導入済みのはずです。

```bash
which q
```

上記コマンドを実行した結果、以下のように返却されれば kiro-cli が利用可能です。

```bash
alias q='kiro-cli'
        /usr/local/bin/kiro-cli
```

#### Amazon Q Developer for CLI が残存している場合

`which q` を実行して `/usr/local/bin/q` のように返却された場合は、古い Amazon Q Developer for CLI が残存しています。

その場合は、以下の手順で Kiro CLI をインストールします。


<details><summary>Amazon Q Developer for CLI が残っている場合のみ実施します</summary>

##### ステップ1: 古いCLIの削除

```bash
sudo rm -f /usr/local/bin/q
```

##### ステップ2: Kiro CLIのインストール

```bash
curl -fsSL https://cli.kiro.dev/install | bash
```

**期待される出力:**

```
Kiro CLI installer:
Downloading package...
✓ Downloaded and extracted
✓ Package installed successfully
🎉 Installation complete! Happy coding!
```

##### ステップ3: PATHの更新

```bash
export PATH="$HOME/.local/bin:$PATH"
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

##### ステップ4: インストールの確認

```bash
kiro-cli --version
```

**期待される出力:**

```
kiro-cli 1.x.x
```

</details>

---

[ハンズオンの開始](01_pre.md)
