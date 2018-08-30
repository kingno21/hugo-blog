+++
title = "Don't use firestore/firebase at ReactJS Update cycle"
image = 'images/firestorereactupdatecycle/code.png'
date = '2018-08-30T0:00:00'
description = 'make a query inside of update cycle will break reactjs normal lifecycle.'
disableComments = false
tags = ['firestore', 'nosql', 'react', 'reactjs', 'nextjs', 'update', 'async']
+++

## Problem

in case when i create chat application. for morden design of chat, we always render 30 messages at once and scroll to Top to loading more messages. when i create firestore base application i have to deal limitation by locally because of SSR. 

The easiest way is to using limits at firestore query. which seems like.

```javascript
firestore.collection('messages')
    .limit(dynamicLimit)
    .snapShot()
```
> sample below is short handle, check [Firestore Query](https://firebase.google.com/docs/firestore/query-data/queries) for more detail.

For ReactJS lifecycle, change `limit` is done by change `props/state` to trigger update cycle. So i try to put this query inside of `componentWillUpdate`, but this case a big performance problem which is obvious because query is slow down render process. 

So, normally we will think about async process, so i try

```javascript
async componentWillUpdate() {
    await firestore...
            .limit(dynamicLimit)
            .snapShot(doc => {
                this.setState({...})
                return true
            })
}
```

its works, but i work too mush every time `props/state` runs will trigger this query. so what i try is using `shouldComponentUpdate` which will let us decide re-render or not. the code is look like this.

```javascript
async shouldComponentUpdate() {
    if (...) return false
    await firestore...
            .limit(dynamicLimit)
            .snapShot(doc => {
                this.setState({...})
                return true
            })
    return true
}
```

yes, it work at first. but when i debug, there is something bad happened. `shouldComponentUpdate` is `return true` even query is not finish. and `shouldComponentUpdate` is triggered multi times. Which means async process is `return true` anyway. and render process to child nodes is still running.

So the reason why it run is case by inner state change of React Lifecycle. 

![img](/images/firestorereactupdatecycle/image01.png)


like image below, ansyc process is running even query is not finish so `setState` is trigger an other `shouldComponentUpdate` and `render` multi times and query is made by every time it triggered and only way to stop this cycle is `return false` which is condition by `if`, so either `shouldComponentUpdate` and `componentWillUpdate` is not good to handle `firestore`.

## Solution

1. move query to `componentDidMount/componentWillMount/contructor/getInitialProps`
2. make HOC to handle all changes
3. split query from `react` application
