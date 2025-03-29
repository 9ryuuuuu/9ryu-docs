# pythonの設定について

## 目次
- [install済みpython](#install済みpython)
- [仮想環境構築方法](#仮想環境構築方法)
- [Python アンインストール手順](#python-アンインストール手順pkg-インストール版)

---
---

## install済みpython
python3

```bash
$ python3 --version
Python 3.13.2
```

インストール先
```bash
$ which python3.13
/Library/Frameworks/Python.framework/Versions/3.13/bin/python3.13
```

多分[こちら](https://www.python.org/downloads/macos/)からpkdを落としてインストールした。

---
---


## 仮想環境構築方法

pythonの作業ディレクトリに移動して、仮想環境を作成し、activateする。
```bash
~$ cd workspace/python 

~/workspace/python$ python3.13 -m venv ex_env

~/workspace/python$ ls
README.md	dify		ex_env

~/workspace/python$ source ex_env/bin/activate

(ex_env) ~/workspace/python$ which python
/Users/akira_kuriyama/workspace/python/ex_env/bin/python
```

deactivate
```bash
(ex_env) ~/workspace/python$ deactivate 
~/workspace/python$ 
```


---
---

## Python アンインストール手順（pkg インストール版）

以下の手順は、Python 3.11 のファイル、シンボリックリンク、およびインストールレシート（Receipts）を削除する方法です。作業前に、必要なバックアップを取得するなどの準備をしてください。

---

### 1. Python Framework の削除

Python.org の pkg インストーラでインストールした場合、フレームワークは以下のディレクトリに配置されています。

```bash
$ which python3.11
/Library/Frameworks/Python.framework/Versions/3.11/bin/python3.11
```

削除します。

```bash
sudo rm -rf /Library/Frameworks/Python.framework/Versions/3.11
```

---

### 2. シンボリックリンクの削除

`/usr/local/bin` 以下に作成されたシンボリックリンクも削除します。以下は、Python 3.11 に関連するリンクの例です。

```bash
$ ls -l /usr/local/bin | grep 3.11
lrwxr-xr-x  1 root  wheel         67  8 11  2023 2to3 -> ../../../Library/Frameworks/Python.framework/Versions/3.11/bin/2to3
lrwxr-xr-x  1 root  wheel         72  8 11  2023 2to3-3.11 -> ../../../Library/Frameworks/Python.framework/Versions/3.11/bin/2to3-3.11
lrwxr-xr-x  1 root  wheel         71  8 11  2023 idle3.11 -> ../../../Library/Frameworks/Python.framework/Versions/3.11/bin/idle3.11
lrwxrwxr-x  1 root  admin         70  8 11  2023 pip3.11 -> ../../../Library/Frameworks/Python.framework/Versions/3.11/bin/pip3.11
lrwxr-xr-x  1 root  wheel         72  8 11  2023 pydoc3.11 -> ../../../Library/Frameworks/Python.framework/Versions/3.11/bin/pydoc3.11
lrwxr-xr-x  1 root  wheel         73  8 11  2023 python3.11 -> ../../../Library/Frameworks/Python.framework/Versions/3.11/bin/python3.11
lrwxr-xr-x  1 root  wheel         80  8 11  2023 python3.11-config -> ../../../Library/Frameworks/Python.framework/Versions/3.11/bin/python3.11-config
lrwxr-xr-x  1 root  wheel         81  8 11  2023 python3.11-intel64 -> ../../../Library/Frameworks/Python.framework/Versions/3.11/bin/python3.11-intel64
```

削除します。

```bash
sudo rm -f /usr/local/bin/2to3
sudo rm -f /usr/local/bin/2to3-3.11
sudo rm -f /usr/local/bin/idle3.11
sudo rm -f /usr/local/bin/pip3.11
sudo rm -f /usr/local/bin/pydoc3.11
sudo rm -f /usr/local/bin/python3.11
sudo rm -f /usr/local/bin/python3.11-config
sudo rm -f /usr/local/bin/python3.11-intel64
```

---

### 3. アプリケーションフォルダの削除

もし `/Applications` 以下に Python 3.11 関連のフォルダが作成されている場合は、以下のように削除します。

```bash
sudo rm -rf "/Applications/Python 3.11"
```

---

### 4. インストールレシート（Receipts）の削除

pkg インストーラで登録されたレシート情報を削除します。以下のようなコマンドを実行して、Python 3.11 に関連するReceiptsを探します。
```bash
$ pkgutil --pkgs | grep -i python | grep 3.11

org.python.Python.PythonFramework-3.11
org.python.Python.PythonDocumentation-3.11
org.python.Python.PythonApplications-3.11
org.python.Python.PythonUnixTools-3.11
```


以下のコマンドで Python 3.11 関連のレシートを削除してください。

```bash
sudo pkgutil --forget org.python.Python.PythonFramework-3.11
sudo pkgutil --forget org.python.Python.PythonDocumentation-3.11
sudo pkgutil --forget org.python.Python.PythonApplications-3.11
sudo pkgutil --forget org.python.Python.PythonUnixTools-3.11
```

---

### 注意点

- **システム全体への影響:**  
  これらの操作はシステムレベルの変更となるため、ほかの Python バージョンとの兼ね合いや依存ツールへの影響を十分に確認してください。

- **作業前のバックアップ:**  
  万が一に備えて、必要なファイルのバックアップを取得してから作業を進めることをお勧めします。

- **確認:**  
  削除後、以下のコマンドで対象のファイルやレシートが削除されたことを確認してください。

  ```bash
  which python3.11
  pkgutil --pkgs | grep -i python
  ```

---

以上の手順に従えば、macOS における pkg インストーラ版 Python 3.11 の削除が完了します。何か不明点があればお知らせください。