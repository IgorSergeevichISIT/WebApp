# Лекция 9: Архитектура фронтенда (расширенная версия)

Добро пожаловать на лекцию по разработке интернет-приложений. Сегодня мы детально изучим архитектуру фронтенда — одну из ключевых тем современной веб-разработки.

## Классы фронтенда

В современной веб-разработке мы четко разделяем **бэкенд** и **фронтенд**[1]. Это фундаментальное разделение позволяет:

- Разрабатывать клиентскую и серверную части независимо
- Использовать разные технологические стеки
- Масштабировать команды разработки
- Обеспечить лучшую производительность и пользовательский опыт

**Пример архитектуры:**
```
Клиент (React/Vue/Angular) ↔ API ↔ Сервер (Node.js/Python/Java)
```

Фронтенд отвечает за:
- Пользовательский интерфейс
- Взаимодействие с пользователем
- Представление данных
- Валидацию форм на клиенте
- Роутинг

Бэкенд обрабатывает:
- Бизнес-логику
- Работу с базами данных
- Аутентификацию и авторизацию
- API endpoints

## Поток данных и сообщений

В архитектуре фронтенда критически важно правильно организовать поток данных[3]. Современные фреймворки используют несколько подходов:

**Однонаправленный поток данных** — данные передаются сверху вниз по иерархии компонентов[3]. Самый известный пример — это Flux и Redux:

```javascript
// Пример однонаправленного потока в Redux
const initialState = { count: 0 };

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// Компонент получает данные через props
function Counter({ count, onIncrement, onDecrement }) {
  return (
    
      {count}
      +
      -
    
  );
}
```

Во Flux приложение состоит из трех главных компонентов[3]:
- **Store** — хранилище данных
- **Dispatcher** — диспетчер
- **View** — представление

## Методы жизненного цикла компонента

В React компоненты проходят через определенные фазы жизненного цикла[4]:

**Этап монтирования:**
- `constructor()` — инициализация компонента
- `componentDidMount()` — компонент добавлен в DOM

**Этап обновления:**
- `componentDidUpdate()` — компонент обновился
- `shouldComponentUpdate()` — решение о необходимости обновления

**Этап размонтирования:**
- `componentWillUnmount()` — компонент удаляется из DOM

```javascript
class LifecycleExample extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    console.log('1. Constructor вызван');
  }

  componentDidMount() {
    console.log('3. ComponentDidMount вызван');
    // Здесь можно делать API запросы
  }

  componentDidUpdate(prevProps, prevState) {
    console.log('ComponentDidUpdate вызван');
    if (prevState.count !== this.state.count) {
      console.log('Count изменился:', this.state.count);
    }
  }

  componentWillUnmount() {
    console.log('ComponentWillUnmount вызван');
    // Очистка подписок, таймеров
  }

  render() {
    console.log('2. Render вызван');
    return (
      
        Count: {this.state.count}
         this.setState({ count: this.state.count + 1 })}>
          Increment
        
      
    );
  }
}
```

## Функциональные компоненты

Современный React ориентируется на функциональные компоненты[1][5], которые имеют ряд преимуществ:

**Преимущества функциональных компонентов:**
- Описание компонентов с помощью чистых функций создает меньше кода, а значит его легче поддерживать[1]
- Чистые функции намного проще тестировать — вы просто передаете props на вход и ожидаете какую-то разметку[1]
- В будущем чистые функции будут выигрывать по скорости работы в сравнении с классами из-за отсутствия методов жизненного цикла[1]

```javascript
// Функциональный компонент
const Button = (props) => (
  
    {props.children}
  
);

// Эквивалентный классовый компонент
class Button extends React.Component {
  render() {
    return (
      
        {this.props.children}
      
    );
  }
}
```

Все это стало возможным благодаря внедрению хуков в React[1][6].

## Хуки

Хуки — это функции, которые позволяют использовать состояние и другие возможности React в функциональных компонентах[6]. Основные хуки включают:

**useState** — для управления локальным состоянием:
```javascript
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    
      Вы кликнули {count} раз
       setCount(count + 1)}>
        Кликни меня
      
    
  );
}
```

