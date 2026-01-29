サーバー構築時のユーザーアカウントのホームディレクトリ内にシンボリックリンクを作成する
=========================

作業内容
-------------------------

`/var/www`直下のディレクトリを抽出し、同名のユーザーアカウントが存在する場合は、
そのホームディレクトリ直下に`public`というシンボリックリンクを作成するタスクを作成します。

また、ユーザーアカウントと同名のデータベースが存在する場合は、
データベースの接続設定ファイルをユーザーのホームディレクトリ直下に作成します。
※これはMariaDBを想定しています。

背景・理由
-------------------------

毎回Playbookの`tasks`セクションに同様のタスクを記述する事が多いため、
このロールに定型処理としてまとめておくことで、再利用性を高め、保守性を向上させることを目的としています。

また、作業内容に記載した処理をなぜ行うのかについてですが、
単一のサーバーに複数の環境を構築することが多く、

Webサーバーの各バーチャルホストのドキュメントルートは`/var/www/{environment}`と作成し、
各環境のドキュメントルートを`/var/www`直下に配置することでサーバー管理者視点で管理を容易にしています。

しかし、アプリケーション開発者視点では、各自のユーザーアカウントのホームディレクトリ直下に
ドキュメントルートが存在した方が利便性が高いため、
ユーザーアカウントのホームディレクトリ直下にシンボリックを作成する必要があります。
また、アプリケーション開発者がデータベースに接続する際に、
接続設定ファイルがユーザーアカウントのホームディレクトリ直下に存在した方が利便性が高いため、
同様に接続設定ファイルを作成する必要があります。

期待する成果物
-------------------------

- tasksディレクトリ以下に適切なファイル名でタスクファイルが作成されていること

参考情報
-------------------------

### 条件・制約事項

除外するユーザーアカウントやデータベース名がある場合は、変数で指定できるようにしてください。

### その他の参考情報

以下の`tasks`セクションが現在の実装内容を示していますがあまりにも効率が悪いため、
前述の通りディレクトリとユーザーの存在を確認し自動で行えることが理想です。

```yaml
 ---
- name: Setup LEMP Server
  hosts: all
  become: true
  pre_tasks:
    - name: 環境変数の設定状況を検証
      ansible.builtin.assert:
        that: lookup('env', item) | length > 1
        success_msg: "環境変数「{{ item }}」は設定されています"
        fail_msg: "環境変数「{{ item }}」が定義されていない、または、空文字です"
      loop: "{{ assert_env_names | default([]) }}"
  roles:
    - role: common
      tags:
        - role_common
    - role: nodejs
      tags:
        - role_nodejs
    - role: logrotate
      tags:
        - role_logrotate
    - role: mackerel_agent
      tags:
        - role_mackerel_agent
    - role: opendkim
      tags:
        - role_opendkim
    - role: mariadb
      tags:
        - role_mariadb
    - role: dehydrated
      tags:
        - role_dehydrated
    - role: nginx
      tags:
        - role_nginx
    - role: php
      tags:
        - role_php
    - role: tools
      tags:
        - role_tools
    - role: dns
      tags:
        - role_dns
  tasks:
    - name: バーチャルホストのオーナー/グループを設定
      ansible.builtin.file:
        path: /var/www/{{ item.key }}
        state: directory
        owner: "{{ item.value.owner | default(item.key) }}"
        group: "{{ item.value.group | default(item.key) }}"
        mode: "0755"
      loop: "{{ virtualhost_maps | dict2items }}"
      tags:
        - virtualhost
    - name: 各アカウントのホームディレクトリ内にドキュメントルートのシンボリックリンク「www」を作成
      ansible.builtin.file:
        path: /home/{{ item.value.owner | default(item.key) }}/www
        src: /var/www/{{ item.key }}
        state: link
        owner: "{{ item.value.owner | default(item.key) }}"
        group: "{{ item.value.group | default(item.key) }}"
        mode: "0755"
      loop: "{{ virtualhost_maps | dict2items }}"
      tags:
        - virtualhost
    - name: データベース接続設定
      ansible.builtin.include_tasks:
        file: tasks/connect_database.yml
      loop:
        - owner: "{{ lookup('env', 'SRMS_KAKOGAWA_USER') }}"
          db_name: "{{ lookup('env', 'SRMS_KAKOGAWA_DB_NAME') }}"
          db_user: "{{ lookup('env', 'SRMS_KAKOGAWA_DB_USER') }}"
          db_password: "{{ lookup('env', 'SRMS_KAKOGAWA_DB_PASSWORD') }}"
        - owner: "{{ lookup('env', 'SRMS_HIMEJI_USER') }}"
          db_name: "{{ lookup('env', 'SRMS_HIMEJI_DB_NAME') }}"
          db_user: "{{ lookup('env', 'SRMS_HIMEJI_DB_USER') }}"
          db_password: "{{ lookup('env', 'SRMS_HIMEJI_DB_PASSWORD') }}"
        - owner: "{{ lookup('env', 'SRMS_STAGING_USER') }}"
          db_name: "{{ lookup('env', 'SRMS_STAGING_DB_NAME') }}"
          db_user: "{{ lookup('env', 'SRMS_STAGING_DB_USER') }}"
          db_password: "{{ lookup('env', 'SRMS_STAGING_DB_PASSWORD') }}"
        - owner: "{{ lookup('env', 'SRMS_DEVELOP_USER') }}"
          db_name: "{{ lookup('env', 'SRMS_DEVELOP_DB_NAME') }}"
          db_user: "{{ lookup('env', 'SRMS_DEVELOP_DB_USER') }}"
          db_password: "{{ lookup('env', 'SRMS_DEVELOP_DB_PASSWORD') }}"
      loop_control:
        loop_var: connect_setting
```

関連ファイル
-------------------------

<!-- 
この作業に関連する作業プランファイルやドキュメントのリンクが自動的に追記されます。
作業開始時は空欄で構いません。

### 作業プラン

作業プランファイルが作成されると、以下の形式で自動追記されます：
- [作業プラン - {作業名/チケット番号}]({作業プランファイルパス})
  - 作成日: YYYY-MM-DD
  - 最終更新: YYYY-MM-DD
  - 状況: {進行中/完了/中断}
  - 進捗: XX%

### その他の関連ドキュメント

必要に応じて手動で追記してください。
-->
