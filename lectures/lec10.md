## Лекция 10: Redux

### Введение

Redux - это библиотека для управления состоянием в React-приложениях. Она помогает централизовать и упростить управление данными, особенно в крупных проектах с сложной логикой и множеством компонентов.

### Доработки фронтенда для авторизации

- **Добавление окон регистрации и авторизации**: Необходимо создать интерфейсы для регистрации и входа в систему.
- **Логика проверки авторизации**: После успешной авторизации меняется состояние приложения.
- **Разлогинивание**: Авторизованный пользователь может выйти из системы.
- **Доступ к функционалу**: Авторизованному пользователю доступен больший объем функционала в зависимости от его роли.

### React и MVC

React не следует традиционной модели MVC (Model-View-Controller), но имеет свои аналоги:

- **Model**: В React это состояние компонентов или глобальное состояние, управляемое Redux.
- **View**: Компоненты React, которые отображают данные.
- **Controller**: Логика обновления состояния и обработки событий.

### Виды Redux

- **Redux классовый**: Использование классовых компонентов для работы с Redux.
- **Redux функциональный через Hooks**: Использование функциональных компонентов и хуков для работы с Redux.
- **Redux Toolkit**: Упрощенный набор инструментов для работы с Redux, используемый в лабораторных работах.
- **Альтернативы**: MobX, стандартный useContext + useReducer.

### Использование useReducer и useContext

- **useReducer**: Хук для управления состоянием с помощью reducer-функций.
- **useContext**: Хук для доступа к контексту приложения.

### Создание Slice

Slice - это часть Redux-состояния и связанные с ней reducers.

```javascript
const dataSlice = createSlice({
    name: "data",
    initialState: {
        Data: []
    },
    reducers: {
        setData(state, {payload}) {
            state.Data = payload
        }
    }
})
```

- **initialState**: Начальное состояние хранилища.
- **reducers**: Функции, которые изменяют состояние.

### Создание Store

Store создается с помощью функции configureStore:

```javascript
export default configureStore({
    reducer: combineReducers({
        ourData: dataReducer
    })
})
```

- **combineReducers**: Объединяет несколько reducer-функций в одну.

### Подключение Store к Приложению

Store подключается к приложению с помощью компонента Provider:

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App'
import store from "./store";
import { Provider } from "react-redux";

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
    <React.StrictMode>
        <Provider store={store}>
            <App />
        </Provider>
    </React.StrictMode>
);
```

### Использование Хуков

- **useSelector**: Хук для выбора данных из store.
- **useDispatch**: Хук для отправки actions.

```javascript
export const useData = () => useSelector((state) => state.ourData.Data)
export const { setData: setDataAction } = dataSlice.actions
```

### Взаимодействие с API

- **Использование axios**: Библиотека для отправки HTTP-запросов.
- **useEffect**: Хук для выполнения кода при монтировании компонента.

```javascript
useEffect(() => {
    axios.get('https://api.example.com/data')
        .then(response => {
            dispatch(setDataAction(response.data))
        })
        .catch(error => {
            console.error('Error fetching data:', error)
        })
}, [])
```

### Заключение

Redux предоставляет мощный инструментарий для управления состоянием в React-приложениях. Использование Redux Toolkit упрощает работу с Redux, уменьшая количество шаблонного кода и предотвращая распространенные ошибки. Правильное использование Redux может значительно улучшить структуру и поддерживаемость крупных React-приложений[1].