**useEffect** — для выполнения побочных эффектов:
```javascript
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Этот эффект выполняется после каждого рендера
    async function fetchUser() {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (error) {
        console.error('Ошибка загрузки пользователя:', error);
      } finally {
        setLoading(false);
      }
    }

    fetchUser();
  }, [userId]); // Эффект выполнится заново при изменении userId

  // Очистка при размонтировании
  useEffect(() => {
    return () => {
      console.log('Компонент размонтируется');
    };
  }, []);

  if (loading) return Загрузка...;
  if (!user) return Пользователь не найден;

  return (
    
      {user.name}
      {user.email}
    
  );
}
```

**useCallback** — для мемоизации функций:
```javascript
import React, { useState, useCallback } from 'react';

function TodoList() {
  const [todos, setTodos] = useState([]);
  const [text, setText] = useState('');

  // Мемоизируем функцию, чтобы избежать лишних перерендеров
  const addTodo = useCallback(() => {
    if (text.trim()) {
      setTodos(prev => [...prev, { id: Date.now(), text, completed: false }]);
      setText('');
    }
  }, [text]);

  const toggleTodo = useCallback((id) => {
    setTodos(prev => prev.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  }, []);

  return (
    
       setText(e.target.value)}
        placeholder="Добавить задачу"
      />
      Добавить
      
        {todos.map(todo => (
          
             toggleTodo(todo.id)}
            >
              {todo.text}
            
          
        ))}
      
    
  );
}
```

**useRef** — для прямого доступа к DOM элементам:
```javascript
import React, { useRef, useEffect } from 'react';

function FocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    // Автоматически фокусируемся на input при монтировании
    inputRef.current.focus();
  }, []);

  const handleButtonClick = () => {
    inputRef.current.focus();
  };

  return (
    
      
      Фокус на input
    
  );
}
```

## Другие важные хуки

**useContext** позволяет работать с контекстом — с механизмом для организации совместного доступа к данным без передачи свойств[1]:

```javascript
import React, { createContext, useContext, useState } from 'react';

// Создаем контекст для темы
const ThemeContext = createContext();

// Провайдер темы
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    
      {children}
    
  );
}

// Компонент, использующий контекст
function ThemedButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    
      Текущая тема: {theme}
    
  );
}

// Использование
function App() {
  return (
    
      
    
  );
}
```

**useReducer** усложняет useState добавляя разделение логики в зависимости от action. Вместе с useContext дают аналог Redux[1]:

```javascript
import React, { useReducer, useContext, createContext } from 'react';

// Reducer для управления состоянием корзины
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      const existingItem = state.items.find(item => item.id === action.payload.id);
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          )
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }]
      };
    
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };
    
    case 'CLEAR_CART':
      return { ...state, items: [] };
    
    default:
      return state;
  }
}

const CartContext = createContext();

function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });

  return (
    
      {children}
    
  );
}

function ProductCard({ product }) {
  const { dispatch } = useContext(CartContext);

  const addToCart = () => {
    dispatch({ type: 'ADD_ITEM', payload: product });
  };

  return (
    
      {product.name}
      Цена: {product.price}₽
      Добавить в корзину
    
  );
}
```

**useMemo** используется для возврата мемоизированного значения. Может применяться, чтобы функция возвратила кешированное значение. Можно сохранить результаты вычислений между вызовами render[1]:

```javascript
import React, { useState, useMemo } from 'react';

function ExpensiveCalculationComponent({ items, multiplier }) {
  const [filter, setFilter] = useState('');

  // Дорогое вычисление, которое выполняется только при изменении items или multiplier
  const expensiveValue = useMemo(() => {
    console.log('Выполняется дорогое вычисление...');
    return items.reduce((sum, item) => sum + item.value * multiplier, 0);
  }, [items, multiplier]);

  // Фильтрация элементов
  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);

  return (
    
       setFilter(e.target.value)}
        placeholder="Фильтр по имени"
      />
      Общая стоимость: {expensiveValue}
      
        {filteredItems.map(item => (
          {item.name}: {item.value}
        ))}
      
    
  );
}
```

