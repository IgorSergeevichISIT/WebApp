# Лекция 10: Redux - управление состоянием в React приложениях (расширенная версия)

Добро пожаловать на лекцию по Redux — одной из самых важных библиотек для управления состоянием в современных React приложениях.

## Доработки фронтенда и необходимость Redux

При разработке сложных фронтенд приложений мы сталкиваемся с необходимостью управления глобальным состоянием:

**Авторизация:**
- Добавить окна регистрации и аутентификации
- Добавить логику проверки авторизации пользователя — после успешной авторизации меняется **состояние приложения**
- Авторизованный пользователь может разлогиниться
- Авторизованному пользователю доступен больший объем функционала в зависимости от его роли
- Поля поиска, сложные формы с дополнительными полями и т.д.

Когда состояние становится сложным и нужно передавать данные между множеством компонентов, стандартные подходы React становятся неэффективными. Именно здесь приходит на помощь Redux.

## MVC vs Flux

Для понимания Redux важно сначала разобрать эволюцию архитектурных паттернов:

### MVC (Model-View-Controller)

**Архитектура MVC:**
- **Model** — управляет данными и бизнес-логикой
- **View** — отображает данные пользователю
- **Controller** — обрабатывает пользовательский ввод и координирует Model и View

```javascript
// Пример проблемы MVC - двунаправленный поток данных
class UserModel {
  constructor() {
    this.users = [];
    this.observers = [];
  }

  addUser(user) {
    this.users.push(user);
    this.notifyObservers();
  }

  notifyObservers() {
    this.observers.forEach(observer => observer.update());
  }
}

class UserView {
  constructor(model) {
    this.model = model;
    this.model.observers.push(this);
  }

  update() {
    // View может изменить Model, что создает сложные зависимости
    this.render();
  }

  handleUserClick(userId) {
    // View напрямую изменяет Model
    this.model.updateUser(userId);
  }
}
```

**Проблемы MVC:**
- Двунаправленный поток данных
- Сложность отладки
- Непредсказуемые изменения состояния

### Flux

Facebook разработал Flux как альтернативу MVC с однонаправленным потоком данных:

**Компоненты Flux:**
- **Action** — объект, описывающий что произошло
- **Dispatcher** — центральный hub для всех actions
- **Store** — содержит состояние и логику
- **View** — React компоненты

```javascript
// Пример Flux архитектуры
const Dispatcher = require('flux').Dispatcher;
const EventEmitter = require('events').EventEmitter;

// Action Creator
const UserActions = {
  createUser(userData) {
    AppDispatcher.dispatch({
      type: 'CREATE_USER',
      userData: userData
    });
  }
};

// Store
const UserStore = Object.assign({}, EventEmitter.prototype, {
  users: [],

  getUsers() {
    return this.users;
  },

  emitChange() {
    this.emit('change');
  },

  addChangeListener(callback) {
    this.on('change', callback);
  }
});

// Dispatcher handler
AppDispatcher.register((action) => {
  switch(action.type) {
    case 'CREATE_USER':
      UserStore.users.push(action.userData);
      UserStore.emitChange();
      break;
  }
});

// React Component
class UserList extends React.Component {
  constructor() {
    super();
    this.state = { users: UserStore.getUsers() };
  }

  componentDidMount() {
    UserStore.addChangeListener(this._onChange);
  }

  _onChange = () => {
    this.setState({ users: UserStore.getUsers() });
  }

  handleCreateUser = (userData) => {
    UserActions.createUser(userData);
  }

  render() {
    return (
      
        {this.state.users.map(user => 
          {user.name}
        )}
      
    );
  }
}
```

## React + Redux

Redux упрощает концепции Flux, используя три принципа:

1. **Единственный источник истины** — всё состояние приложения хранится в одном store
2. **Состояние только для чтения** — изменять состояние можно только через действия (actions)
3. **Изменения описываются чистыми функциями** — reducers указывают, как изменяется состояние

