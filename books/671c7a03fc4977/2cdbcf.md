---
title: "Firebaseの初期設定"
free: false
---

## はじめに
この章では、FirebaseとNext.jsの接続、及び初期設定について説明します。
前提として、Googleアカウントは必須のため、持っていない方はアカウントを作成してください。

それでは、Firebaseの初期設定をしていきましょう。

## プロジェクトの作成
下記URLからFirebaseのコンソールにアクセスして、プロジェクトを作成していきます。
https://firebase.google.com/?hl=ja

下記の手順でプロジェクトの初期設定をしていきます。
1.「コンソールに移動する」を押下
2.「プロジェクトを追加」から新規プロジェクトを作成
3.プロジェクト名の入力（任意の名前で構いません。私は、「chat-app」としました。)
4.アナリティクスの有効化
5.アカウントの選択（Default Accountを選択しました）

これで新規プロジェクトが作成されます。

## Firebaseとの接続
Firebaseのプロジェクトが作成できたら、次は開発しているwebアプリからFirebaseにアクセスができるように設定します。

Firebaseにwebアプリの登録をします。
下記画像の赤丸で囲われたアイコンから手順を進めていきます。

![](https://storage.googleapis.com/zenn-user-upload/15eee97a215c-20230708.png)

ここでは、アプリ名を登録して、「アプリを登録」を押下します。
（Firebase Hostingについてチェック欄がありますが、使用しないので無効で問題ありません）
下記のような画面が出てきます。

![](https://storage.googleapis.com/zenn-user-upload/1eb95c495279-20230708.png)

これらのkeyはFirebaseとの接続に必須で、他人に知られると勝手にアクセスすることが可能になってしまうため、知られないように注意してください。
※コードに直書きしないようにしてください

取得したkeyを使用してFirebaseとプロジェクトの接続を行っていきます。
まず初めに、以下のパッケージをインストールします。

```
npm install firebase
npm install firebase-admin --save
```

次に、プロジェクトの1番上の階層に`.env`ファイルを作成します。
![](https://storage.googleapis.com/zenn-user-upload/746b9e7ce405-20230708.png)

このファイルは環境変数や、直にコードに記載しないもの、今回は、Firebaseのkeyなどを記載しておきます。これらのkeyは、vercelにデプロイするときもvercel側の環境変数として定義します。

```typescript: .env
# firebase
NEXT_PUBLIC_FIREBASE_API_KEY='your firebase apiKey'
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN='your firebase authDomain'
NEXT_PUBLIC_FIREBASE_PROJECT_ID='your firebase projectId'
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET='your firebase chat-app-c10ca.appspot.com'
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID='your firebase messagingSenderId'
NEXT_PUBLIC_FIREBASE_APP_ID='your firebase appId'
```
自身のFirebaseのkeyに対応する値を入力してください。

次に、この値をプロジェクト内で使用できるように設定します。
下記のフォルダに`config.ts`というファイルを作成します。
```
src /
　　└ lib /
	└ config.ts
```

`config.ts`ファイルで、Firebaseとの接続に関する設定を実装します。

```typescript: src/lib/config.ts
import { initializeApp } from 'firebase/app';

export const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
}

const app = initializeApp(firebaseConfig)
```

これで、Firebaseとの接続は完了です。
しかし、このままでは今回使用するAuthenticationとcloudFirestoreがまだ使用できません。
これらを使用できるようにコードを追加します。

```
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth'
import { getFirestore } from 'firebase/firestore'

export const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
}

const app = initializeApp(firebaseConfig)
// authを使えるように宣言
export const auth = getAuth(app)
// cloudFirestoreを使えるように宣言
export const db = getFirestore(app)
```

これで、Next.jsのプロジェクト側でAuthenticationとcloudFirestoreを使用することができます。
次は、Firebaseのコンソール側でもAuthenticationとcloudFirestoreが利用できるように設定しましょう。

## Authenticationの設定
今回、アプリのログインなどの認証周りは、Googleアカウントを使用してログインします。
まずは、Authenticationを有効化にします。
サイドメニューから「Authentication」を選択し、「始める」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/6e78e190b4c9-20230708.png)

さまざまな方法でログインすることができますね。今回はGoogleを有効化にするため、選択します。

![](https://storage.googleapis.com/zenn-user-upload/cc8668a40155-20230708.png)

有効化にして、サポートメールをFirebaseのデフォルトのメールを割り当て保存します。

![](https://storage.googleapis.com/zenn-user-upload/54ce10b991d6-20230708.png)

これで、Firebase側のAuthenticationの設定は完了です。

![](https://storage.googleapis.com/zenn-user-upload/d057bee3eeb2-20230708.png)

ユーザがアプリを通じて、登録すると、「Users」タブの一覧に追加されていきます。

![](https://storage.googleapis.com/zenn-user-upload/bc32423b3730-20230708.png)

## cloudFirestoreの設定
次に、データベースとして使用するcloudFirestoreの設定に移ります。
cloudFirestoreの利用を有効化にします。
サイドメニューから「Firestore Database」を選択し、「データベースの作成」を押下します。

![](https://storage.googleapis.com/zenn-user-upload/5992db2ff4c2-20230708.png)

今回は「本番環境モード」にして実装を進めていきましょう。

![](https://storage.googleapis.com/zenn-user-upload/054391cc5b02-20230708.png)

ロケーションは「asia-northeast1 (Tokyo)」を選択して有効にします。

![](https://storage.googleapis.com/zenn-user-upload/944ec8b92533-20230708.png)

また、このタイミングでセキュリティルールもフルアクセスできるように設定しておきます。
ルールタブから、`if false`から`if true`にして「公開」ボタンを押下します。

![](https://storage.googleapis.com/zenn-user-upload/992c705a538c-20230708.png)

※本書ではセキュリティルールの説明は省きますが、サービスを公開、運用するときには、適切なセキュリティルールを記載してください。フルアクセスができてしまうと、本来ユーザ情報は本人にだけ参照・更新ができるようにしたいが、他人が見えて更新もできてしまうなどの問題が発生します。

これでFirebase側の初期設定は完了です。画面側の実装をして、うまく接続ができているかを確認しましょう。