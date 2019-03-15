---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: single
classes: wide
title: Admin Guide
permalink: /docs/admin/
sidebar:
  nav: "docs"
---

本ドキュメントは2019.2.28時点の情報である。

# 本書の目的
本書では、システム管理者を対象として、KAMONOHASHIの使用中に管理者が行う操作を説明する事を目的としている。

# 前提
KAMONOHASHIのセットアップが完了していることを前提としている。

# テナント

## テナント情報の編集
テナント情報、ストレージ情報、Git情報、Dockerレジストリ情報は、各管理画面からそれぞれ編集できる。
一覧に表示されている各項目をクリックして編集画面を表示し、必要な情報を入力し、更新する。

# ユーザ

## ユーザの追加とテナントへの関連付け

KAMONOHASHIではローカルユーザか、LDAPユーザかによって、ユーザの追加フローが異なる。
ただし、ユーザの種類に寄らず、ユーザ名はシステム内で一意である必要がある。

### ローカルユーザの追加

ローカルユーザの場合、KAMONOHASHI内部で認証情報を管理される。

ユーザの新規作成は、以下のフローで実行できる。

1. 管理権限を持つユーザでログイン
1. [システム管理] → [ユーザ管理]を選択し、ユーザ一覧画面を表示する
1. 画面右上の新規作成ボタンを押下する
1. 画面に下記の情報を入力し、ローカルユーザを新規に追加する。
※ローカルユーザの場合、初期テナントの関連付けを作成時にまとめて行う。


![ユーザ新規作成画面(スクショ3-5)](/assets/images/admin_figure5.png)

|項目  |要否  |説明  |
|---|---|---|
|ユーザアカウント  |必須  |ユーザ名。システム内で一意である必要がある。英数字およびハイフンのみが使用可能。 |
|パスワード  |必須  |ユーザ認証に用いるパスワード。 |
|パスワード(再入力)  |必須  |パスワードの確認用。上記パスワードと一致している必要がある。 |
|システムロール  |任意  |追加ユーザに付与するシステムロールを選択する。 |
|テナント |必須  |ユーザを関連付けるテナントを選択する。複数可。 |
|テナントロール  |任意  |上記テナント項目で選択した分のテナントについて、ユーザが所有するテナントロールをそれぞれ選択する。複数可。 |

### LDAPユーザの追加

LDAPユーザの場合、認証情報の管理にKAMONOHASHI外のLDAP認証サービスを利用する。
パスワード変更など、LDAPユーザの管理はKAMONOHASHI上では行えない。

LDAPユーザは、管理者ではなく、LDAPユーザ自身がKAMONOHASHIにログインすることで追加できる。
ユーザがLDAPサービスに登録されているユーザ名とパスワードでKAMONOHASHIにログインすると、KAMONOHASHI内にユーザ情報が登録され、Sandboxテナントに自動で関連付けられる。

LDAPユーザに対して、システムロールの付与や、Sandbox以外のテナントへの関連付けを行う場合、下記の処理を実行する。

1. ユーザがLDAPサービスに登録されているユーザ名とパスワードでKAMONOHASHIにログインする
    * KAMONOHASHI内にユーザ情報が登録される
    * 既定ではSandboxテナントに自動で関連付けられる
1. 管理権限を持つユーザが、[システム管理] → [ユーザ管理]を選択し、ユーザ一覧画面を表示する
1. 追加されているLDAPユーザ名を一覧から探し、列を選択する
1. 表示されたユーザ編集画面で、ローカルユーザの場合と同様に、システムロールと関連付けるテナントおよびそのテナントロールをそれぞれ選択する

## ロール
ユーザにロールを付与することで、操作可能な機能を制御できる。
一人のユーザには複数のロールを関連付けられる。

ロールは大きく以下の2種類に大別される。

* システムロール
    * テナントを横断する、システム全体に関連した情報を取り扱うためのロール
    * 既定では全てのシステム管理権限がある「Adminロール」が使用できる