## React Router Hooks

Для навигации в React приложениях используются специальные хуки[1]:

- **useLocation** — все данные о текущем пути url[1]
- **useNavigate** — объект истории браузера[1]
- **useParams** — параметры из url[1]

```javascript
import React from 'react';
import { useLocation, useNavigate, useParams } from 'react-router-dom';

function UserProfile() {
  const { userId } = useParams(); // Получаем параметры из URL
  const location = useLocation(); // Информация о текущем местоположении
  const navigate = useNavigate(); // Функция для навигации

  const handleGoBack = () => {
    navigate(-1); // Назад в истории
  };

  const handleGoToSettings = () => {
    navigate(`/users/${userId}/settings`);
  };

  return (
    
      Профиль пользователя {userId}
      Текущий путь: {location.pathname}
      Query параметры: {location.search}
      
      Назад
      Настройки
    
  );
}

// Пример использования в маршрутах
function App() {
  return (
    
      } />
      } />
    
  );
}
```

## Breadcrumbs и навигация

Для удобной навигации пользователя по страницам приложения используются[1]:

- **Самописные хлебные крошки** — показывают путь пользователя в иерархии страниц[1]
- **Навигационная панель (меню)** — часто из библиотек вроде react-bootstrap[1]

```javascript
import React from 'react';
import { useLocation, Link } from 'react-router-dom';

function Breadcrumbs() {
  const location = useLocation();
  
  // Разбиваем путь на сегменты
  const pathSegments = location.pathname.split('/').filter(segment => segment);
  
  // Создаем breadcrumb элементы
  const breadcrumbs = pathSegments.map((segment, index) => {
    const path = '/' + pathSegments.slice(0, index + 1).join('/');
    const isLast = index === pathSegments.length - 1;
    
    return {
      label: segment.charAt(0).toUpperCase() + segment.slice(1),
      path: path,
      isLast: isLast
    };
  });

  return (
    
      
        
          Главная
          {breadcrumbs.length > 0 &&  / }
        
        {breadcrumbs.map((breadcrumb, index) => (
          
            {breadcrumb.isLast ? (
              {breadcrumb.label}
            ) : (
              <>
                {breadcrumb.label}
                 / 
              
            )}
          
        ))}
      
    
  );
}

// Компонент поиска
function SearchComponent() {
  const [searchValue, setSearchValue] = useState('');
  const navigate = useNavigate();

  const handleSubmit = (e) => {
    e.preventDefault();
    if (searchValue.trim()) {
      navigate(`/search?q=${encodeURIComponent(searchValue)}`);
    }
  };

  return (
    
       setSearchValue(e.target.value)}
        placeholder="Поиск..."
      />
      Найти
    
  );
}
```

## Mock и тестирование

В реальной разработке часто возникает проблема связывания фронтенда и бэкенда[1]:

- Что-то не готово в backend[1]
- Проблемы с развертыванием[1]
- CORS проблемы[1]

Поэтому бэкенд проверяем через Swagger/Postman, а для фронтенда готовим Mock — имитацию API для независимой разработки[1].

```javascript
// mock/api.js - Mock данные для разработки
const MOCK_USERS = [
  { id: 1, name: 'Иван Петров', email: 'ivan@example.com' },
  { id: 2, name: 'Мария Сидорова', email: 'maria@example.com' },
  { id: 3, name: 'Алексей Иванов', email: 'alexey@example.com' }
];

const MOCK_POSTS = [
  { id: 1, title: 'Первый пост', content: 'Содержимое первого поста', authorId: 1 },
  { id: 2, title: 'Второй пост', content: 'Содержимое второго поста', authorId: 2 }
];

// Имитация задержки сети
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

// Mock API функции
export const mockApi = {
  async getUsers() {
    await delay(500); // Имитируем задержку
    return Promise.resolve(MOCK_USERS);
  },

  async getUserById(id) {
    await delay(300);
    const user = MOCK_USERS.find(u => u.id === parseInt(id));
    if (user) {
      return Promise.resolve(user);
    }
    return Promise.reject(new Error('Пользователь не найден'));
  },

  async getPosts() {
    await delay(700);
    return Promise.resolve(MOCK_POSTS);
  },

  async createUser(userData) {
    await delay(800);
    const newUser = {
      id: MOCK_USERS.length + 1,
      ...userData
    };
    MOCK_USERS.push(newUser);
    return Promise.resolve(newUser);
  }
};

// Использование в компоненте
function UsersList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function loadUsers() {
      try {
        setLoading(true);
        const usersData = await mockApi.getUsers();
        setUsers(usersData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    loadUsers();
  }, []);

  if (loading) return Загрузка пользователей...;
  if (error) return Ошибка: {error};

  return (
    
      {users.map(user => (
        
          {user.name} - {user.email}
        
      ))}
    
  );
}
```

