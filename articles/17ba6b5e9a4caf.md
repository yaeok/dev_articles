---
title: "cloud functionsのトリガーを実装してみる"
emoji: "🗂"
type: "tech"
topics:
  - "nextjs"
  - "cloudfunctions"
  - "cloudfirestore"
  - "firebasefunctions"
  - "firebaseauth"
published: true
published_at: "2023-08-19 15:33"
---

# はじめに
最近firebaseにはまっているyappiです。
今回は、cloud functionsでトリガーについて、便利だなと思ったので、記事に残しておこうと思います。
firebase authってめちゃくちゃ便利で、よく使用するんですが、私は、authに登録したら、とりあえずfirestoreにもユーザ情報を登録しようみたいな流れになっちゃうんですね。
私だけなのか、あるあるなのか。。まぁそれはさておき。
いつもGoogle認証だけで実装していたりするんですが、Email／Passwordも追加する機会があり、その時にfunctionsを使った方が便利だなと思ったので、記事にまとめてみようかと思います！

# なぜトリガーを使ったか？
私は普段、Next.js × firebaseで個人開発してまして、基本Google認証完了後、取得したユーザデータを使用して、firestoreにユーザ情報を格納するみたいな流れで実装をしてました。

```typescript
/**
 * Googleアカウントでサインイン
 * @returns Promise<FirebaseResult>
 */
export const signInWithGoogle = async (): Promise<FirebaseResult> => {
  let result: FirebaseResult = { isSuccess: false, message: '' }
  const provider = new GoogleAuthProvider()
  try {
    const user = await signInWithPopup(auth, provider)
    const docRef = doc(db, 'users', user.user.uid)
    const docSnap = await getDoc(docRef)
    if (!docSnap.exists()) {
      setDoc(docRef, {
        uid: user.user.uid,
        username: user.user.displayName,
        email: user.user.email,
        photoURL: user.user.photoURL,
        createdAt: serverTimestamp(),
      })
    }
    if (user) {
      result = { isSuccess: true, message: 'ログインに成功しました' }
    }
  } catch (error) {
    result = { isSuccess: false, message: 'ログインに失敗しました' }
  }
  return result
}
```

しかし、Email／Password認証も追加で実装するとなると、これまでGoogle認証では取得できていた、`displayName`や`photoURL`を取得できなくなることに気づきました。
そして、なんといっても、firestoreへの登録処理を再度実装しないといけないのと、ユーザの項目を追加しようかと考えたら、どっちにも追加しないといけないなぁとか、、（共通化するっていう手もあったな・・って書いてて思いました）

ということがあって、cloud functionsでトリガー作っちゃおう！functionsも勉強したかったし、ちょうどいい！ということで、authにユーザが登録されたら、これらの項目が追加されるように設定しました。

# トリガーの実装と妥協
半分興味と勉強がてらと思ってトリガーを実装しようと思いましたが、`displayName`と`photoURL`は取れないので、そこは妥協しました。
とりあえず、`username`は、Emailの`@`の前の部分を登録して、`photoURL`はユーザアイコンを使用している部分は、`null`の場合は、デフォルトでアイコンが表示されるようにしておきました。

functionsの初期設定は、以前記事におこしたので、それを参考にしていただける幸いです。
[functionsの初期設定](https://qiita.com/yappi-dev/items/807040e7c71dccc91b97)

コードの内容は以下になります。
```typescript
/** Authにユーザが新規登録されたときに動作する処理  */
const registerUserTriggerFromAuth = functions
  .region('asia-northeast1')
  .auth.user()
  .onCreate(async (user) => {
    try {
      await usersColRef.doc(user.uid).set({
        uid: user.uid,
        username:
          user.displayName || user.email?.split('@')[0] || 'ユーザ名未設定',
        email: user.email,
        photoURL: user.photoURL || null,
        createdAt: admin.firestore.FieldValue.serverTimestamp(),
      })
      return
    } catch (error) {
      console.error(error)
      return
    }
  })
```

authにユーザが登録されたら動作する関数は、`onCreate()`です。
これで、Google認証またはEmail／Password認証どちらで新規登録しても同じようにfirestoreにユーザ情報が登録されます！ちょっと便利になった気がします。
最近はこのトリガーを使って、自分のサービスにユーザが新規登録されたら、メールが飛ぶように設定しました。
わざわざfirebaseを開かずとも、ユーザが増えていることを確認することができます！
（そんなに利用ユーザがいないので、functionsの使用量もそこまで気になりませんが、、良いのか悪いのか・・汗）

# 終わりに
今回は、cloud functionsのトリガーについて記事をまとめました。
次回は、お問合せ内容を登録したら、メールで通知がいくようなトリガーや頻度を設定したバッチ処理など、cloud functionsを使用した処理についてまとめていきたいと思います。

この記事が何かの役に立てると幸いです。間違い等ございましたら、ご指摘等、お願いします。