### Поток данных в React + Redux

```
View → Action → Reducer → Store → View
```

```javascript
// Основные концепции Redux

// 1. Action - описывает что произошло
const addUser = (user) => ({
  type: 'ADD_USER',
  payload: user
});

// 2. Reducer - как изменяется состояние
const usersReducer = (state = { users: [] }, action) => {
  switch (action.type) {
    case 'ADD_USER':
      return {
        ...state,
        users: [...state.users, action.payload]
      };
    default:
      return state;
  }
};

// 3. Store - хранилище состояния
import { createStore } from 'redux';
const store = createStore(usersReducer);

// 4. Подключение к React
import { Provider, useSelector, useDispatch } from 'react-redux';

function App() {
  return (
    
      
    
  );
}

function UserList() {
  const users = useSelector(state => state.users);
  const dispatch = useDispatch();

  const handleAddUser = (user) => {
    dispatch(addUser(user));
  };

  return (
    
      {users.map(user => {user.name})}
       handleAddUser({ id: Date.now(), name: 'New User' })}>
        Add User
      
    
  );
}
```

## Виды Redux

### Redux классовый (устаревший подход)

```javascript
import { connect } from 'react-redux';

class UserListClass extends React.Component {
  handleAddUser = () => {
    this.props.addUser({ id: Date.now(), name: 'New User' });
  }

  render() {
    return (
      
        {this.props.users.map(user => 
          {user.name}
        )}
        Add User
      
    );
  }
}

const mapStateToProps = (state) => ({
  users: state.users
});

const mapDispatchToProps = (dispatch) => ({
  addUser: (user) => dispatch(addUser(user))
});

export default connect(mapStateToProps, mapDispatchToProps)(UserListClass);
```

### Redux функциональный через Hooks

```javascript
import { useSelector, useDispatch } from 'react-redux';

function UserListHooks() {
  const users = useSelector(state => state.users);
  const dispatch = useDispatch();

  const handleAddUser = () => {
    dispatch(addUser({ id: Date.now(), name: 'New User' }));
  };

  return (
    
      {users.map(user => {user.name})}
      Add User
    
  );
}
```

### Redux Toolkit (рекомендуемый подход)

Redux Toolkit упрощает работу с Redux, решая основные проблемы:
- Сложная конфигурация store
- Много дополнительных пакетов
- Слишком много шаблонного кода

```javascript
import { configureStore, createSlice } from '@reduxjs/toolkit';

// Создание slice с Redux Toolkit
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    users: [],
    loading: false,
    error: null
  },
  reducers: {
    addUser: (state, action) => {
      // Redux Toolkit использует Immer, поэтому можно "мутировать" состояние
      state.users.push(action.payload);
    },
    removeUser: (state, action) => {
      state.users = state.users.filter(user => user.id !== action.payload);
    },
    setLoading: (state, action) => {
      state.loading = action.payload;
    },
    setError: (state, action) => {
      state.error = action.payload;
      state.loading = false;
    }
  }
});

export const { addUser, removeUser, setLoading, setError } = usersSlice.actions;

// Создание store
const store = configureStore({
  reducer: {
    users: usersSlice.reducer
  }
});

// Использование в компоненте
function UserList() {
  const { users, loading, error } = useSelector(state => state.users);
  const dispatch = useDispatch();

  const handleAddUser = (userData) => {
    dispatch(addUser(userData));
  };

  const handleRemoveUser = (userId) => {
    dispatch(removeUser(userId));
  };

  if (loading) return Загрузка...;
  if (error) return Ошибка: {error};

  return (
    
      {users.map(user => (
        
          {user.name}
           handleRemoveUser(user.id)}>Удалить
        
      ))}
       handleAddUser({ id: Date.now(), name: 'New User' })}>
        Добавить пользователя
      
    
  );
}
```

