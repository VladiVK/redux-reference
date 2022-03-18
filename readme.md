# Redux Reference

## Теория по "старой части" до `toolKit`

Ставим не только сам `redux` но и именно `react-redux`, что последний связал

состояние с реакт-компонентами

`npm i redux react-redux`

Есть состояние, оно измненяется исходя из задач, которые формулируются в `action`.

Но напрямую обратится нельзя. Нужен диспетчер `dispatch`, который примет заявку на обращение к

состоянию от экшенов. Но и диспетчер только принимает-передает заявку. Следующий шаг - это редьюсер

`reducer`. Вся логика работы с данными - это задача именно редьюсера!

1. Диспетчер принимает экшены и передает их редьюсеру

2. Редьюсер знает все возможные экшены и как менять состояние исходя из этого

Сначала при помощи функции `createStore` надо создать объект `store`.

Это объект, содержащий несколько методов:

1. получить состояние

2. `dispatch`

3. подписаться на изменение состояния

```
index.js

import { Provider } from 'react-redux';
import { createStore } from 'redux';

const defaultState = {
  cash: 0,
};
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD_CASH':
      return { ...state, cache: state.cash + action.payload };
    case 'GET_CASH':
      return { ...state, cache: state.cash - action.payload };
    default:
      return state;
  }
};

const store = createStore(reducer);
```

Все приложение оборачиваем в провайдер, чтобы обеспечить состоянием все компоненты.

```
ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);
```

Вернемся в приложение:

```
App:

// чтобы менять состояние нужен диспатч
 const dispatch = useDispatch();

// чтобы получить состояние

  const cash = useSelector((state) => state.cash);

  const addCash = (sum) => {
    dispatch({ type: 'ADD_CASH', payload: sum });
  };
  const getCash = (sum) => {
    dispatch({ type: 'GET_CASH', payload: sum });
  };

return ()

    <button className='btn' onClick={() => addCash(Number(prompt()))}>
        Top up account
    </button>
     <button className='btn' onClick={() => getCash(Number(prompt()))}>
        Withdraw money
    </button>

```

## Декомпизация и несколько редьюсеров

Удобнее создать на этом этапе папу `store`:

А в ней:

`index.js`

`cashReducer.js`

`customReducer.js`

```
index.js:

import { createStore, combineReducers } from 'redux';
import { cashReducer } from './cashReducer';
import { customersReducer } from './customReducer';

// можно просто редьюсеры предавать,
// а можно и ключ-значение
const rootReducer = combineReducers({
  cash: cashReducer,
  customers: customersReducer,
});

export const store = createStore(rootReducer);

```

```
const defaultState = {
  cash: 0,
};
export const cashReducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD_CASH':
      return { ...state, cash: state.cash + action.payload };
    case 'GET_CASH':
      return { ...state, cash: state.cash - action.payload };
    default:
      return state;
  }
};

```

```
const defaultState = {
  customers: [],
};

export const customersReducer= (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD_CUSTOMER':
      return { ...state, customers: [...state.customers, action.payload] };
    case 'GET_ALL_CUSTOMERS':
      return [...state.customers];
    default:
      return state;
  }
};

```

Теперь в основно корневом индексе:

```
import { Provider } from 'react-redux';
import { store } from './store/index';


ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

App:

Все то же самое, кроме необходимости глубже зайти: `store` -> `reducer` -> `значение`

```
store = {
  cash: {cash:0},
  customers: {customers: []}
}

```

```
  const cash = useSelector((state) => state.cash.cash);
```

## Второй параметр для createStore и отслеживание состояния

Для удобства разработки нужно использовать что-то для отслеживания состояния!

Поэтому вторым параметром для `createStore()` можно передать как `middleware` так и

инструменты разработчика!

Чтобы использовать middleware вместе с инструментами разработчика надо установить

`npm i @redux-devtools/extension`

Расширение для браузера `redux devtools` тоже должно стоять уже!

```
import { createStore, combineReducers} from 'redux';
import { cashReducer } from './cashReducer';
import { customersReducer } from './customReducer';
import { composeWithDevTools } from '@redux-devtools/extension';


const rootReducer = combineReducers({
  cash: cashReducer,
  customers: customersReducer,
});

export const store = createStore(rootReducer, composeWithDevTools());
```

## Оптимизация `actions`

1. все `actions` вынести в константы

```
const defaultState = {
  customers: [],
};
const ADD_CUSTOMER = 'ADD_CUSTOMER';
const REMOVE_CUSTOMER = 'REMOVE_CUSTOMER';

