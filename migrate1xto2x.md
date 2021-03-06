---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: single
classes: wide
title: "KAMONOHASHI 1.xから2.xへの移行"
permalink: /docs/install-and-update/migrate1xto2x
sidebar:
  nav: "docs"
---

## 考慮事項
* KAMONOHASHI2.xではインフラ部分に[deepops](https://github.com/NVIDIA/deepops)を採用する変更を行いました。
  * Ubuntu16.04はサポートされないため、18.04へのバージョンアップが必要になります
  * インフラ部分を含めたアンインストールとインストールが必要になります
* 2.xへのバージョンアップ後は、1.xへのバージョンダウンはできません
* 2.xの[マシンの要件](/docs/install-and-update/prerequisite)を事前にご確認ください

## 事前準備
* KAMONOHASHIのバックアップが取れていることを確認します
* KAMONOHASHIのAdminアカウントでログインできることを確認します

## KAMONOHASHI 1.xのアンインストール
次のコマンドを実行します
```
cd /var/lib/kamonohashi/deploy-tools/<version>/infra/
./deploy-kqi-infra.sh prepare
./deploy-kqi-infra.sh clean
```
* deepopsがインストールするGPU driverとの競合を防ぐため、GPUドライバをアンインストールします
* Ubuntu 16.04をUbuntu 18.04にバージョンアップしてください
* Kubernetes Masterの`/var/lib/kamonohashi/deploy-tools/`ディレクトリを別の場所に退避します
* アンインストール完了後、[マシンの要件](/docs/install-and-update/prerequisite)に合わせた設定をしてください

## 2.x構築ツールのセットアップ
- Kubernetes masterをインストールするマシンにログインします。
- `sudo su -`を実行し、rootユーザーになります
- `mkdir -p /var/lib/kamonohashi/ && cd /var/lib/kamonohashi/ `を実行します
- `git clone https://github.com/KAMONOHASHI/deploy-tools.git -b 2.0.0.6 --recursive`を実行してデプロイスクリプトを入手します
- `/var/lib/kamonohashi/deploy-tools/`に移動します
- プロキシ環境下では`./deepops/scripts/proxy.sh`ファイルにプロキシ設定を記載してください
  *  no_proxyには`localhost,127.0.0.1,.cluster.local,使用するマシンのIPアドレス・ホスト名`の記載をしてください
- `./deploy-kamonohashi.sh prepare`を実行して構築に必要なソフトウェアをインストールします

* ansibleでエラーが出る場合はansibleのアンインストールを実行してから`prepare`を実行してください
  * スクリプト実行中に適切なansibleがインストールされます
  * ubuntuアップグレードに際し、ansibleの実行ファイルのみ残存するケースがあります。ansibleのアンインストールと`prepare`実行時に次のようなエラーが発生します。その場合は、`mv /usr/local/bin/ansible /tmp/`のようなコマンドでansible実行ファイルを退避して`prepare`を実行してください

  * ansibleのアンインストールでのエラー

    ```
    Cannot uninstall requirement ansible, not installed
    ```

  * prepare実行時のエラー

    ```
    Traceback (most recent call last):
      File "/usr/local/bin/ansible", line 40, in <module>
        from ansible.errors import AnsibleError, AnsibleOptionsError, AnsibleParserError
    ModuleNotFoundError: No module named 'ansible'
    ```



## デプロイ構成の設定 
`./deploy-kamonohashi.sh configure cluster`を実行します。
対話形式で聞かれる以下の内容を入力します


|質問文|解説|
|---|---|
|Kubernetes masterを<br>デプロイするサーバ名||
|KAMONOHASHIを<br>デプロイするサーバ名||
|Storageをデプロイするサーバ名|HWベンダーのNFSを使用する場合は[カスタマイズ設定ガイド](/docs/install-and-update/customize-2x)を参照してください|
|計算ノード名|,区切りで複数指定できます。<br>例: gpu1,gpu2,gpu3 |
|SSHで利用するユーザー名:|構築時に使用するSSHユーザーを指定します。構築ツールがSSH経由で構築を行う仕様のため、指定が必要になります|

入力内容に応じ、以下の設定ファイルに書き込みが行われます
* deepopsの設定ファイル(deepops/config/inventry)
* kamonohashiの設定ファイル(kamonohashi/conf/settings.yml)

設定内容をカスタマイズする場合は次を参照し、設定ファイルの編集を行ってください。
[カスタマイズ設定ガイド](/docs/install-and-update/customize-2x)

## デプロイの実行
以下のコマンドではデプロイ構成の設定で指定したユーザーでSSHが実行されます。
指定したユーザーでのSSHにパスワードが必要な場合は`-k`、
指定したユーザーでのsudoにパスワードが必要な場合は`-K`のオプションを指定します。
例： `./deploy-kamonohashi.sh clean nvidia-repo -k -K`

* sshキーを~/.ssh/id_rsaで配置している場合は、ペアのid_rsa.pubも~/.sshに配置してください

1. `./deploy-kamonohashi.sh clean nvidia-repo`を実行します
  * ubuntu16用のnvidiaリポジトリ登録が残っているためです
2. `./deploy-kamonohashi.sh deploy nvidia-gpg-key`を実行します。
  * nvidiaのGPGキーは期限が半年であり、期限が切れるとaptが失敗することの対応です
3. `./deploy-kamonohashi.sh deploy all`を実行します。

実行後、対話形式で聞かれる以下の内容を入力します


|質問文|解説|
|---|---|
|Admin Passwordを入力:|KAMONOHASHIのadminアカウントで使用する8文字以上のパスワードです。数字のみのパスワードは使用不可となっているので注意してください。KAMONOHASHI Web UIログイン・DB接続、Object Storageへのログインに使用します。<br>一度構築に使用したパスワードはデプロイツールでは変更できません。パスワードを変える場合は、完全にデータを削除するか、パスワード変更手順を実施する必要があります。パスワード変更手順は[kamonohashi-support@jp.nssol.nipponsteel.com]にお問い合わせください|
|SSH password: |構築時に使用する、sshユーザーのパスワードです。`-k`指定時のみ聞かれます|
|SUDO password[defaults to SSH password]: |構築時に使用する、sshユーザーのsudoパスワードです。`-K`指定時のみ聞かれます。|

入力後に構築が始まります。
構築には20分程かかります。

構築後にアクセス用のURLが表示されるので、それをブラウザで開きます