## useReducer + useContext как альтернатива

Для простых приложений можно использовать встроенные React хуки вместо Redux:

```javascript
import React, { createContext, useContext, useReducer } from 'react';

// Reducer
const usersReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_USER':
      return {
        ...state,
        users: [...state.users, action.payload]
      };
    case 'REMOVE_USER':
      return {
        ...state,
        users: state.users.filter(user => user.id !== action.payload)
      };
    case 'SET_LOADING':
      return {
        ...state,
        loading: action.payload
      };
    default:
      return state;
  }
};

// Context
const UsersContext = createContext();

// Provider
export function UsersProvider({ children }) {
  const [state, dispatch] = useReducer(usersReducer, {
    users: [],
    loading: false,
    error: null
  });

  const actions = {
    addUser: (user) => dispatch({ type: 'ADD_USER', payload: user }),
    removeUser: (id) => dispatch({ type: 'REMOVE_USER', payload: id }),
    setLoading: (loading) => dispatch({ type: 'SET_LOADING', payload: loading })
  };

  return (
    
      {children}
    
  );
}

// Hook для использования
export function useUsers() {
  const context = useContext(UsersContext);
  if (!context) {
    throw new Error('useUsers must be used within UsersProvider');
  }
  return context;
}

// Использование
function UserList() {
  const { state, actions } = useUsers();

  return (
    
      {state.users.map(user => (
        
          {user.name}
           actions.removeUser(user.id)}>Удалить
        
      ))}
       actions.addUser({ id: Date.now(), name: 'New User' })}>
        Добавить
      
    
  );
}

function App() {
  return (
    
      
    
  );
}
```

## Практический пример: корзина без API

Создадим полноценную корзину с использованием Redux Toolkit:

```javascript
// store/cartSlice.js
import { createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    totalPrice: 0,
    totalItems: 0
  },
  reducers: {
    addToCart: (state, action) => {
      const { product, quantity = 1 } = action.payload;
      const existingItem = state.items.find(item => item.id === product.id);

      if (existingItem) {
        existingItem.quantity += quantity;
      } else {
        state.items.push({
          ...product,
          quantity
        });
      }

      // Пересчитываем общую стоимость
      state.totalPrice = state.items.reduce(
        (total, item) => total + (item.price * item.quantity), 0
      );
      state.totalItems = state.items.reduce(
        (total, item) => total + item.quantity, 0
      );
    },

    removeFromCart: (state, action) => {
      const productId = action.payload;
      state.items = state.items.filter(item => item.id !== productId);
      
      state.totalPrice = state.items.reduce(
        (total, item) => total + (item.price * item.quantity), 0
      );
      state.totalItems = state.items.reduce(
        (total, item) => total + item.quantity, 0
      );
    },

    updateQuantity: (state, action) => {
      const { productId, quantity } = action.payload;
      const item = state.items.find(item => item.id === productId);

      if (item) {
        if (quantity  item.id !== productId);
        } else {
          item.quantity = quantity;
        }

        state.totalPrice = state.items.reduce(
          (total, item) => total + (item.price * item.quantity), 0
        );
        state.totalItems = state.items.reduce(
          (total, item) => total + item.quantity, 0
        );
      }
    },

    clearCart: (state) => {
      state.items = [];
      state.totalPrice = 0;
      state.totalItems = 0;
    }
  }
});

export const { addToCart, removeFromCart, updateQuantity, clearCart } = cartSlice.actions;
export default cartSlice.reducer;

// Selectors
export const selectCartItems = (state) => state.cart.items;
export const selectCartTotal = (state) => state.cart.totalPrice;
export const selectCartItemsCount = (state) => state.cart.totalItems;
export const selectCartItemById = (productId) => (state) => 
  state.cart.items.find(item => item.id === productId);
```