## Обратный прокси-сервер для CORS

Одно из решений CORS проблем — не отправляем запросы напрямую в веб-сервис, а проксируем через наш сервер фронтенда. Это решение похоже на production при проксировании через Nginx[1].

```javascript
// vite.config.js - конфигурация прокси для Vite
export default {
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
}

// webpack.config.js - конфигурация прокси для Webpack
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        pathRewrite: { '^/api': '' },
        changeOrigin: true
      }
    }
  }
}

// Использование в коде
async function fetchUsers() {
  // Запрос идет на /api/users, но проксируется на http://localhost:3001/users
  const response = await fetch('/api/users');
  return response.json();
}
```

## BFF и GraphQL

**Backend for Frontend (BFF)** — это шлюз к нашим API, адаптированный под каждого из потребителей: веб-приложение, мобильное, десктоп[1].

**GraphQL** — язык запросов и сервер с открытым исходным кодом, который позволяет клиентам запрашивать только нужные данные[1].

```javascript
// Пример GraphQL запроса
const GET_USER_WITH_POSTS = gql`
  query GetUserWithPosts($userId: ID!) {
    user(id: $userId) {
      id
      name
      email
      posts {
        id
        title
        content
        createdAt
      }
    }
  }
`;

// Использование с Apollo Client
function UserProfile({ userId }) {
  const { loading, error, data } = useQuery(GET_USER_WITH_POSTS, {
    variables: { userId }
  });

  if (loading) return Загрузка...;
  if (error) return Ошибка: {error.message};

  const { user } = data;

  return (
    
      {user.name}
      {user.email}
      Посты пользователя:
      {user.posts.map(post => (
        
          {post.title}
          {post.content}
          {new Date(post.createdAt).toLocaleDateString()}
        
      ))}
    
  );
}

// BFF пример - разные API для разных клиентов
// Mobile BFF - минимальные данные
const MOBILE_USER_QUERY = gql`
  query GetMobileUser($userId: ID!) {
    user(id: $userId) {
      id
      name
      avatar
    }
  }
`;

// Web BFF - полные данные
const WEB_USER_QUERY = gql`
  query GetWebUser($userId: ID!) {
    user(id: $userId) {
      id
      name
      email
      avatar
      bio
      createdAt
      stats {
        postsCount
        followersCount
        followingCount
      }
    }
  }
`;
```

## Микрофронтенды

Микрофронтенды позволяют использовать в одном интерфейсе несколько фронтенд фреймворков[1]:

- Разные команды фронтенд разработчиков могут работать независимо[1]
- Разные API могут быть интегрированы[1]
- Один из способов уйти от зависимости от одной технологии[1]
- Позволяет постепенно внедрять новые технологии[1]

```javascript
// Пример конфигурации Module Federation (Webpack 5)
// webpack.config.js для основного приложения
const ModuleFederationPlugin = require('@module-federation/webpack');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        userMicroapp: 'userMicroapp@http://localhost:3001/remoteEntry.js',
        productMicroapp: 'productMicroapp@http://localhost:3002/remoteEntry.js',
      },
    }),
  ],
};

// Загрузка микрофронтендов
const UserMicroapp = React.lazy(() => import('userMicroapp/UserApp'));
const ProductMicroapp = React.lazy(() => import('productMicroapp/ProductApp'));

function App() {
  return (
    
      
        Пользователи
        Товары
      
      
      Загрузка микроприложения...}>
        
          } />
          } />
        
      
    
  );
}
```

