# LimaとQEMU、そしてファイルシステム共有の仕組み

このドキュメントでは、macOS 上で Linux 環境を実現するためのツール「Lima」と、その基盤となる QEMU の役割、ならびにホストOSと Linux 仮想マシン間でのファイルシステム共有の仕組みについて解説しています。また、9p プロトコルによるマウント設定例と、その動作の違いについても説明します。

---

## 1. Lima の概要

- **Lima とは**  
  Lima は、macOS 上で Linux 環境を軽量な仮想マシンとして提供するオープンソースのツールです。Docker やその他 Linux ベースのコンテナランタイムを macOS 上で利用するために、内部で Linux VM を起動し、ホストとゲスト間の通信やファイル共有を実現しています。

- **内部動作**  
  - **Linux VM の構築:**  
    Lima は QEMU などの軽量な仮想化技術を利用して、Linux 環境を提供します。Docker デーモンはこの VM 内で起動し、実際のコンテナ実行は Linux カーネル上で行われます。
  - **SSH を利用した通信:**  
    ホスト（macOS）と Linux VM 間のコマンド伝達や管理は SSH 経由で行われ、セキュアな通信を実現しています。
  - **ファイルシステムの共有:**  
    Lima はホスト側の特定ディレクトリを Linux VM 内にマウントし、Docker のバインドマウントなどの仕組みがシームレスに動作するように構成されています。

---

## 2. QEMU の役割と活用例

- **QEMU とは**  
  QEMU (Quick EMUlator) は、オープンソースのエミュレータ・仮想化ツールです。  
  - **エミュレーション:**  
    CPU、メモリ、ストレージ、ネットワークなどのハードウェアをソフトウェア上で模倣し、異なるアーキテクチャの OS を実行できます。
  - **仮想化:**  
    ハードウェア支援型仮想化 (KVM など) と組み合わせ、ホストマシンのリソースを効率的に利用して VM を実行できます。

- **活用例**  
  - **KVM との組み合わせ:**  
    Linux カーネルの KVM と連携し、virt-manager や GNOME Boxes などの GUI ツールで VM の管理を行います。
  - **クラウドプラットフォーム:**  
    OpenStack や Proxmox VE などの大規模な仮想化基盤で利用されています。
  - **エミュレーション用途:**  
    異なるアーキテクチャ（ARM、MIPS など）のソフトウェア開発、組み込みシステムのシミュレーション、OS のテストなどに活用されます。

---

## 3. ホストと Linux VM のファイルシステム共有

### 3.1 共有の目的

- **Docker のバインドマウント**  
  Docker コンテナ内にホスト OS のディレクトリをマウントする際、実際には Linux VM 内にホスト側のディレクトリがマッピングされ、そこからコンテナにバインドマウントされます。  
- **通信の仕組みの違い:**  
  SSH は管理やコマンド伝達に利用され、ファイル共有そのものは QEMU が提供するファイルシステム共有機能（virtiofs や 9p）によって実現されています。

### 3.2 主要なファイルシステム共有方式

#### virtiofs

- **特徴:**  
  高速かつ効率的なファイル共有を実現するための新しい技術。ホスト側のファイルシステム（例: APFS）と Linux VM の間で透過的なアクセスを提供します。
- **制約:**  
  デフォルト設定では、ホストのファイルシステムの所有権やパーミッション情報をそのまま反映するため、コンテナ内での `chown` 操作が制限される場合があります。

#### 9p (Plan 9 Filesystem Protocol)

- **特徴:**  
  もともと Plan 9 OS 用に設計されたプロトコルで、ホストとゲスト間のファイル属性や所有権の変更を柔軟に扱えます。  
- **メリット:**  
  9p は、virtiofs の所有権変更制約がなく、コンテナ内から `chown` などの操作を行った際に問題が発生しにくいという特徴があります。

---

## 4. Lima における 9p マウントの設定例とその解説

以下は、Lima の公式ドキュメントに記載されている 9p の設定例です。