```javascript
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import cartReducer from './cartSlice';

export const store = configureStore({
  reducer: {
    cart: cartReducer
  },
  devTools: process.env.NODE_ENV !== 'production'
});

export type RootState = ReturnType;
export type AppDispatch = typeof store.dispatch;
```

```javascript
// components/ProductCard.jsx
import { useDispatch, useSelector } from 'react-redux';
import { addToCart, selectCartItemById } from '../store/cartSlice';

function ProductCard({ product }) {
  const dispatch = useDispatch();
  const cartItem = useSelector(selectCartItemById(product.id));

  const handleAddToCart = () => {
    dispatch(addToCart({ product, quantity: 1 }));
  };

  return (
    
      
      {product.name}
      {product.price}₽
      {product.description}
      
      
        
          Добавить в корзину
        
        {cartItem && (
          
            В корзине: {cartItem.quantity}
          
        )}
      
    
  );
}

// components/Cart.jsx
import { useSelector, useDispatch } from 'react-redux';
import { 
  selectCartItems, 
  selectCartTotal, 
  selectCartItemsCount,
  removeFromCart, 
  updateQuantity, 
  clearCart 
} from '../store/cartSlice';

function Cart() {
  const cartItems = useSelector(selectCartItems);
  const totalPrice = useSelector(selectCartTotal);
  const totalItems = useSelector(selectCartItemsCount);
  const dispatch = useDispatch();

  const handleQuantityChange = (productId, newQuantity) => {
    dispatch(updateQuantity({ productId, quantity: parseInt(newQuantity) }));
  };

  const handleRemoveItem = (productId) => {
    dispatch(removeFromCart(productId));
  };

  const handleClearCart = () => {
    dispatch(clearCart());
  };

  if (cartItems.length === 0) {
    return (
      
        Корзина пуста
        Добавьте товары для оформления заказа
      
    );
  }

  return (
    
      
        Корзина ({totalItems} товаров)
        
          Очистить корзину
        
      

      
        {cartItems.map(item => (
          
            
            
              {item.name}
              {item.price}₽
            
            
            
              Количество:
               handleQuantityChange(item.id, e.target.value)}
              />
            

            
              {(item.price * item.quantity).toFixed(2)}₽
            

             handleRemoveItem(item.id)}
              className="remove-item"
            >
              Удалить
            
          
        ))}
      

      
        
          Итого: {totalPrice.toFixed(2)}₽
        
        
          Перейти к оформлению
        
      
    
  );
}
```

## Добавление взаимодействия с API

Для работы с асинхронными операциями в Redux Toolkit используется `createAsyncThunk`:

```javascript
// store/productsSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

// Асинхронный thunk для загрузки продуктов
export const fetchProducts = createAsyncThunk(
  'products/fetchProducts',
  async (_, { rejectWithValue }) => {
    try {
      const response = await axios.get('/api/products');
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response.data.message || 'Ошибка загрузки продуктов');
    }
  }
);

// Асинхронный thunk для создания продукта
export const createProduct = createAsyncThunk(
  'products/createProduct',
  async (productData, { rejectWithValue }) => {
    try {
      const response = await axios.post('/api/products', productData);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response.data.message || 'Ошибка создания продукта');
    }
  }
);

// Асинхронный thunk для обновления продукта
export const updateProduct = createAsyncThunk(
  'products/updateProduct',
  async ({ id, data }, { rejectWithValue }) => {
    try {
      const response = await axios.put(`/api/products/${id}`, data);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response.data.message || 'Ошибка обновления продукта');
    }
  }
);

const productsSlice = createSlice({
  name: 'products',
  initialState: {
    items: [],
    loading: false,
    error: null,
    selectedProduct: null
  },
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
    setSelectedProduct: (state, action) => {
      state.selectedProduct = action.payload;
    }
  },
  extraReducers: (builder) => {
    builder
      // Загрузка продуктов
      .addCase(fetchProducts.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })
      
      // Создание продукта
      .addCase(createProduct.pending, (state) => {
        state.loading = true;
      })
      .addCase(createProduct.fulfilled, (state, action) => {
        state.loading = false;
        state.items.push(action.payload);
      })
      .addCase(createProduct.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })
      
      // Обновление продукта
      .addCase(updateProduct.fulfilled, (state, action) => {
        const index = state.items.findIndex(item => item.id === action.payload.id);
        if (index !== -1) {
          state.items[index] = action.payload;
        }
      });
  }
});

export const { clearError, setSelectedProduct } = productsSlice.actions;
export default productsSlice.reducer;

// Selectors
export const selectProducts = (state) => state.products.items;
export const selectProductsLoading = (state) => state.products.loading;
export const selectProductsError = (state) => state.products.error;
export const selectSelectedProduct = (state) => state.products.selectedProduct;
```