## Next.js и серверный рендеринг

SPA — сильный инструмент, но как индексировать страницы в поисковиках? На SPA странице изначально нет никаких данных[1].

Для решения этой проблемы используем серверные компоненты Next.js[1]:

- Хуков нет — данные передаем через props[1]
- **SSG (Static Site Generation)** — HTML генерируется при сборке приложения[1]
- **SSR (Server-Side Rendering)** — HTML генерируется на сервере при каждом запросе[1]

```javascript
// pages/users/[id].js - SSR пример
export async function getServerSideProps(context) {
  const { id } = context.params;
  
  // Данные загружаются на сервере при каждом запросе
  const res = await fetch(`https://api.example.com/users/${id}`);
  const user = await res.json();

  return {
    props: {
      user,
    },
  };
}

function UserPage({ user }) {
  return (
    
      {user.name}
      {user.email}
      Дата регистрации: {user.createdAt}
    
  );
}

export default UserPage;

// pages/blog/[slug].js - SSG пример
export async function getStaticProps({ params }) {
  // Данные загружаются во время сборки
  const res = await fetch(`https://api.example.com/posts/${params.slug}`);
  const post = await res.json();

  return {
    props: {
      post,
    },
    // Регенерация страницы каждые 60 секунд
    revalidate: 60,
  };
}

export async function getStaticPaths() {
  // Определяем какие страницы нужно предварительно генерировать
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  const paths = posts.map((post) => ({
    params: { slug: post.slug },
  }));

  return {
    paths,
    fallback: 'blocking', // Генерировать новые страницы по требованию
  };
}

function BlogPost({ post }) {
  return (
    
      {post.title}
      
    
  );
}

export default BlogPost;

// App Router (Next.js 13+) - серверные компоненты
// app/users/[id]/page.js
async function UserPage({ params }) {
  // Это серверный компонент - выполняется на сервере
  const res = await fetch(`https://api.example.com/users/${params.id}`, {
    // Кеширование запроса
    next: { revalidate: 3600 }
  });
  const user = await res.json();

  return (
    
      {user.name}
      
    
  );
}

// Клиентский компонент для интерактивности
'use client';
function UserPosts({ userId }) {
  const [posts, setPosts] = useState([]);
  
  useEffect(() => {
    fetch(`/api/users/${userId}/posts`)
      .then(res => res.json())
      .then(setPosts);
  }, [userId]);

  return (
    
      {posts.map(post => (
        
      ))}
    
  );
}
```

## Feature-Sliced Design (FSD)

FSD — это архитектурная методология для организации кода фронтенд приложений[8][14]. Рассмотрим на примере приложения социальной сети[1].

**Структура проекта FSD:**
- **app/** содержит настройку роутера, глобальные хранилища и стили[1]
- **pages/** содержит компоненты роутов на каждую страницу в приложении, преимущественно композирующие, по возможности, без собственной логики[1]
- **widgets/** содержит "собранную" карточку поста, с содержимым и интерактивными кнопками, в которые вшиты запросы к бэкенду[1]
- **features/** содержит всю интерактивность карточки (например, кнопку лайка) и логику обработки этой интерактивности[1]
- **entities/** содержит скелет карточки со слотами под интерактивные элементы[1]

```
src/
├── app/                     # Слой приложения
│   ├── providers/          # Провайдеры (Redux, Router, Theme)
│   ├── styles/             # Глобальные стили
│   └── index.tsx           # Точка входа
│
├── pages/                   # Слой страниц
│   ├── home/               # Главная страница
│   │   ├── ui/
│   │   └── index.ts
│   ├── profile/            # Страница профиля
│   └── settings/           # Страница настроек
│
├── widgets/                 # Слой виджетов
│   ├── header/             # Шапка сайта
│   ├── post-card/          # Карточка поста
│   └── user-menu/          # Меню пользователя
│
├── features/                # Слой фич
│   ├── auth/               # Аутентификация
│   │   ├── ui/
│   │   ├── model/
│   │   ├── api/
│   │   └── index.ts
│   ├── like-post/          # Лайк поста
│   ├── add-comment/        # Добавление комментария
│   └── search/             # Поиск
│
├── entities/                # Слой сущностей
│   ├── user/               # Сущность пользователя
│   │   ├── ui/
│   │   ├── model/
│   │   ├── api/
│   │   └── index.ts
│   ├── post/               # Сущность поста
│   └── comment/            # Сущность комментария
│
└── shared/                  # Общий слой
    ├── ui/                 # UI компоненты
    │   ├── button/
    │   ├── input/
    │   └── modal/
    ├── lib/                # Утилиты
    ├── api/                # API клиент
    └── config/             # Конфигурация