```yaml
vmType: "qemu"
mountType: "9p"
mounts:
- location: "~"
  9p:
    # Supported security models are "passthrough", "mapped-xattr", "mapped-file" and "none".
    # "mapped-xattr" and "mapped-file" are useful for persistent chown but incompatible with symlinks.
    # 🟢 Builtin default: "none" (since Lima v0.13)
    securityModel: null
    # Select 9P protocol version. Valid options are: "9p2000" (legacy), "9p2000.u", "9p2000.L".
    # 🟢 Builtin default: "9p2000.L"
    protocolVersion: null
    # The number of bytes to use for 9p packet payload, where 4KiB is the absolute minimum.
    # 🟢 Builtin default: "128KiB"
    msize: null
    # Specifies a caching policy. Valid options are: "none", "loose", "fscache" and "mmap".
    # Try choosing "mmap" or "none" if you see a stability issue with the default "fscache".
    # See https://www.kernel.org/doc/Documentation/filesystems/9p.txt
    # 🟢 Builtin default: "fscache" for non-writable mounts, "mmap" for writable mounts
    cache: null
```

### 各項目の解説

- **vmType: "qemu"**  
  Lima が QEMU を利用して仮想マシンを起動することを指定しています。

- **mountType: "9p"**  
  ファイルシステムの共有に 9p プロトコルを利用する設定です。virtiofs では発生する chown の制限を回避するために選択されることがあります。

- **mounts セクション**  
  ホストの特定ディレクトリ（この例ではユーザーディレクトリ "~"）を Linux VM 内にマウントします。

- **9p の詳細オプション** 
  - [参照](https://lima-vm.io/docs/config/mount/#9p)

  - **securityModel:**  
    - ファイルの所有権やパーミッションの扱い方を決定する。  
    - 選択肢には `"passthrough"`, `"mapped-xattr"`, `"mapped-file"`, `"none"` などがあり、デフォルトは `"none"`（Lima v0.13以降）。
    - `"mapped-xattr"` や `"mapped-file"` を選ぶと、永続的な chown が可能になるが、シンボリックリンクとの互換性に問題が出る可能性がある。

  - **protocolVersion:**  
    - 使用する 9p プロトコルのバージョンを指定します。  
    - オプションは `"9p2000"`, `"9p2000.u"`, `"9p2000.L"` で、ビルトインデフォルトは `"9p2000.L"` です。

  - **msize:**  
    - 9p パケットのペイロードサイズの設定です。最低 4KiB が必要ですが、デフォルトは `"128KiB"` となり、効率的なデータ転送を実現します。

  - **cache:**  
    - キャッシングポリシーの指定です。  
    - オプションとして `"none"`, `"loose"`, `"fscache"`, `"mmap"` があり、デフォルトは非書き込みマウントなら `"fscache"`、書き込み可能なら `"mmap"` が用いられます。
    - 安定性の問題がある場合は `"mmap"` や `"none"` に変更することが推奨されています。

### 9p を選ぶ理由

- **chown の問題解消:**  
  Lima のデフォルトでは virtiofs を利用しており、ホスト側のファイルシステムの所有権情報が固定されるため、コンテナ内で chown を試みるとエラーになる場合があります。  
  9p は、所有権変更の操作を柔軟に扱えるため、chown の問題が解消され、ファイル属性の変更がより自由に行える仕組みとなります。

---

## 5. まとめ

- **Lima** は macOS 上で Linux コンテナ環境を提供するため、QEMU を利用して Linux 仮想マシンを起動し、SSH を用いて管理・コマンド伝達を行います。また、ホストと Linux VM 間のファイル共有は、virtiofs や 9p などの技術によって実現されます。
- **QEMU** はエミュレーションと仮想化の両面を持つツールであり、Lima の他にも KVM のバックエンド、クラウドプラットフォーム、エミュレーション用途など多岐にわたって利用されています。
- **ファイルシステムの共有メカニズム** では、virtiofs は高速ですが所有権変更の制約があるのに対し、9p は chown 操作などの柔軟なファイル属性操作が可能です。
- **9p の設定例** では、各オプション（securityModel、protocolVersion、msize、cache）を用いて細かく挙動を調整でき、特に chown エラーの問題に対する対策として利用されます。

このように、Lima の仕組みは、ホスト OS と Linux 仮想マシン間の連携や、コンテナ環境での柔軟なファイル共有を実現するための各種技術が組み合わされて構築されています。