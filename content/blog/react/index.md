---
title: オブジェクトを使った条件分岐
date: "2020-09-28"
description: "オブジェクトを使った条件分岐"
tags: ["react", "Typescript"]
---

React でアプリを作ってます。
7 月から作り続けて、６０％程度出来上がってきました。
なんとか、10 月までには完成させたい。。。

コードを書いていると条件分岐が結構でてきますよね。

「if」や「?:」を使っていましたが、もう一段レベルアップしたく、
object による分岐を意識しています。

下記は、form コンポーネントを
「login」の場合と「signUp」の場合とで
object を使って分岐させた例です。

```ts

const status = "login" | "signUp"
 <Form status={status} />


Form.tsx


interface Props {
  status: 'signUp' | 'login';
}

const formDatas = {
  signUp: {
  },
  login: {
  },
};
const buttonValue = {
  signUp: 'SIGN UP',
  login: 'LOGIN',
};
const onSubmit = {
    signUp: () => {
    },
    login: () => {
    },
  };
const SignLoginForm: React.FC<Props> = ({ status }) => {
  <form onSubmit={onSubmit[status]}>
      {Object.entries(formDatas[status]).map(([key,value]: string[], num: number) => {
        return (
          <React.Fragment key={num}>
              <SignLoginItem valueKey={key}  value={value}/>
          </React.Fragment>
        );
      })}
      <button type="submit">{buttonValue[status]}</button>
    </form>
}

```
