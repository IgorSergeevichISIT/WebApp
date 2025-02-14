# Методические указания по GitHub Pages, PWA и адаптивности

## План работы
1. Развертывание приложения React в GitHub Pages
2. Progressive Web Application
3. Добавление адаптивности

## 1. Развертывание приложения React в GitHub Pages

С помощью `GitHub Pages` возможно развернуть статическое приложение, например наш React проект. Но развернуть наш бекенд здесь не получится.

[Пример развертывания React + Vite][vite-gh-pages]

### Использование библиотеки gh-pages
Для удобства используем библиотеку gh-pages:
```shell
npm install gh-pages
```

Добавим в `package.json` команду `"deploy": "gh-pages -d dist"`:
```json
{
  "name": "RepoName",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview",
    "deploy": "gh-pages -d dist"
  }
}
```

### Важные аспекты для успешного деплоя
- Убедитесь, что в проекте нет ошибок и предупреждений.
- Настройте роутинг корректно, предполагается использование react-router-dom. 

#### Пример правильной настройки роутинга:
```tsx
import "./App.css";
import { BrowserRouter, Route, Routes } from "react-router-dom";
import { BasketPage, HomePage, ProductPage, ProductsPage } from "./pages";

function App() {
  return (
    <BrowserRouter basename="/RepoName"> {/* RepoName - название вашего репозитория */}
      <Routes>
        <Route path="/" index element={<HomePage />} />
        <Route path="/basket" element={<BasketPage />} />
        <Route path="/products" element={<ProductsPage />} />
        <Route path="/products/:id" element={<ProductPage />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

#### В ```a href=""``` не учитывается basename, поэтому их нужно заменить на ```Link to=""```, который будет автоматически подставлять basename

### Настройка vite.config.ts
Укажите название вашего репозитория в vite.config.ts:
```ts
export default defineConfig({
  plugins: [react()],
  base: "/RepoName", // Замените RepoName на имя вашего репозитория
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:8080",
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, "/"),
      },
    },
  },
});
```

### Сборка и развертывание приложения
Используйте следующие команды для сборки и развертывания вашего приложения:
```shell
npm run build
npm run deploy
```

### Доступ к приложению
После выполнения этих шагов, ваше приложение будет доступно по адресу https://YourGitHubUsername.github.io/RepoName/, где YourGitHubUsername - ваше имя пользователя на GitHub, а RepoName - название вашего репозитория. Ссылку на приложение можно найти во вкладке "deployments" вашего репозитория.

### Обратите внимание

При развертывании приложения `React` через `GitHub Pages`, ваши `AJAX` запросы будут идти по `http`, в то время как приложение будет доступно по `https`. Работать это будет только при использовании адреса `localhost` в `AJAX` запросах.

## 2. Progressive Web Application

[Решения](https://github.com/iu5git/web/issues/14) для типовых проблем при выполнении задания по PWA

[PWA](https://ru.wikipedia.org/wiki/Прогрессивное_веб-приложение) (Progressive Web Application, Прогрессивное web-приложение) - это веб-приложение с характеристиками мобильного приложения. Приложения также запускается в браузере, но браузер пустой, без тулбаров и разных менюшек.

С помощью pwa можно:

- создать иконку приложения на рабочем столе устройства
- достучаться до аппаратуры
- отправлять push уведомления
- работать в оффлайн

Подробнее про pwa можно почитать [здесь](https://habr.com/ru/post/418923/)

Для создания `PWA` базово нужно 2 шага:

- иметь manifest.json
- и зарегистрированный service worker, который умеет кешировать запросы, то есть приложение может работать в оффлайн

### manifest.json

Начнем с manifest.json:

```json
{
  "name": "Tile Notes",
  "short_name": "Tile Notes",
  "start_url": "/RepoName/",
  "display": "standalone",
  "background_color": "#fdfdfd",
  "theme_color": "#db4938",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "/logo192.png",
      "type": "image/png", "sizes": "192x192"
    },
    {
      "src": "/logo512.png",
      "type": "image/png", "sizes": "512x512"
    }
  ]
}
```

Вот так он выглядит. Зачем он нужен? Он нужен, чтобы сказать браузеру, что наше приложение pwa, и задать некоторые настройки: название приложения - оно будет появляться на рабочем столе телефона, start_url - какую страницу браузеру запустить при старте, иконки, фоновый цвет и еще парочка опций.

Этот файлик должен быть доступен по пути /manifest.json относительно корня, то есть url примерно такой http://localhost:3000/manifest.json. Если мы используем react, то кладем данный файл в public директорию, которая находится в корне проекта. Туда же кладем иконки.

Чтобы его подключить к проекту, нужно его добавить в head в index.html:
```html
<link rel="manifest" href="manifest.json">
```
Проверить то, что файлик корректно подтянулся можно так:

![](assets/1.png)

В DevTools (инструменты разработчика, Ctrl+Shift+I, браузер Chrome). Заходим во вкладку Application, там должно быть что-то похожее как на скрине, без предупреждений и ошибок.

### Service Worker

Что это такое? Это скрипт, который выполняется в фоне, в отдельном потоке, то есть отдельно от страницы. Он умеет разные штуки, в частности, можно перехватывать все сетевые запросы и кешировать их.

Регистрируем service worker, делаем это в файле `index.js` после рендера корневого компонента:

```javascript
if ("serviceWorker" in navigator) {
  window.addEventListener("load", function() {
    navigator.serviceWorker
      .register("/serviceWorker.js")
      .then(res => console.log("service worker registered"))
      .catch(err => console.log("service worker not registered", err))
  })
}
```

Создаем файл serviceWorker.js и кладем его в директорию public:

```javascript
self.addEventListener('fetch',() => console.log("fetch"));
```

Здесь мы формально выполняем требования браузера: нужно повесить обработчик на событие fetch. Если хотите поиграться с разными стратегиями кеширования, то можно глянуть [тут](https://habr.com/ru/company/2gis/blog/345552/)

Если по пути, что-то пошло не так, то возможно нужно будет перезагрузить service worker:

![](assets/2.png)

Делается это нажатием кнопочки `Unregister` и перезагрузкой страницы.

Теперь можно проверить, что все работает:

![](assets/3.png)

Должна появиться иконочка `Install Apllication` справа в десктопном браузере. Нажимаем на нее, должно открыться отдельное окно с приложением.

Делаем то же самое на смартфоне. Для этого нужно поместить компьютер и телефон в одну сеть, взять адрес компьютера (на Linux/Mac можно сделать через `ip addr`) Должно быть что-то похожее на 192.168.199.97. Открыть приложения в мобильном браузере по ссылке http://192.168.199.97:3000. И установить приложение через меню:

![](assets/4.jpg)

Нажимаем `Добавить на главный экран`. Готово - иконка должна появиться на рабочем столе.

### Vite PWA

При работе с Vite так же существует более простой способ настройки PWA - библиотека vite-plugin-pwa.
Во-первых, необходимо установить библиотеку с помощью команды:

```shell
npm install -D vite-plugin-pwa
```
После этого необходимо настроить библиотеку в `vite.config.ts`
```ts
import { VitePWA } from 'vite-plugin-pwa'