```

## Архитектура FSD: слои и принципы

**Слои стандартизированы** во всех проектах и расположены вертикально. Модули на одном слое могут взаимодействовать лишь с модулями, находящимися на слоях строго ниже[1][8].

**Семь слоев FSD (снизу вверх):**

1. **shared** — переиспользуемый код, не имеющий отношения к специфике приложения/бизнеса (например, UIKit, libs, API)[1][8]
2. **entities (сущности)** — бизнес-сущности (например, User, Product, Order)[1][8]
3. **features (фичи)** — взаимодействия с пользователем, действия, которые несут бизнес-ценность для пользователя (например, SendComment, AddToCart, UsersSearch)[1][8]
4. **widgets (виджеты)** — композиционный слой для соединения сущностей и фич в самостоятельные блоки (например, IssuesList, UserProfile)[1][8]
5. **pages (страницы)** — композиционный слой для сборки полноценных страниц из сущностей, фич и виджетов[1][8]
6. **processes (процессы, устаревший слой)** — сложные сценарии, покрывающие несколько страниц (например, авторизация)[1][8]
7. **app** — настройки, стили и провайдеры для всего приложения[1][8]

**Практический пример структуры:**

```javascript
// shared/ui/button/Button.tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'small' | 'medium' | 'large';
  children: React.ReactNode;
  onClick?: () => void;
}

export function Button({ variant = 'primary', size = 'medium', children, onClick }: ButtonProps) {
  return (
    
      {children}
    
  );
}

// entities/user/model/types.ts
export interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  createdAt: string;
}