* テナントロール
    * 特定のテナント内に限定された情報を扱うためのロール
    * 所属するテナントごとに、ユーザに付与するテナントロールを指定する
        * 例えば、同一のユーザに対し、AテナントにはManagerロールを付与し、BテナントにはUserロールとResercherロールを付与する、など
    * 既定では以下の3種類のテナントロールが使用できる

|ロール  |説明  |
|---|---|
|User  |データの追加・削除、およびデータセットの作成・変更を行う権限を持つ。  |
|Researcher  |ジョブの実行、結果の評価を行う権限を持つ。  |
|Manager  |そのテナント内で使用する計算リソースや権限の管理を行う権限を持つ。  |

## カスタムロールの追加

既定されているロールに加え、独自のロールを追加することができる。追加されたロールは、全テナントで利用可能になる。

ロールの新規作成画面は、以下のフローで表示できる。

1. 管理権限を持つユーザでログイン
1. [システム管理] → [ロール管理]を選択し、ロール管理画面を表示する
1. 画面右上の新規登録ボタンを押下する
1. 画面に下記の情報を入力し、カスタムロールを新規に追加する。

![ロール新規登録画面(スクショ3-9)](/assets/images/admin_figure9.png)


|項目  |要否  |説明  |
|---|---|---|
|ロール名  |必須  |ロールの識別名。システム内で一意である必要がある。英数字およびハイフンのみが使用可能。 |
|表示名  |必須  |ユーザに表示される名前。任意の文字を使用可能。 |
|種別  |必須  |テナントロールの場合は"テナント"、システムロールの場合は"システム"を選択する。 |
|ソート順  |必須  |並び順を入力する。小さいほど前に表示される。一意である必要はないが、同じ値の場合のソート順は変動する可能性がある。 |

