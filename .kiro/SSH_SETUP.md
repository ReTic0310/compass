# SSH認証セットアップガイド

GitHubでSSH認証を使用するための完全ガイド（推奨）

---

## なぜSSH認証？

### ✅ メリット

1. **パスワード不要**
   - プッシュ・プル時にパスワード入力不要
   - 鍵ベース認証で自動化

2. **セキュリティ**
   - Personal Access Tokenより安全
   - 秘密鍵はローカルにのみ保存
   - パブリックキーのみをGitHubに登録

3. **使いやすさ**
   - 一度設定すれば永続的に動作
   - 2FA（2要素認証）でもスムーズ
   - 複数のリポジトリで同じ鍵を使用可能

4. **企業環境**
   - エンタープライズ環境で標準
   - セキュリティポリシーに準拠

### ⚠️ HTTPS認証の問題点

- Personal Access Tokenの管理が必要
- トークンの有効期限管理
- 毎回認証が必要（キャッシュしない場合）
- 2FAで追加の手順

---

## セットアップ（5分）

### Step 1: SSH鍵の生成

既存のSSH鍵を確認：

```bash
ls -al ~/.ssh
# id_rsa.pub, id_ed25519.pub などがあればOK
```

**新しいSSH鍵を生成**（既存の鍵がない場合）：

```bash
# Ed25519アルゴリズム（推奨・最新・高速）
ssh-keygen -t ed25519 -C "your_email@example.com"

# または RSA（互換性重視）
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**プロンプトの応答**：
```
Enter file in which to save the key (/Users/you/.ssh/id_ed25519):
→ Enterキーを押す（デフォルトパス使用）

Enter passphrase (empty for no passphrase):
→ パスフレーズ入力（推奨）またはEnterでスキップ

Enter same passphrase again:
→ 同じパスフレーズを再入力
```

**生成完了**：
```
Your identification has been saved in /Users/you/.ssh/id_ed25519
Your public key has been saved in /Users/you/.ssh/id_ed25519.pub
```

---

### Step 2: SSH-agentに鍵を追加

SSH-agentを起動し、秘密鍵を追加：

```bash
# SSH-agentをバックグラウンドで起動
eval "$(ssh-agent -s)"

# macOSの場合、~/.ssh/configを作成/編集
cat << EOF >> ~/.ssh/config
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
EOF

# SSH鍵を追加
ssh-add --apple-use-keychain ~/.ssh/id_ed25519

# Linux/Windowsの場合
ssh-add ~/.ssh/id_ed25519
```

**確認**：
```bash
ssh-add -l
# 鍵が表示されればOK
```

---

### Step 3: GitHubに公開鍵を登録

#### 3-1. 公開鍵をコピー

```bash
# macOS
pbcopy < ~/.ssh/id_ed25519.pub

# Linux
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard

# Windows (Git Bash)
cat ~/.ssh/id_ed25519.pub | clip

# または手動でコピー
cat ~/.ssh/id_ed25519.pub
```

公開鍵の例：
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJl3dIeudNqd0DPMRD6OIh65A9pu9p8kvI8q6WPLjFZs your_email@example.com
```

#### 3-2. GitHubに登録

1. GitHubにログイン
2. 右上のプロフィール → **Settings**
3. 左メニュー → **SSH and GPG keys**
4. **New SSH key** をクリック

**登録内容**：
- **Title**: `Compass Development - MacBook Pro`（わかりやすい名前）
- **Key type**: `Authentication Key`
- **Key**: コピーした公開鍵をペースト

5. **Add SSH key** をクリック
6. パスワード確認（必要に応じて）

---

### Step 4: 接続テスト

GitHubへの接続を確認：

```bash
ssh -T git@github.com
```

**初回実行時のプロンプト**：
```
The authenticity of host 'github.com (140.82.121.4)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
→ `yes` と入力

**成功メッセージ**：
```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

✅ これが表示されれば成功！

---

### Step 5: Kiro SDDで使用

```bash
# SSH認証でGit初期化
/kiro/git init git@github.com:username/compass.git

# または短縮形
/kiro/git init username/compass
```

**出力例**：
```
✅ Git repository initialized (SSH authentication)

Remote: origin → git@github.com:username/compass.git
Authentication: SSH (Ed25519)
Branch: main
Initial commit: ✅

You're all set! No password needed for push/pull.
```

---

## トラブルシューティング

### Q: `Permission denied (publickey)` エラー

**原因**: SSH鍵がGitHubに登録されていない

**解決**:
```bash
# 公開鍵を確認
cat ~/.ssh/id_ed25519.pub

# GitHubに登録されているか確認
# Settings → SSH and GPG keys で確認

# ssh-agentに追加されているか確認
ssh-add -l
```