// entities/user/api/userApi.ts
export const userApi = {
  async getById(id: string): Promise {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  },

  async update(id: string, data: Partial): Promise {
    const response = await fetch(`/api/users/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }
};

// entities/user/ui/UserCard.tsx
import { Button } from 'shared/ui/button';
import { User } from '../model/types';

interface UserCardProps {
  user: User;
  actions?: React.ReactNode;
}

export function UserCard({ user, actions }: UserCardProps) {
  return (
    
      
      {user.name}
      {user.email}
      {actions && {actions}}
    
  );
}

// features/follow-user/ui/FollowButton.tsx
import { Button } from 'shared/ui/button';
import { useFollowUser } from '../model/useFollowUser';

interface FollowButtonProps {
  userId: string;
  isFollowing: boolean;
}

export function FollowButton({ userId, isFollowing }: FollowButtonProps) {
  const { follow, unfollow, isLoading } = useFollowUser();

  const handleClick = () => {
    if (isFollowing) {
      unfollow(userId);
    } else {
      follow(userId);
    }
  };

  return (
    
      {isFollowing ? 'Отписаться' : 'Подписаться'}
    
  );
}

// widgets/user-profile/ui/UserProfile.tsx
import { UserCard } from 'entities/user';
import { FollowButton } from 'features/follow-user';
import { EditProfileButton } from 'features/edit-profile';

interface UserProfileProps {
  user: User;
  isOwnProfile: boolean;
  isFollowing: boolean;
}

export function UserProfile({ user, isOwnProfile, isFollowing }: UserProfileProps) {
  return (
    
        ) : (
          
        )
      }
    />
  );
}

// pages/profile/ui/ProfilePage.tsx
import { UserProfile } from 'widgets/user-profile';
import { UserPosts } from 'widgets/user-posts';
import { useParams } from 'react-router-dom';
import { useUser } from 'entities/user';

export function ProfilePage() {
  const { userId } = useParams();
  const { user, isLoading } = useUser(userId);
  const currentUserId = useCurrentUserId();

  if (isLoading) return Загрузка...;

  return (
    
      
      
    
  );
}
```

**Слайсы** разделяют код по предметной области, группируя логически связанные модули[1][9]. **Сегменты** — это маленькие модули для разделения кода внутри слайса по техническому назначению: ui, model (store, actions), api и lib (utils/hooks)[1][9].

## Преимущества FSD

**Стандартизация архитектуры:**
Код распределяется согласно области влияния (слой), предметной области (слайс) и техническому назначению (сегмент). Благодаря этому архитектура стандартизируется и становится более простой для ознакомления[1][10].

**Контролируемое переиспользование логики:**
Каждый компонент архитектуры имеет свое назначение и предсказуемый список зависимостей. Благодаря этому сохраняется баланс между соблюдением принципа DRY и возможностью адаптировать модуль под разные цели[1][10].

**Устойчивость к изменениям и рефакторингу:**
Один модуль не может использовать другой модуль, расположенный на том же слое или на слоях выше. Благодаря этому приложение можно изолированно модифицировать под новые требования без непредвиденных последствий[1][10].

**Ориентированность на потребности бизнеса и пользователей:**
Разбиение приложения по бизнес-доменам помогает глубже понимать, структурировать и находить фичи проекта[1][10].

## Практический пример FSD

Для изучения FSD архитектуры доступен простой пример, рекомендуемый для начинающих. В нем каждый слой состоит из слайсов, например, Header[1][11].

```javascript
// Пример полной фичи "Добавление комментария"
// features/add-comment/model/types.ts
export interface Comment {
  id: string;
  text: string;
  authorId: string;
  postId: string;
  createdAt: string;
}

export interface AddCommentData {
  text: string;
  postId: string;
}

// features/add-comment/api/commentApi.ts
import { Comment, AddCommentData } from '../model/types';

export const commentApi = {
  async add(data: AddCommentData): Promise {
    const response = await fetch('/api/comments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }
};

// features/add-comment/model/useAddComment.ts
import { useState } from 'react';
import { commentApi } from '../api/commentApi';
import { AddCommentData } from './types';

export function useAddComment() {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  const addComment = async (data: AddCommentData) => {
    try {
      setIsLoading(true);
      setError(null);
      const comment = await commentApi.add(data);
      return comment;
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Ошибка добавления комментария');
      throw err;
    } finally {
      setIsLoading(false);
    }
  };

  return { addComment, isLoading, error };
}

// features/add-comment/ui/AddCommentForm.tsx
import { useState } from 'react';
import { Button } from 'shared/ui/button';
import { useAddComment } from '../model/useAddComment';

interface AddCommentFormProps {
  postId: string;
  onSuccess?: () => void;
}

export function AddCommentForm({ postId, onSuccess }: AddCommentFormProps) {
  const [text, setText] = useState('');
  const { addComment, isLoading, error } = useAddComment();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!text.trim()) return;

    try {
      await addComment({ text: text.trim(), postId });
      setText('');
      onSuccess?.();
    } catch {
      // Ошибка уже обработана в хуке
    }
  };

  return (
    
       setText(e.target.value)}
        placeholder="Напишите комментарий..."
        rows={3}
        disabled={isLoading}
      />
      {error && {error}}
      
        {isLoading ? 'Отправка...' : 'Отправить'}
      
    
  );
}

// features/add-comment/index.ts - Public API
export { AddCommentForm } from './ui/AddCommentForm';
export { useAddComment } from './model/useAddComment';
export type { Comment, AddCommentData } from './model/types';
```

FSD представляет собой современный подход к организации архитектуры фронтенд приложений, который помогает создавать масштабируемые и поддерживаемые проекты[7][8][10].

Это завершает нашу расширенную лекцию по архитектуре фронтенда. На следующих занятиях мы более детально изучим практическое применение этих концепций в реальных проектах и разберем дополнительные паттерны проектирования.