export default defineConfig({
  plugins: [
    VitePWA({ registerType: 'autoUpdate' })
  ]
})
```
На данном этапе если вы развернете приложение на Github Pages или запустите его в режиме preview, то вы увидите работающий manifest.json, но его не будет при запуске в режиме dev.
Это происходит потому, что по умолчанию VitePWA не работает в режиме разработчика. Для того чтобы это исправить, можно добавить данное поле VitePWA в `vite.config.ts`
```ts
VitePWA({
  registerType: 'autoUpdate',
  devOptions: {
    enabled: true,
  },
})
```

Теперь, у нас есть и manifest, но при этом manifest, отображаемы на сервера — это созданный по умолчанию manifest, а не тот, который мы создали выше. Для указания собственного manifest его надо прописать как отдельное поле VitePWA в `vite.config.ts`
```ts
VitePWA({
  registerType: 'autoUpdate',
  devOptions: {
    enabled: true,
  },
  manifest:{
    name: "Tile Notes",
    short_name: "Tile Notes",
    start_url: "/",
    display: "standalone",
    background_color: "#fdfdfd",
    theme_color: "#db4938",
    orientation: "portrait-primary",
    icons: [
      {
      	"src": "/logo192.png",
      	"type": "image/png", "sizes": "192x192"
      },
      {
      	"src": "/logo512.png",
      	"type": "image/png", "sizes": "512x512"
      }
    ],
  }
})
```
Теперь перейдем к Service Worker. в vite-plugin-pwa уже есть работающий Service Worker который также производит кеширование, в отличие от прописанного нами. Данный Service Worker при запуске Github Pages вы можете увидеть, но он не будет active. Для того чтобы он заработал, нам нужно его зарегистрировать. 
Для этого перейдем в `main.tsx` и зарегистрируем Service Worker.

Подключим registerSW:
```ts
import {registerSW} from "virtual:pwa-register";
```
И после кода, где мы подключили наше приложение, пропишем:
```ts
if ("serviceWorker" in navigator) {
  registerSW()
}
```
На данном этапе настройка Service Worker закончена, и при запуске build на Github Pages можно будет увидеть и Mainfest.json и Service Worker. Убедительно проверьте, что в Manifest нет ошибок, которые могли бы повлиять на установку PWA. Эти ошибки отмечены во вкладке Manifest в отдельной группе. Если ошибок нет, то после скачивание PWA вы должны иметь возможность перейти в авиарежим и при этом приложение все еще будет работать, даже если его закрыть и открыть снова.

Важно! Если при импортировании registerSW появляется ошибка об отсутствии модуля, необходимо перейти в `tsconfig.app.json ` и в `CompilerOptions` добавить:
```json
{
  "compilerOptions": {
    "types": [
      "vite-plugin-pwa/info.d.ts",
      "vite-plugin-pwa/client.d.ts"
    ],
```

Однако, как вы могли заметить, при запуске где-либо кроме Github Pages, Service Worker либо вообще не запускается, либо выдает ошибку `The script has an unsupported MIME type ('text/html').`
В дополнение к этому, на вашем телефоне (если у вас Android) не будет возможности скачать PWA по ip адрессу. 

Обе эти ошибки связаны с тем, что для работы PWA сайт должен работать по протоколу https, а не http. Как настроить сайт для работы с https рассмотрим ниже.
## 3. Настройка https на React

Для того, чтобы сайт работал по протоколу https он должен иметь сертификат. Данные сертификаты подтверждаются несколько одобренных компаний за небольшую сумму, но для нас будет достаточно автоматически сгенерированного локально подтвержденного сертификата.

Для работы с сертификатами, во-первых, установим модуль mkcert
```shell
npm install -g mkcert
```
После этого нам нужно создать Authority которая будет подтверждать наш сертификат. Сертификаты, подтвержденные данным образом, реально работают только при работе на localhost. 

После создания Authority необходимо создать сам сертификат. Для этого использует команды:
```shell
mkcert create-ca
mkcert create-cert
```

После выполнения данных команд у вас появится 4 новых файла: ca.crt, ca.key, cert.crt, cert.key.


`ca.crt` и `cert.crt` - это публичные ключи Authority и сертификата соответственно, а ca.key и cert.key - приватные ключи. 

##### Никогда не выкладывайте частные ключи в общий доступ, даже на GitHub!

После создания данных ключей остался один шаг - настроить `vite.config.ts`, перед началом установив `vite-plugin-mkcert` и `@types/node`:
```ts
import mkcert from 'vite-plugin-mkcert'
import fs from 'fs';
import path from 'path';

plugins: [react(), mkcert(), VitePWA({})]

server:{
  https:{
    key: fs.readFileSync(path.resolve(__dirname, 'cert.key')),
    cert: fs.readFileSync(path.resolve(__dirname, 'cert.crt')),
  },
}
```
После добавления данного кода, вы можете открыть ваш сервер по вашему основному ip(не localhost и не vpn) и увидеть, что Service Worker подключён.


## 4. Добавление адаптивности
Зачем это нужно? Адаптивность помогает вашему веб-приложению нормально выглядить на устройствах с разными размерами экрана, а также влияет на продвижение сайта в поисковых системах. Сделать это можно c помощью использования относительно новых моделей макета (flexbox, grid), а так же через media queries в css. [Здесь](https://ru.hexlet.io/courses/css-adaptive/lessons/media-queries/theory_unit) можно почитать про последние.

### Практические примеры:

1. Список товаров

Предположим, что у нас есть список товаров (в данном случае - книги), и мы должны сверстать такое расположение карточек, чтобы при уменьшении размеров экрана товары съезжали вниз и полностью покрывали новое пространство. В этом нам поможет flexbox и немного медиа запросы.

Структура карточки, обертки, в которой находятся все товары, и их css-свойства:
```html
<div class="cards__wrapper">
    <div class="card__item">
        <div class="card__img">
            <img src="./images/book.jpg" alt="book">
        </div>
        <div class="card__info">
            <div class="card__text">
                <div class="card__title">Название книги</div>
                <div class="card__description">Описание книги</div>
            </div>
            <button class="card__btn">Приобрести</button>
        </div>
    </div>
</div>
```

```css
.cards__wrapper {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 20px;
}

.card__item {
    flex: 1 1 300px;
    padding: 15px;
    background-color: bisque;
    border-radius: 10px;
    display: flex;
    justify-content: space-between;
}

.card__img {
    width: 40%;
}

.card__img img {
    max-width: 100%;
    height: auto;
}

.card__info {
    width: 55%;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
}

.card__title {
    font-size: 24px;
}

.card__description {
    font-size: 18px;
}

.card__btn {
    padding: 10px 15px;
    border-radius: 10px;
    border: none
}
```

На что здесь надо обратить внимание? 
1. на свойства обертки, которые позволяют переносить элементы на новую строчку, если предыдущая заполнилась (`flex-wrap`), а так же задают отступы между соседними элементами сверху и снизу (`gap`)
2. на свойства самой карточки: в данном случае нас интересует первое свойство `flex`, а точнее последнее значение в нем. Это значение определяет, в какой момент элементы переносятся на новую строчку, а именно если размер элемента становится равным `300px`

Подробнее про flexbox можете прочитать [здесь](https://medium.com/@stasonmars/%D0%B2%D0%B5%CC%88%D1%80%D1%81%D1%82%D0%BA%D0%B0-%D0%BD%D0%B0-flexbox-%D0%B2-css-%D0%BF%D0%BE%D0%BB%D0%BD%D1%8B%D0%B8%CC%86-%D1%81%D0%BF%D1%80%D0%B0%D0%B2%D0%BE%D1%87%D0%BD%D0%B8%D0%BA-e26662cf87e0)

Получается такая картина:
- 1200px

![](./assets/6.png)
- 768px

![](./assets/7.png)
- 545px

![](./assets/8.png)

При этом на мобильных устройствах уже не рекомендуется ставить элементы "в строчку", так как при длинных текстах (например, большое описание книги) наша верстка будет съезжать, поэтому следует разместить картинку сверху.

Для этого нам понадобятся медиа запросы. Пишем ключение слово `@media`, и после него следует указать условие, по которому будет выполняться запрос. Например:

```css
@media (max-width: 460px) {
    .card__item {
        flex-direction: column;
        align-items: center;
        gap: 20px 0
    }

    .card__img, .card__info {
        width: 100%;
    }

    .card__btn {
        margin-top: 20px;
    }
}
```
В данном случае, если ширина viewport будет меньше либо равно `460px`, то применяются все стили к элементам, которые расположены внутри блока `@media (max-width: 460px) {}`.

Итого, при просмотре на ширине экрана `320px` наш список товаров принимает такой вид.

![](./assets/9.png)

2. Создание адаптивной панели навигации

Панель навигации является важнейшим элементом в веб-приложении, ведь пользователь на протяжении всего серфинга по сайту его видит, поэтому следует уделить особое внимание адаптивности данного компонента.

Пример макета компонента `Navbar`:
```jsx
import React from 'react'
import {NavLink} from "react-router-dom";
import './Navbar.css'

export const Navbar = () => {
  return (
      <nav className='nav'>
        <div className='nav__wrapper'>
          <div className='nav__links'>
            <NavLink to='/' className='nav__link'>Главная</NavLink>
            <NavLink to='/items' className='nav__link'>Товары</NavLink>
            <NavLink to='/orders' className='nav__link'>Заказы</NavLink>
            <NavLink to='/about' className='nav__link'>О магазине</NavLink>
          </div>
          <div className='nav__cart'>
            <NavLink to='/cart' className='nav__link nav__link--cart'>Корзина</NavLink>
          </div>
        </div>
      </nav>
  )
}
```

`Navbar.css`
```css
.nav {
    width: calc(100% - 70px);
    max-width: 1200px;
    height: 40px;
    background-color: #387ef6;
    box-shadow: 0 0 15px rgb(0 0 0 / 10%);
    padding: 10px 20px;
    border-radius: 15px;
    position: fixed;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    align-items: center;
}

.nav__wrapper {
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.nav__links {
    width: 90%;
    display: flex;
    align-items: center;
    gap: 0 35px
}

.nav__link {
    text-decoration: none;
    color: #fff;
    position: relative;
}

.nav__link--cart {
    color: #387ef6;
    padding: 5px 10px;
    background-color: #fff;
    border-radius: 5px;
    font-weight: 700;
}

.nav__link.active::after {
    content: '';
    position: absolute;
    width: 100%;
    height: 3px;
    left: 0;
    bottom: -7px;
    background-color: #fff;
}
```

В итоге мы получаем такую навигацию:

![](./assets/10.png)

Можно заметить, что при ширине экрана меньше `545px` наш компонент выглядит очень не презентабильно:

![](./assets/11.png)

Что же нам делать в такой ситуации? На помощь приходит компонент, в народе называемый бургер меню (если присмотреться, то действительно похоже на бургер - 2 булочки и котлета посередине).

![](./assets/14.png)

Встроим его в наш компонент `Navbar.jsx`
```jsx
import React from 'react'
import {NavLink} from "react-router-dom"
import './Navbar.css'

export const Navbar = () => {
  return (
      <nav className='nav'>
        <div className='nav__wrapper'>
          <div className='nav__links'>
            <NavLink to='/' className='nav__link'>Главная</NavLink>
            <NavLink to='/items' className='nav__link'>Товары</NavLink>
            <NavLink to='/orders' className='nav__link'>Заказы</NavLink>
            <NavLink to='/about' className='nav__link'>О магазине</NavLink>
          </div>
          <div className='nav__cart'>
            <NavLink to='/cart' className='nav__link nav__link--card'>Корзина</NavLink>
          </div>
          <div className='nav__mobile-wrapper'>
            <div className='nav__mobile-target' />
            <div className='nav__mobile-menu'>
              <NavLink to='/' className='nav__link'>Главная</NavLink>
              <NavLink to='/items' className='nav__link'>Товары</NavLink>
              <NavLink to='/orders' className='nav__link'>Заказы</NavLink>
              <NavLink to='/about' className='nav__link'>О магазине</NavLink>
            </div>
          </div>
        </div>
      </nav>
  )
}
```

Бургер меню имеет всего два состояния:
1. активное - показывается меню со ссылками на другие страницы, сам бургер превращается в крестик
2. не активное - меню c навигационными ссылками скрыта, бургер принимает знакомое всем форму

Чтобы отслеживать состояние бургер меню, сделаем простой обработчик события `onClick` при клике на элемент с классом `nav__mobile-wrapper`:
```jsx
 <div className='nav__mobile-wrapper'
      onClick={(event) => event.currentTarget.classList.toggle('active')}
  >
    <div className='nav__mobile-target' />
    <div className='nav__mobile-menu'>
      <NavLink to='/' className='nav__link'>Главная</NavLink>
      <NavLink to='/items' className='nav__link'>Товары</NavLink>
      <NavLink to='/orders' className='nav__link'>Заказы</NavLink>
      <NavLink to='/about' className='nav__link'>О магазине</NavLink>
    </div>
</div>
```

Что он делает? Всего лишь "переключает" класс `active` у элемента `nav__mobile-wrapper`, то есть добавляет, если `active` отсутствует, и наоборот.

Осталось лишь добавить стили для нашего бургер меню и обработать ситуацию, когда `nav__mobile-wrapper` имеет класс `acitve`.
```css
.nav__mobile-wrapper {
    display: none;
    height: 30px;
    width: 30px;
    transition: all .4s linear;
    cursor: pointer;
    position: relative;
}

.nav__mobile-target {
    height: 2px;
    width: 100%;
    background-color: black;
    position: relative;
    transition: all .2s linear;
}

.nav__mobile-wrapper.active .nav__mobile-target {
    transform: rotate(45deg)
}

.nav__mobile-target::after, .nav__mobile-target::before {
    content: '';
    position: absolute;
    left: 0;
    height: 2px;
    width: 100%;
    background-color: black;
}

.nav__mobile-target::after {
    top: -7px;
}

.nav__mobile-target::before {
    bottom: -7px;
}

.nav__mobile-wrapper.active .nav__mobile-target::after {
    display: none;
}

.nav__mobile-wrapper.active .nav__mobile-target::before {
    top: 0;
    transform: rotate(90deg);
}

.nav__mobile-menu {
    position: absolute;
    top: 30px;
    left: 0;
    flex-direction: column;
    gap: 15px 0;
    background-color: #387ef6;
    padding: 20px;
    min-width: 150px;
    border-radius: 10px;
    display: none;
}

.nav__mobile-wrapper.active .nav__mobile-menu {
    display: flex;
}

@media (max-width: 545px) {
    .nav__wrapper {
        flex-direction: row-reverse;
    }

    .nav__links {
        display: none;
    }

    .nav__mobile-wrapper {
        display: flex;
        align-items: center;

    }
}
```

Как можно заметить, на брейкпоинте `545px` мы скрываем ссылки, которые расположены в строчку, и отображаем меню. Следует обратить внимание на псевдоэлементы `::after, ::before`, которые позволяют стилизовать элемент, не добавляя дополнительные теги в DOM-дерево. Подробнее про псевдоэлемты и их практическое использование можно почитать [здесь](https://doka.guide/css/pseudoelements/).

В конце можно столкнуться с такой `проблемой`, при нажатии на ссылку в нашем бургер меню, элемент сразу же закрывается. Это связано со `всплытием события`. Если кратко, то при нажатии на дочерний элемент, событие `click` передастся всем родительским, в том числе и `nav__mobile-wrapper`, что приведет к исключению класса `active` из списка классов этого элемента. Подробнее про всплытие можно почитать [здесь](https://learn.javascript.ru/bubbling-and-capturing)

Чтобы этого избежать, надо перехватить событие, например в элементе `nav__mobile-menu`, и вызвать у `event` метод, останавливающий всплытие - `stopPropagation()`

```jsx
<div className='nav__mobile-wrapper'
     onClick={(event) => event.currentTarget.classList.toggle('active')}
>
    <div className='nav__mobile-target' />
    <div className='nav__mobile-menu' onClick={(event) => event.stopPropagation()}>
        <NavLink to='/' className='nav__link'>Главная</NavLink>
        <NavLink to='/items' className='nav__link'>Товары</NavLink>
        <NavLink to='/orders' className='nav__link'>Заказы</NavLink>
        <NavLink to='/about' className='nav__link'>О магазине</NavLink>
    </div>
</div>
```

Итого, наше меню имеет следующий вид:
- в неактивном состоянии

![](./assets/12.png)

- в активном состоянии

![](./assets/13.png)

[vite-gh-pages]: https://rashidshamloo.hashnode.dev/deploying-vite-react-app-to-github-pages