### Q: `Host key verification failed` エラー

**原因**: GitHubのホスト鍵が未確認

**解決**:
```bash
# GitHubのホスト鍵を手動で追加
ssh-keyscan github.com >> ~/.ssh/known_hosts

# または接続テストで yes を選択
ssh -T git@github.com
```

### Q: `Could not open a connection to your authentication agent`

**原因**: SSH-agentが起動していない

**解決**:
```bash
# SSH-agentを起動
eval "$(ssh-agent -s)"

# 鍵を追加
ssh-add ~/.ssh/id_ed25519
```

### Q: 複数のGitHubアカウントを使いたい

**解決**: `~/.ssh/config` で設定

```bash
# ~/.ssh/config
Host github.com-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work

Host github.com-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
```

使用時：
```bash
# Work account
/kiro/git init git@github.com-work:company/project.git

# Personal account
/kiro/git init git@github.com-personal:username/compass.git
```

### Q: パスフレーズを毎回入力したくない

**解決**: Keychainに保存（macOS）

```bash
# ~/.ssh/config
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519

# 鍵を追加（Keychain保存）
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

---

## セキュリティベストプラクティス

### ✅ DO

1. **パスフレーズを設定**
   - 鍵が漏洩しても保護される
   - 強力なパスワード（12文字以上）

2. **Ed25519を使用**
   - RSAより高速で安全
   - 鍵サイズが小さい

3. **定期的に鍵をローテーション**
   - 年に1回程度、新しい鍵を生成
   - 古い鍵をGitHubから削除

4. **鍵を共有しない**
   - 秘密鍵（id_ed25519）は絶対に共有しない
   - 公開鍵（id_ed25519.pub）のみ共有

5. **複数デバイスで別々の鍵**
   - MacBook, デスクトップで別の鍵
   - Title で識別可能に

### ❌ DON'T

1. **秘密鍵をクラウドに保存**
   - Dropbox, Google Drive 等に保存しない
   - Git リポジトリに追加しない

2. **パスフレーズなしの鍵**
   - セキュリティリスク
   - 最低限のパスフレーズを設定

3. **古いRSA 2048bit**
   - 最低でも4096bit
   - できればEd25519

4. **同じ鍵を複数のサービスで使用**
   - GitHub, GitLab, Bitbucket で別々の鍵
   - または同じでもOK（個人判断）

---

## HTTPS認証との比較

| 項目 | SSH認証 | HTTPS認証 |
|------|---------|-----------|
| セットアップ | 初回のみ鍵生成 | Personal Access Token発行 |
| 使いやすさ | ✅ パスワード不要 | ⚠️ トークン管理必要 |
| セキュリティ | ✅ 鍵ベース | ⚠️ トークン管理 |
| 2FA対応 | ✅ スムーズ | ⚠️ 追加手順 |
| ファイアウォール | ⚠️ Port 22が必要 | ✅ Port 443（通過しやすい）|
| 企業環境 | ✅ 標準 | △ 環境による |
| 自動化 | ✅ 完全自動 | △ トークン埋め込み必要 |

**推奨**: SSH認証（ファイアウォール問題がない限り）

---

## 既存リポジトリをHTTPSからSSHに変更

既にHTTPSで設定している場合：

```bash
# 現在のリモートURLを確認
git remote -v

# HTTPSの場合:
# origin  https://github.com/username/compass.git (fetch)
# origin  https://github.com/username/compass.git (push)

# SSHに変更
git remote set-url origin git@github.com:username/compass.git

# 確認
git remote -v
# origin  git@github.com:username/compass.git (fetch)
# origin  git@github.com:username/compass.git (push)

# テスト
git fetch
```

---

## まとめ

### SSH認証セットアップチェックリスト

- [ ] SSH鍵を生成（Ed25519推奨）
- [ ] SSH-agentに鍵を追加
- [ ] GitHubに公開鍵を登録
- [ ] 接続テスト成功（`ssh -T git@github.com`）
- [ ] Kiro SDDで使用（`/kiro/git init git@github.com:username/repo.git`）

### 推奨設定

```bash
# 1. 鍵生成
ssh-keygen -t ed25519 -C "your_email@example.com"

# 2. SSH-agent設定（macOS）
cat << EOF >> ~/.ssh/config
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
EOF

# 3. 鍵追加
ssh-add --apple-use-keychain ~/.ssh/id_ed25519

# 4. 公開鍵をGitHubに登録
cat ~/.ssh/id_ed25519.pub
# → GitHub Settings → SSH Keys に追加

# 5. テスト
ssh -T git@github.com

# 6. Kiro SDDで使用
/kiro/git init git@github.com:username/compass.git
```

セットアップ完了！これで快適なGit操作が可能です 🎉