export const customersReducer = (state = defaultState, action) => {
  switch (action.type) {
    case ADD_CUSTOMER:
      return { ...state, customers: [...state.customers, action.payload] };
    case REMOVE_CUSTOMER:
      return {
        ...state,
        customers: [
          ...state.customers.filter(
            (customer) => customer.id !== action.payload
          ),
        ],
      };
    default:
      return state;
  }
};
```

2. Создать креэйторы `actions`, чтобы не прописывать в `dispatch` каждый раз новый объект:

В этом же файле:

```
export const addCustomerAction = (payload) => ({
  type: ADD_CUSTOMER,
  payload: payload,
});
export const removeCustomerAction = (payload) => ({
  type: REMOVE_CUSTOMER,
  payload: payload,
});

```

Теперь вернемся в App:

```
import {
  addCustomerAction,
  removeCustomerAction,
} from './store/customersReducer';


const addCustomer = (name) => {
    if (name.trim().length) {
      const newCustomer = { name: name, id: new Date().toISOString() };
      dispatch( addCustomerAction(newCustomer) );
    }
  };
const removeCustomer = (customer) => {
    dispatch( removeCustomerAction(customer.id) );
  };

```

## Работа с асинхронным кодом в `Redux`: модуль`redux-thunk`

```
npm i redux-thunk
```

Модуль`redux-thunk` - это `middleware (промежуточное ПО)`

Подключим его к нашему состоянию:

```

import { createStore, combineReducers, applyMiddleware } from 'redux';
import { cashReducer } from './cashReducer';
import { customersReducer } from './customersReducer';
import { composeWithDevTools } from '@redux-devtools/extension';
import thunk from 'redux-thunk';

const rootReducer = combineReducers({
  cash: cashReducer,
  customers: customersReducer,
});

export const store = createStore(
  rootReducer,
  composeWithDevTools(applyMiddleware(thunk))
);


```

Измение `customersReducer`:

```
const defaultState = {
  customers: [],
};
const ADD_CUSTOMER = 'ADD_CUSTOMER';
const REMOVE_CUSTOMER = 'REMOVE_CUSTOMER';

const ADD_MANY_CUSTOMERS = 'ADD_MANY_CUSTOMERS';

export const customersReducer = (state = defaultState, action) => {
  switch (action.type) {

    case ADD_MANY_CUSTOMERS:
      return { ...state, customers: [...state.customers, ...action.payload] };

    case ADD_CUSTOMER:
      return { ...state, customers: [...state.customers, action.payload] };
    case REMOVE_CUSTOMER:
      return {
        ...state,
        customers: [
          ...state.customers.filter(
            (customer) => customer.id !== action.payload
          ),
        ],
      };
    default:
      return state;
  }
};

export const addCustomerAction = (payload) => ({
  type: ADD_CUSTOMER,
  payload: payload,
});
export const removeCustomerAction = (payload) => ({
  type: REMOVE_CUSTOMER,
  payload: payload,
});

export const addManyCustomersAction = (payload) => ({
  type: ADD_MANY_CUSTOMERS,
  payload,
});
```

Создадим папку для асинхронных действий

`asyncActions`

В ней: `customers.js`

```
import { addManyCustomersAction } from '../store/customersReducer';

export const fetchCustomers = () => {

  // функция dispatch как параметр потом сама автоматом передается
  // с помощью мидлвейр redux thunk

  return function (dispatch) {
    fetch('https://jsonplaceholder.typicode.com/users')
      .then((response) => response.json())
      .then((customers) => {
        dispatch(addManyCustomersAction(customers));
      });
  };
};
```

Теперь осталось в `App` добавить кнопку с функцией

```
import { fetchCustomers } from './asyncActions/customers';


 <button className='btn' onClick={() => dispatch(fetchCustomers())}>
   Database customers
 </button>
```

## Работа с асинхронными событиями в `Redux Saga`

В `Redux Saga` есть три основных понятия:

- `workers`

- `watchers`

- `effects`

Вся работа `Redux Saga` построена вокруг функций-генераторов

1. `worker` - функция-генератор, внутри которой выполняется какая-то

асинхронная логика(таймауты, асинхронные запросы...)

2. `watcher` - также функция-генератор, в которой с помощью специальных функций указывается

тип `action` и `worker`, срабатывающий на данный `action`

Еще раз: `watcher` - функция-наблюдатель, ожидяющая когда отработает соответствующий `action`.

И если к этому событию привязан какой-нибудь `worker`, то он его вызывает!

3. `effects` - набор встроенных в `Redux Saga` функций, которые помогают

делать запросы, делать `dispatch`, следить за воркерами ....
