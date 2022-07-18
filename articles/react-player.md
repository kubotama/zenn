---
title: "Cloud Storage for Firebaseにある音声ファイルをReact Playerで再生する"
emoji: "🔈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [React, Firebase]
published: true
---

## 概要

[Cloud Storage for Firebase](https://firebase.google.com/docs/storage)に音声ファイル(wav 形式や mp3 形式など)を
[react-player](https://www.npmjs.com/package/react-player)で再生します。
音声ファイルは[Firebase Authentication](https://firebase.google.com/docs/auth)でアクセスを制限しています。

<!--more-->

## リポジトリ

[kubotama/react-player](https://github.com/kubotama/react-player)

## 前提条件

Next.js でサイトを構築しています。

- node.js: 16.15.1
- Next.js: 12.2.0

詳しくは上記リポジトリで確認して下さい。

## 事前の準備

1. [Firebase](https://console.firebase.google.com/) でプロジェクトを作成します。
1. 作成したプロジェクトで Authentication を開始します。
1. 開始した Authentication でメール/パスワードプロバイダを有効にします。
1. Authentication にユーザーを追加します。
1. 上記リポジトリをローカル環境にクローンします。
1. 作成したプロジェクトにウェブアプリを追加して、「SDK の設定と構成」で「構成」を選択すると表示される「アプリのキーと ID が含まれている Firebase 構成オブジェクト:」をコピーして、ローカル環境にクローンしたリポジトリの./lib/firebase.js に貼り付けます。
1. firebaseConfig の変数宣言の先頭に export を追加します。
1. Cloud Storage for Firebase に音声ファイル(main.wav)をアップロードします。
1. Cloud Storage for Firebase のルールで、Authentication に追加したユーザーのみアクセスを許可します。

```json
// Grants a user access to a node matching their user ID
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if request.auth != null && request.auth.uid == "追加したユーザーのユーザーUID";
    }
  }
}
```

## プログラムの説明

アクセス可能なユーザーが制限されているため、音声ファイルにアクセスする前にログインします。
ログインしていなければログイン画面(login.tsx)、ログインしていればメイン画面(index.tsx)を表示します。

Authentication でアクセスを制限をされている Cloud Storage for Firebase にあるファイルにアクセスするためには、ダウンロード URL を取得する必要があります。

1. 上記リポジトリのディレクトリで`yarn dev`を実行すると、ローカル環境で検証用の web サイトが立ち上がりますので、<http://localhost:3000> にアクセスすると、ログインしていませんのでログイン画面が表示されます。

1. 事前の準備で Authentication に追加したユーザーのメールアドレスとパスワードを入力してログインすると、メイン画面(index.tsx)にログインしたユーザーのメールアドレスと、音声ファイルのダウンロード URL が表示されます。表示された react-player の再生ボタンをクリックすると音声ファイルが再生されます。

```tsx:index.tsx
const [downloadUrl, setDownloadUrl] = useState("");
  const filename = "main.wav";

  useEffect(() => {
    const authStateChanged = onAuthStateChanged(auth, async (user) => {
      if (user) {
        const storage = getStorage();
        getDownloadURL(ref(storage, filename)).then((url) => {
          setDownloadUrl(url);
        });
      }
      const email = user?.email || "no name";
      setName(email);
    });
    return () => {
      authStateChanged();
    };
  }, []);

```