追加したカスタムロールの権限設定は、[メニューアクセス管理](#menu-access)で行う。

# ノード
<a name="new_node"></a>

## ノードの追加
KAMONOHASHIの管理下に新しくノードを追加する。

ノードの新規登録画面は、以下のフローで表示できる。

1. 管理権限を持つユーザでログイン
1. [システム管理] → [ノード管理]を選択し、ノード一覧画面を表示する
1. 画面右上の新規作成ボタンを押下する
1. 画面に下記の情報を入力し、ノードを新規に追加する

![ノード新規作成画面(スクショ3-6)](/assets/images/admin_figure6.png)


|項目  |要否  |説明  |
|---|---|---|
|名前  |必須  |ノード名。システム内で一意である必要がある。英数字およびハイフンのみが使用可能。 |
|メモ  |任意  |管理用のメモを自由に入力できる。 |
|パーティション |任意  |ノードにパーティションを設定できる。パーティションについての詳細は後述。 |
|アクセスレベル |任意  |ノードへのアクセスレベルを設定できる。アクセスレベルについての詳細は後述。Privateを選択した場合、アクセスを許可するテナントを選択するUIが表示される。指定しなかった場合はDisabledが設定される。 |
|TensorBoard実行可否 |任意  |ノード上でTensorBoardコンテナの実行を許可するか指定できる。 |

### パーティション
パーティションを設定することで、ジョブを実行するノードを指定することができる。
ジョブの実行時にパーティションを指定すると、そのパーティションが付与されたノードでのみ、ジョブが実行される。パーティションとして指定できる文字は、英数字及びハイフンのみである。

![パーティション管理(図8-1)](/assets/images/admin_figure7.png)

### アクセスレベル

アクセスレベルは、テナント単位で利用可能なノードを設定するための仕組みである。
アクセスレベルには以下の3種類がある。

* Disabled
    * どのテナントも、そのノードを利用することができない
    * ノードのメンテナンスを行う際などに指定する
* Private
    * 指定されたテナント（複数可）のみが、そのノードを利用することができる
* Public
    * すべてのテナントがそのノードを利用することができる。

# クォータ
クォータはテナントがクラスタで利用できる最大のCPUコア数、メモリ容量、GPU数の上限を表す。
クォータを利用することで、特定のテナントがクラスタのリソースを大量に利用することを抑制できる。

## クォータ管理
クォータを設定するには、[システム管理] → [クォータ管理]を選択し、クォータ設定の一覧画面を表示し行う。

各テナントに対して、次の情報を入力する。
クォータの各項目に0を設定した場合、無制限に利用できる。

|項目  |要否  |説明  |
|---|---|---|
|CPU  |任意  |テナントが利用できる最大CPUの量。CPUのコア数で入力する。  |
|Memory  |任意  |テナントが利用できる最大メモリの量。ギガバイト(Gi)単位で入力する。  |
|GPU  |任意  |テナントが利用できる最大GPUの数。GPUの枚数で入力する。  |


# リソース

リソースとは、クラスタを構成する各ノードの計算リソース（CPU、メモリ、GPU）を表す。

## リソース管理

クラスタの各ノード上に現在配備されているコンテナと、それらに割り当てられたリソースを一覧で確認する事ができる。これによって、ノード単位の負荷状況や、リソースを大量消費しているテナントやユーザを把握することができるので、必要に応じて、不要なコンテナを削除し、リソースの解放処理を行うことができる。

現在のリソース割当状況を確認するには、管理権限を持つユーザでログインした後、[システム管理] → [リソース管理] を選択し、リソース管理の一覧画面を表示し行う。

この画面には、以下三つの表示形式がある。

- ノード別
    - ノードごとに、現在使用しているリソースの合計と、稼働しているコンテナの一覧が確認できる
    - [ノード管理](#new_node)でまだ追加されていないノードがクラスタ上で確認された場合、ノード名の前に"Unknown:"が接頭語として表示される
    - まだ配備先のノードが決定しておらず、待機状態のコンテナがある場合、"Unassined"というグループにまとめられる
- テナント別
    - テナントごとに、現在使用しているリソースの合計と、稼働しているコンテナの一覧が確認できる
    - クォータが設定されておらず、無制限にリソースが使用できる場合、"Infinity"が表示される
- コンテナ一覧
    - 全コンテナの一覧と、その利用リソースが表示される


# メニュー

メニューとは、"データ管理"や"テナント管理"など、KAMONOHASHIが提供する各機能セットを表す。

メニューには、以下の4つの種別がある

- Public
    - 未ログインユーザにも公開されるメニュー（e.g. ログイン）
- Internal
    - ログインユーザに公開されるメニュー（e.g. ダッシュボード）
    - ロールを一つも持っていないユーザにも利用可能
- Tenant
    - 特定のテナントロールを持つユーザのみに公開されるメニュー（e.g. データ管理）
    - 接続中のテナントでに限定された情報を取り扱う
- System
    - 特定のシステムロールを持つユーザ飲みに公開されるメニュー（e.g. テナント管理）
    - テナントを横断する、システム全体に関連した情報を取り扱う

<a name="menu-access"></a>

## メニューアクセス管理

メニューアクセス管理画面では、全メニューを一覧で確認できる。
加えて、種別がTenantあるいはSystemであるメニューに対しては、公開先のロールを編集することができる。

メニューアクセス管理画面を表示するには、管理権限を持つユーザでログインした後、[システム管理] → [メニューアクセス管理]を選択する。

![メニューアクセス管理(スクショ308)](/assets/images/admin_figure8.png)

メニューの種別がTenantあるいはSystemである場合、以下の手順でそのメニューを利用可能なロールを改廃できる。

1. ロールを改廃したいメニューの右側に表示されているボタン群を確認する。青く塗られているロールがアクセス権があるロール、白く塗られているロールがアクセス権がないロールである。
1. アクセス権を付与したいロールをクリックし、背景を白から青に変える。
1. アクセス権を削除したいロールをクリックし、背景を青から白に変える。
1. 全ての変更が完了したら、右下の「更新」ボタンを選択する
