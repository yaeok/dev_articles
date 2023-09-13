---
title: "ホーム画面の実装"
free: false
---

## はじめに
この章では、ホーム画面について説明します。
ホーム画面では、以下の機能を搭載した画面を実装します。
- 友達一覧を表示
　⇒友達の名前を押下したらチャット画面に遷移する
- 友達追加ボタンの表示
　⇒友達検索・追加画面に遷移する

それでは実装にうつっていきましょう。

## ホーム画面のレイアウト
まずは、ホーム画面を実装するファイルを作成していきます。
下記のフォルダに`page.tsx`ファイルを作成します。
```
src /
　　└ app /
	└ home /
		└ page.tsx
```

友達一覧の表示については、今回`List`形式で実装します。
chakra-uiには、`Table`や`Stack`、`Accordion`など様々なコンポーネントがあります。
今回は一覧を表示し、タップしたら遷移するというシンプルな実装のため、`List`を使用します。
配列を一覧で表示する処理はそこまで差が無いので、追加したい機能や要件によっては他のコンポーネントを使用しても構いません。

まだ、Firebaseと接続をしていないため、ダミーのデータを使って一覧表示を実装していきます。
以下のデータをダミーデータとして、定義してください。

```
const dummyFriend = [
  {
    chatId: 'chatId123', // チャットルームのId
    uid: 'uid123', // ユーザId
    username: 'Aさん', // ユーザ名
  },
  {
    chatId: 'chatId234', // チャットルームのId
    uid: 'uid234', // ユーザId
    username: 'Bさん', // ユーザ名
  },
]
```

画面側は以下のコードで実装します。

```typescript: home/page.tsx
'use client'

import { useRouter } from 'next/navigation'
import { useState } from 'react'

import { Divider, List, ListItem } from '@/common/design'

// ダミーデータ省略

export default function HomeScreen() {
  const router = useRouter()
  const [friends] = useState(dummyFriend)

  return (
    <List>
      {friends.map((friend, index) => (
        <>
          <ListItem
            key={index}
            paddingY='5px'
            onClick={() => router.push(`chat/${friend.chatId}`)}
          >
            {friend.username}
          </ListItem>
          <Divider />
        </>
      ))}
    </List>
  )
}
```

一度、画面を確認してみましょう。
urlは`localhost:3000/home`で画面を表示します。

![](https://storage.googleapis.com/zenn-user-upload/c8b28cfbcdde-20230708.png)

「Aさん」「Bさん」の名前がリストで表示されました。
しかし、ホーム画面にヘッダーとフッターが表示されておらず、メイン画面のレイアウトも反映されていません。

ホーム画面にはヘッダー、フッターを表示させたいので、`home`フォルダ直下に`layout.tsx`ファイルを作成し、レイアウトを整えていきましょう。

```typescript: home/layout.tsx
import Footer from '@/common/components/footer.component'
import Header from '@/common/components/header.component'
import Main from '@/common/components/main.component'

export default function HomeLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <>
      <Header />
      <Main>{children}</Main>
      <Footer />
    </>
  )
}
```

画面を確認します。
![](https://storage.googleapis.com/zenn-user-upload/422d4b320805-20230708.png)

ヘッダーとフッター、そしてメイン画面は左右にスペースができました！

## 遷移先画面の実装
次に、遷移先の画面の実装にうつります。
完成形のチャットアプリでは、友達一覧から対象の友達をタップしたら、チャット画面に遷移します。
この章では、ダミー画面として仮実装しておきましょう。
そのため、ホーム画面からダミーのチャット画面へ遷移し、チャット画面では誰とのチャットなのかがわかるように、`chatId`を画面に表示させておきます。

下記のフォルダに`page.tsx`ファイルを作成します。
```
src /
　　└ app /
	└ chat /
		└ [chatId] /
			└ page.tsx
```

`[chatId]`この形式のフォルダ名は、一覧画面から詳細画面に遷移する時に使用され、画面間の値渡しとして使用されます。
urlのパスは`localhost:3000/chat/12345`のように`chat/`のあとは、Idなどの動的な値がはいります。

ダミーのチャット画面は以下のコードで実装します。

```typescript: src/app/chat/[chatId]/page.tsx
'use client'

type Props = {
  param: {
    chatId: string
  }
}

export default function ChatScreen({ param }: Props) {
  return (
    <>
      <p>chatId: {param.chatId}</p>
    </>
  )
}
```

`type Props`では、一覧画面から渡されたPropsを受け取ります。
例）
`localhost:3000/chat/12345`というurlの場合、`12345`が`chatId`にあたり、`param.chatId`は`12345`が表示されます。
遷移時のPropsを渡す際に使用されます。

今の状態では、ヘッダー・フッターが定義されている`layout.tsx`ファイルを継承していないため、ヘッダー・フッターが表示されません。
![](https://storage.googleapis.com/zenn-user-upload/23be85eec153-20230708.png)

`chat`フォルダ直下にもレイアウトを反映させる必要があります。
そのため、`/src/app/chat`フォルダ直下に、`layout.tsx`ファイルを作成します。

```
src /
　　└ app /
	└ chat /
		└ layout.tsx
```

```typescript: /src/app/chat/layout.tsx
import Footer from '@/common/components/footer.component'
import Header from '@/common/components/header.component'
import Main from '@/common/components/main.component'

export default function ChatLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <>
      <Header />
      <Main>{children}</Main>
      <Footer />
    </>
  )
}
```

`[chatId]`フォルダの画面は作成した`layout.tsx`を継承します。
画面を確認してみましょう。
![](https://storage.googleapis.com/zenn-user-upload/688cc7cde726-20230708.png)

レイアウトが反映されたことが確認できました。
`chatId`が正しく画面に渡されているか確認してみましょう。
ホーム画面から、「Aさん」を押下すると、`chatId123`と画面に表示され、「Bさん」を押下すると、`chatId234`と表示されればOKです！