+++
title = 'Complex relational App by, ReactJS + Firestore + ServerLess'
image = 'images/firestorenextjsserverless/code.jpg'
date = '2018-08-06T0:00:00'
description = 'Serverless Application with ReactJS + Cloud Firestore deal with complex relational data, To improved atomic design in real application.'
disableComments = false
+++

## Intro

As the need of serverless app group, client side have to deal with relational data either sql or nosql. 

i try to find best practice, but there is no good tutorial for complex relational nosql handle method which is needed by firestore/firebase to handle all data only by it.

## Problem

> we assume backend support by firestore to achieve server-less.

* Manage Relational data with Server-less application is painful.
* firestore only support simple query.
* Javascript is running async with data query.
* ReactJS Component's code will grow too big, with more relational data.

### example

#### for sql

we assume there are some data relation like this:
`user -> wallets -> cards`

if we want to query the `cards` of user what is the best solution.
if we using sql, it might like this.

```sql
select * from cards c
inner join wallets w
n w.id = c.wallet_id
inner join user u
on u.id = w.user_id
;
```

#### for nosql

if we using firestore, we will get two way to store data. flatten and nested show like below

##### flat data storage

```json
{
  "users": [
    {
      "id": "xxx",
    }
  ],
  "wallet": [
    {
      "id": "yyy",
      "user_id": "xxx",
    }
  ],
  "cards": [
    {
      "id": "zzz",
      "wallet_id": "yyy",
      "name": "kkk",
    }
  ]
}
```

query will be

```javascript
firestore.collection('wallets')
	.where('user_id', '==', user.id)
	.get()
	.then(collections => {
		collections.forEach(collection => {
			firestore.collection('cards')
				.where('wallet_id', '==', collection.id)
				.get()
				.then(doc => {
					console.log(`card is: ${doc.data().id}`)
				})
		})
	})
```

##### nested data storage

```json
{
  "users": [
    {
      "id": "xxx",
      "wallets": [
        {
          "id": "yyy",
          "cards": [
            {
              "id": "zzz",
              "name": "kkk"
            }
          ]
        }
      ]
    }
  ]
}
```

query will be

```javascript
firestore.collection('users')
	.doc(user.id)
	.collection('wallet')
	.get()
	.then(collections => {
		collections.forEach(collection => {
			firestore.collection('cards')
				.where('wallet_id', '==', collection.id)
				.get()
				.then(doc => {
					console.log(`card is: ${doc.data().id}`)
				})
		})
	})
```

so, as it shows 3 level of relation data will create this big query at firestore, what if it comes 4 level. And with atomic design, Component have to remain small and less dependency.

## Solution

With the idea of atomic design, what if i can split query part from component with HOC. And in advance split query with HOC at each level.

### example

```javascript
// user has many wallets
// HOCao/queryUserHasManyWallets.jsx
const queryUserHasManyWallets = (BaseComponent) {

	type Props = {
		user_id: string,
		...any,
	}

	class HOC extends Component{
		render() {
			return (
				<BaseComponent
					{...props}
				/>
			)
		}
	}

	const mapStateToProps = state => {
		return {
			wallets: state.firestore.ordered.wallets,
		}
	}

	return compose(
		firebaseConnect(props => [
			{
				collection: 'wallets',
				where: ['user_id', '==', props.user_id]
				storeAs: 'wallets'
			}
		]),
		connect(mapStateToProps, null),
	)(HOC);
}

export default queryUserHasManyWallets;
```

### explain

like example, we using HOC to handle each query in each level, with split function which means we can create query HOC and Render Component separately. which will enhance atomic design with non-state component that can get props from HOCs.

### A sample of list by previous architecture

#### file structure

```
// HOC Query for general query
HOCao/queryUserHasManyWallets.jsx
HOCao/queryWalletHasManyCards.jsx

// HOC Query for special cases
qhoc/userCardsList.jsx

// Atomic Design Component
organisms/listBody.jsx
molecules/list.jsx
```

#### sample code

```javascript
// https://github.com/prescottprue/react-redux-firebase
import {firebaseConnect, isEmpty} from 'react-redux-firebase';
import { connect } from 'react-redux';
import compose from 'recompose/compose';
import {map} from 'lodash';

// HOCao/queryUserHasManyWallets.jsx
const queryUserHasManyWallets = (BaseComponent) {

	type Props = {
		user_id: string,
		...any,
	}

	class HOC extends Component{
		render() {
			return (
				<BaseComponent
					{...props}
				/>
			)
		}
	}

	const mapStateToProps = state => {
		return {
			wallets: state.firestore.ordered.wallets,
		}
	}

	return compose(
		firebaseConnect(props => [
			{
				collection: 'wallets',
				where: ['user_id', '==', props.user_id]
				storeAs: 'wallets'
			}
		]),
		connect(mapStateToProps, null),
	)(HOC);
}

export default queryUserHasManyWallets;

// HOCao/queryWalletHasManyCards.jsx
const queryWalletHasManyCards = (BaseComponent) {

	type Props = {
		wallet_id: string,
		...any,
	}

	class HOC extends Component{
		render() {
			return (
				<BaseComponent
					{...props}
				/>
			)
		}
	}

	const mapStateToProps = state => {
		return {
			cards: state.firestore.ordered.cards,
		}
	}

	return compose(
		firebaseConnect(props => [
			{
				collection: 'cards',
				where: ['wallet_id', '==', props.wallet_id]
				storeAs: 'cards'
			}
		]),
		connect(mapStateToProps, null),
	)(HOC);
}

export default queryWalletHasManyCards;

// molecules/list.jsx
type Props = {
	user_id: string,
	wallets: any,
	cards: any,
	...any,
}

const List = enhance(({cards}: Props) => {
	return (
		<ul>
			{
				!isEmpty(cards) &&
				_.map(cards, (c) => {
					return(
						<li>{c.id}</li>
					)
				})
			}
		</ul>
	)
});

export default List;

// qhoc/userCardsList.jsx
import List from 'molecules/list';
import queryUserHasManyWallets from 'HOCao/queryUserHasManyWallets';
import queryWalletHasManyCards from 'HOCao/queryWalletHasManyCards';

const enhance = compose(
	// add more qHOC if needed
	queryUserHasManyWallets,
	queryWalletHasManyCards
);

export default enhance(List);

// organisms/listBody.jsx
import HOCList from 'qhoc/userCardsList';
import Head from 'molecules/head';

const ListBody = () => {
	return (
		<Head />
		<HOCList
			user_id={"xxx"}
		>
	)
}

export default ListBody;
```

### Props

* enhance atomic design with Nosql Server interaction.
* easy to manage files with more efficient way.
* split Query part with design part, lower the cost.

### Cons

* Will case slow rendering issue, case by async query.
* Many query will run each time, when state change.

### Some advice

* with this kind of design, firestore should not using nested too much. because nested data will make too many dependency issue. and it will against the idea of atomic design.

### sample code can be find at:

[https://github.com/kingno21/firestore-react-ssr#master](https://github.com/kingno21/firestore-react-ssr#master)
