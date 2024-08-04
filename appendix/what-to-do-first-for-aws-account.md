# AWS アカウント開設後に最初に行うべきこと

## 1. root アカウントへの MFA デバイスの設定

ルートユーザーの E メールアドレスとパスワードを設定した「root アカウント」は、権限が強大であることから通常時の利用が推奨されていません。

そのため、通常時の利用には、別途、IAM ユーザーを作成・利用する（あるいは IAM Identity Center を利用する）ことが望ましいです。

その上で、パスワードの漏えいなどによる不正利用を避けるために、多要素認証の設定を行い管理者本人以外の「root アカウント」による AWS マネジメントコンソールへのアクセスを防ぎます。

### 必要なもの

以下いずれかを準備しておく必要がある。

- ハードウェア TOTP トークン
- Google Authenticator、Duo Mobile、Authy などの認証アプリケーション（対応しているアプリケーションであればいずれか一つで構いません）
- パスキーまたはセキュリティキー（Macbook などの TouchID や、Windows Hello などのパスキーや FIDO2 セキュリティキー）

1. 「root アカウント」で AWS マネジメントコンソールへログインします。
2. AWS マネジメントコンソールログイン後の右上の「アカウント名 と下向き三角」が表示されている部分をクリックします。
3. 展開したメニューの中から「セキュリティ認証情報」をクリックします。
4. 「MFA が割り当てられていません」と表示されている注意文の箇所の「MFA を割り当てる」ボタン、あるいはページ中ほどの「多要素認証 (MFA)」セクションの「MFA デバイスの割り当て」ボタンをクリックします。
5. デバイス名に任意の名称をつけたのちに、MFA デバイスとしていずれかを選択し、「次へ」ボタンをクリックします。
   1. ハードウェア TOTP トークンを選択した場合：
      1. デバイスの背面のシリアル番号を入力します。
      2. デバイスに表示された6桁の数字を MFA コード 1 欄に入力します。（前面にあるボタンを押す必要があるデバイスもあります。）
      3. 30 秒経過してから、デバイスに表示された異なる 6桁の数字を MFA コード 2 欄に入力します。（前面にあるボタンを押す必要があるデバイスもあります。）
      4. 上記入力が整ったら「MFA を追加」ボタンをクリックします。
   2. 認証アプリケーションを選択した場合：
      1. 対応する認証アプリケーションを起動します。
      2. AWS マネジメントコンソール 「Authenticator app」画面の 2番目のステップで「QR コードを表示」リンクをクリックし、表示された QR を＜1＞で起動した認証アプリケーションで読み取ります
      3. 認証アプリケーションに表示されているコードを MFA コード 1 に入力し、次回、認証アプリケーションのコードの表示が変わったタイミングで、MFA コード 2 に入力し、「MFA を追加」ボタンをクリックします。
   3. パスキーまたはセキュリティキーを選択した場合：
      1. Yubikey などの USB デバイスの場合は事前に挿入しておきます
      2. 「デバイスの設定」画面に遷移後、検出されたパスキーやセキュリティキーデバイスを用いるかどうかのポップアップ画面が表示されます。  
         問題がなければ、デバイス用の要素（指紋など）を読み取り、設定を継続します。
         1. 表示されているデバイスではないものを選択したい場合はポップアップ画面の「その他のオプション」などのリンクをクリックし、適宜選択します。
6. それぞれの MFA デバイスの登録が正常に完了すると、「自分の認証情報」画面に戻りますので、ページ中ほどの「多要素認証 (MFA)」セクションに先ほど登録した MFA デバイスが追加されていることを確認します。  
   ここまでの手順で追加した MFA デバイスによる追加認証は、次回の「root アカウント」でのログイン時以降から要求されるようになります。

## 2. IAM ユーザーの作成

<details><summary>CLI で行う場合</summary>
一連の操作を CloudShell を用いて AWS CLI で実行が可能です。
### 変数の設定

```bash
USER_NAME="replace user name"
PASSWORD="replace user password"
MFA_DEVICE_NAME="replace device name"
OUTFILE_NAME="QRCode.png"
```

### ユーザーの作成

```bash
aws iam create-user \
  --user-name "${USER_NAME}"
```

### ログインプロファイルの作成

```bash
aws iam create-login-profile \
  --user-name "${USER_NAME}" \
  --password "${PASSWORD}"
```

### 仮想 MFA デバイスの作成

```bash
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name "${MFA_DEVICE_NAME}" \
  --outfile "${OUTFILE_NAME}" \
  --bootstrap-method QRCodePNG
```

### 作成済み仮想 MFA デバイスの ARN の取得

```bash
MFA_DEVICE_ARN=$(aws iam list-virtual-mfa-devices \
  --query "VirtualMFADevices[?contains(SerialNumber, '"${MFA_DEVICE_NAME}"')].SerialNumber" \
  --output text)
```

### 認証アプリケーションでの読み取り

1. CloudShell 環境内に生成された QR コード画像をダウンロードす
2. CloudShell 画面内の右上部分「アクション」プルダウンをクリックし、展開後のメニューから「ファイルのダウンロード」をクリックする
3. 個別のファイルパス入力欄に、QR コード画像のフルパスを入力する。`OUTFILE_NAME="QRCode.png"` としていた場合は、`/home/cloudshell-user/QRCode.png` となるので同様の要領で入力し、「ダウンロード」ボタンをクリックする。
4. ダウンロードした QR コード画像を、Google Authenticator、Duo Mobile、Authy などの認証アプリケーションで読み取る。
5. 認証アプリケーションに表示されているコードを変数に格納（1回目） 
   1. `MFACODE1=` に続けて認証アプリケーションに表示されているコードを手入力し <key>Enter</key>を押す  
      ```bash
      MFACODE1=
      ```
   2. MFA デバイスのコードを変数に格納（2回目）  
      `MFACODE2=` に続けて認証アプリケーションに表示されているコードを手入力し <key>Enter</key>を押す  
      
      ```bash
      MFACODE2=
      ```

### 仮想 MFA デバイスの有効化（IAM ユーザーへの割り当て）

```bash
aws iam enable-mfa-device \
  --user-name "${USER_NAME}" \
  --serial-number "${MFA_DEVICE_ARN}" \
  --authentication-code1 "${MFACODE1}" \
  --authentication-code2 "${MFACODE2}"
```

</details>

## 3. IAM グループの作成
