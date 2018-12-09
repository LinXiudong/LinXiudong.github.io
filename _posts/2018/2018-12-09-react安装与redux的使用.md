---
layout: mypost
title: react安装与redux的使用
categories: [react]
---

# 通过create-react-app安装

先装脚手架

> npm install -g create-react-app

初始化项目

> create-react-app 你的项目名

cd进项目，运行

> yarn start


# 安装和使用redux

[来源](https://segmentfault.com/a/1190000011474522#articleHeader3)

> yarn add redux 或 npm install redux

## redux使用

删除src文件夹中除index.js以外的所有文件。打开index.js，删除所有代码，键入以下内容：
```
//从redux包中引入createStore()方法
import { createStore } from "redux";

//创建名为reducer的方法。
//第一个参数state是当前保存在store中的数据，第二个参数action是一个容器，用于：
//type - 一个简单的字符串常量，例如ADD, UPDATE, DELETE等。
//payload - 用于更新状态的数据。
const reducer = function(state=[], action) {
    return state;
}

//创建一个Redux存储区，它只能使用reducer作为参数来构造。
//存储在Redux存储区中的数据可以被直接访问，但只能通过提供的reducer进行更新。
const store = createStore(reducer);
```

如何使用多个reducer？我们将用到Redux包中提供的combineReducers函数。修改代码如下：
```
import { createStore } from "redux";
import { combineReducers } from 'redux';

const productsReducer = function(state=[], action) {
    return state;
}

const cartReducer = function(state=[], action) {
    return state;
}

const allReducers = {
    products: productsReducer,
    shoppingCart: cartReducer
}

const rootReducer = combineReducers(allReducers);

let store = createStore(rootReducer);
```

定义数据
```
// …
const initialState = {
    cart: [
        {
            product: 'bread 700g',
            quantity: 2,
            unitCost: 90
        },
        {
            product: 'milk 500ml',
            quantity: 1,
            unitCost: 47
        }
    ]
}

const cartReducer = function(state=initialState, action) {
    return state;
}

//…

let store = createStore(rootReducer);

//可输出上面定义的参数
console.log("initial state: ", store.getState());
```

添加“加入购物车”方法
```
//…

const ADD_TO_CART = 'ADD_TO_CART';

const cartReducer = function(state=initialState, action) {
    switch (action.type) {
        case ADD_TO_CART: {
            return {
                ...state,
                cart: [...state.cart, action.payload]
            }
        }

        default:
          return state;
    }
}

//…
```

定义一个action，作为store.dispatch()的一个参数。action是一个Javascript对象，有一个必须的type和可选的payload。我们在cartReducer函数后定义一个：
```
//…
function addToCart(product, quantity, unitCost) {
    return {
        type: ADD_TO_CART,
        payload: { product, quantity, unitCost }
    }
}
//…
//在这里，我们定义了一个函数，返回一个JavaScript对象。在我们分发消息之前，我们添加一些代码，让我们能够监听store事件的更改。
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

unsubscribe();

//接下来，我们通过分发消息到store来向购物车中添加商品。将下面的代码添加在unsubscribe()之前：
store.dispatch(addToCart('Coffee 500gm', 1, 250));
store.dispatch(addToCart('Flour 1kg', 2, 110));
store.dispatch(addToCart('Juice 2L', 1, 250));
```

下面是整个index.js文件：
```
import { createStore } from "redux";
import { combineReducers } from 'redux';

const productsReducer = function(state=[], action) {
    return state;
}

const initialState = {
    cart: [
        {
            product: 'bread 700g',
            quantity: 2,
            unitCost: 90
        },
        {
            product: 'milk 500ml',
            quantity: 1,
            unitCost: 47
        }
    ]
}

const ADD_TO_CART = 'ADD_TO_CART';

const cartReducer = function(state=initialState, action) {
    switch (action.type) {
        case ADD_TO_CART: {
            return {
                ...state,
                cart: [...state.cart, action.payload]
            }
        }

        default:
            return state;
    }
}

function addToCart(product, quantity, unitCost) {
    return {
        type: ADD_TO_CART,
        payload: {
            product,
            quantity,
            unitCost
        }
    }
}

const allReducers = {
    products: productsReducer,
    shoppingCart: cartReducer
}

const rootReducer = combineReducers(allReducers);

let store = createStore(rootReducer);

console.log("initial state: ", store.getState());

let unsubscribe = store.subscribe(() =>
    console.log(store.getState())
);

store.dispatch(addToCart('Coffee 500gm', 1, 250));
store.dispatch(addToCart('Flour 1kg', 2, 110));
store.dispatch(addToCart('Juice 2L', 1, 250));

unsubscribe();
```

## 组织Redux代码

index.js中的代码逐渐变得冗杂。接下来，我们来看一下如何组织Redux项目。
![p01][01]
```
// src/actions/cart-actions.js
export const ADD_TO_CART = 'ADD_TO_CART';

export function addToCart(product, quantity, unitCost) {
  return {
    type: ADD_TO_CART,
    payload: { product, quantity, unitCost }
  }
}

// src/reducers/products-reducer.js
export default function(state=[], action) {
  return state;
}

// src/reducers/cart-reducer.js
import  { ADD_TO_CART }  from '../actions/cart-actions';

const initialState = {
  cart: [
    {
      product: 'bread 700g',
      quantity: 2,
      unitCost: 90
    },
    {
      product: 'milk 500ml',
      quantity: 1,
      unitCost: 47
    }
  ]
}

export default function(state=initialState, action) {
  switch (action.type) {
    case ADD_TO_CART: {
      return {
        ...state,
        cart: [...state.cart, action.payload]
      }
    }

    default:
      return state;
  }
}

// src/reducers/index.js
import { combineReducers } from 'redux';
import productsReducer from './products-reducer';
import cartReducer from './cart-reducer';

const allReducers = {
  products: productsReducer,
  shoppingCart: cartReducer
}

const rootReducer = combineReducers(allReducers);

export default rootReducer;

// src/store.js
import { createStore } from "redux";
import rootReducer from './reducers';

let store = createStore(rootReducer);

export default store;

// src/index.js
import store from './store.js';
import { addToCart }  from './actions/cart-actions';

console.log("initial state: ", store.getState());

let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

store.dispatch(addToCart('Coffee 500gm', 1, 250));
store.dispatch(addToCart('Flour 1kg', 2, 110));
store.dispatch(addToCart('Juice 2L', 1, 250));

unsubscribe();
```

现在我们来添加修改和删除购物车中商品的逻辑。修改cart-actions.js和cart-reducer.js文件：
```
// src/reducers/cart-actions.js
//…
export const UPDATE_CART = 'UPDATE_CART';
export const DELETE_FROM_CART = 'DELETE_FROM_CART';
//…
export function updateCart(product, quantity, unitCost) {
  return {
    type: UPDATE_CART,
    payload: {
      product,
      quantity,
      unitCost
    }
  }
}

export function deleteFromCart(product) {
  return {
    type: DELETE_FROM_CART,
    payload: {
      product
    }
  }
}

// src/reducers/cart-reducer.js
//…
export default function(state=initialState, action) {
  switch (action.type) {
    case ADD_TO_CART: {
      return {
        ...state,
        cart: [...state.cart, action.payload]
      }
    }

    case UPDATE_CART: {
      return {
        ...state,
        cart: state.cart.map(item => item.product === action.payload.product ? action.payload : item)
      }
    }

    case DELETE_FROM_CART: {
      return {
        ...state,
        cart: state.cart.filter(item => item.product !== action.payload.product)
      }
    }

    default:
      return state;
  }
}
```

最后，我们在index.js中分发这两个action：
```
// src/index.js
//…
// Update Cart
store.dispatch(updateCart('Flour 1kg', 5, 110));

// Delete from Cart
store.dispatch(deleteFromCart('Coffee 500gm'));
//…
```


[01]: 01.png