```javascript
// components/ProductList.jsx
import { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { 
  fetchProducts, 
  selectProducts, 
  selectProductsLoading, 
  selectProductsError,
  clearError 
} from '../store/productsSlice';
import ProductCard from './ProductCard';

function ProductList() {
  const products = useSelector(selectProducts);
  const loading = useSelector(selectProductsLoading);
  const error = useSelector(selectProductsError);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchProducts());
  }, [dispatch]);

  useEffect(() => {
    if (error) {
      const timer = setTimeout(() => {
        dispatch(clearError());
      }, 5000);
      return () => clearTimeout(timer);
    }
  }, [error, dispatch]);

  if (loading) {
    return (
      
        
        Загрузка продуктов...
      
    );
  }

  if (error) {
    return (
      
        Ошибка загрузки
        {error}
         dispatch(fetchProducts())}>
          Попробовать еще раз
        
      
    );
  }

  return (
    
      Каталог товаров
      
        {products.map(product => (
          
        ))}
      
    
  );
}

// components/ProductForm.jsx
import { useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { createProduct, updateProduct, selectProductsLoading } from '../store/productsSlice';

function ProductForm({ product, onClose }) {
  const [formData, setFormData] = useState({
    name: product?.name || '',
    price: product?.price || '',
    description: product?.description || '',
    image: product?.image || ''
  });

  const loading = useSelector(selectProductsLoading);
  const dispatch = useDispatch();

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      if (product) {
        await dispatch(updateProduct({ id: product.id, data: formData })).unwrap();
      } else {
        await dispatch(createProduct(formData)).unwrap();
      }
      onClose();
    } catch (error) {
      console.error('Ошибка сохранения:', error);
    }
  };

  const handleChange = (e) => {
    setFormData({
      ...formData,
      [e.target.name]: e.target.value
    });
  };

  return (
    
      {product ? 'Редактировать' : 'Добавить'} товар
      
      
        Название:
        
      

      
        Цена:
        
      

      
        Описание:
        
      

      
        Изображение (URL):
        
      

      
        
          {loading ? 'Сохранение...' : 'Сохранить'}
        
        
          Отмена
        
      
    
  );
}
```

## RTK Query для продвинутой работы с API

RTK Query предоставляет мощные возможности для кеширования и синхронизации данных:

```javascript
// api/productsApi.js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const productsApi = createApi({
  reducerPath: 'productsApi',
  baseQuery: fetchBaseQuery({
    baseUrl: '/api/',
    prepareHeaders: (headers, { getState }) => {
      const token = getState().auth.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    },
  }),
  tagTypes: ['Product'],
  endpoints: (builder) => ({
    getProducts: builder.query({
      query: (params = {}) => ({
        url: 'products',
        params
      }),
      providesTags: ['Product']
    }),

    getProduct: builder.query({
      query: (id) => `products/${id}`,
      providesTags: (result, error, id) => [{ type: 'Product', id }]
    }),

    createProduct: builder.mutation({
      query: (newProduct) => ({
        url: 'products',
        method: 'POST',
        body: newProduct
      }),
      invalidatesTags: ['Product']
    }),

    updateProduct: builder.mutation({
      query: ({ id, ...patch }) => ({
        url: `products/${id}`,
        method: 'PUT',
        body: patch
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Product', id }]
    }),

    deleteProduct: builder.mutation({
      query: (id) => ({
        url: `products/${id}`,
        method: 'DELETE'
      }),
      invalidatesTags: ['Product']
    })
  })
});

export const {
  useGetProductsQuery,
  useGetProductQuery,
  useCreateProductMutation,
  useUpdateProductMutation,
  useDeleteProductMutation
} = productsApi;
```

```javascript
// components/ProductListRTK.jsx
import { useGetProductsQuery, useDeleteProductMutation } from '../api/productsApi';

function ProductListRTK() {
  const {
    data: products = [],
    error,
    isLoading,
    refetch
  } = useGetProductsQuery();

  const [deleteProduct] = useDeleteProductMutation();

  const handleDelete = async (id) => {
    if (window.confirm('Удалить товар?')) {
      try {
        await deleteProduct(id).unwrap();
      } catch (error) {
        console.error('Ошибка удаления:', error);
      }
    }
  };

  if (isLoading) return Загрузка...;
  if (error) return Ошибка: {error.message};

  return (
    
      Обновить
      {products.map(product => (
        
          {product.name}
          {product.price}₽
           handleDelete(product.id)}>
            Удалить
          
        
      ))}
    
  );
}
```

## Организация store в больших приложениях

```javascript
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query';

// Reducers
import authReducer from './slices/authSlice';
import cartReducer from './slices/cartSlice';
import uiReducer from './slices/uiSlice';

// APIs
import { productsApi } from './apis/productsApi';
import { usersApi } from './apis/usersApi';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    cart: cartReducer,
    ui: uiReducer,
    [productsApi.reducerPath]: productsApi.reducer,
    [usersApi.reducerPath]: usersApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    })
    .concat(productsApi.middleware)
    .concat(usersApi.middleware),
  devTools: process.env.NODE_ENV !== 'production',
});

setupListeners(store.dispatch);

export type RootState = ReturnType;
export type AppDispatch = typeof store.dispatch;
```

## Паттерны и лучшие практики

### Структура папок

```
src/
├── store/
│   ├── slices/
│   │   ├── authSlice.js
│   │   ├── cartSlice.js
│   │   └── uiSlice.js
│   ├── apis/
│   │   ├── productsApi.js
│   │   └── usersApi.js
│   └── index.js
├── hooks/
│   ├── useAppDispatch.ts
│   └── useAppSelector.ts
└── components/
```

### Типизированные хуки для TypeScript

```typescript
// hooks/redux.ts
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from '../store';

export const useAppDispatch = () => useDispatch();
export const useAppSelector: TypedUseSelectorHook = useSelector;
```

### Нормализация данных

```javascript
import { createEntityAdapter, createSlice } from '@reduxjs/toolkit';

const usersAdapter = createEntityAdapter({
  selectId: (user) => user.id,
  sortComparer: (a, b) => a.name.localeCompare(b.name)
});

const usersSlice = createSlice({
  name: 'users',
  initialState: usersAdapter.getInitialState({
    loading: false,
    error: null
  }),
  reducers: {
    userAdded: usersAdapter.addOne,
    usersLoaded: usersAdapter.setAll,
    userUpdated: usersAdapter.updateOne,
    userRemoved: usersAdapter.removeOne
  }
});

export const {
  selectAll: selectAllUsers,
  selectById: selectUserById,
  selectIds: selectUserIds
} = usersAdapter.getSelectors((state) => state.users);
```

Redux представляет собой мощный инструмент для управления состоянием в сложных React приложениях. Правильное использование Redux Toolkit и понимание принципов архитектуры позволяет создавать масштабируемые и поддерживаемые приложения.

