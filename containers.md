#Контейнер {#top}

Компоненты-контейнеры в общем случае соответствуют страницам/разделам проекта, определяют структуру 
проекта, логику связывания, обновления состояния, роутинг. По возможности, в контейнерах 
используются готовые компоненты. Вёрстка, оформление полностью выносятся в повторно используемые 
компоненты. 

Требования к контейнерам аналогичны требованиям к повторно используемым компонентам - именование, 
структура файлов, структура класса. 

По файловой структуре контейнеров необходимо находить компромисс между количеством файлов на одном 
уровне и глубиной вложенности папок. Строгого правила, что все контейнеры в containers на одном 
уровне или про древовидную вложенность в соответствии с роутингом - нету. Нужно, чтобы было удобно 
ориентироваться среди множества контейнеров. 

##Связывание с redux {#redux}

Связывание с redux функцией connect. 
Указываются только необходимые контейнеру store. 
Указание конкретных свойств конкретного store зависит от особенностей store и делается это, 
если store жирный.

```javascript
export default connect(state=>({
   account: state.account,
   login: state.login
}))(Login);
```

Через connect не прокидываются действия в `this.props`. В контейнере используется функция 
`this.props.disptach()` с вызовом нужного action. 

> Явное использование `dispatch()` даёт быстрое 
понимание, что вызывается action. Также не требуется прописывать заранее желаемых actions в connect 
и в `propTypes` и решать потенциальную проблему с конфликтом именования actions разных store.

Если action асинхронный (возвращает `Promise` или определен как `async`), то результат вызова 
action можно обработать через `.then()`, а не ожидая соответствующего изменения `props` в 
`componentWillReceiveProps()`

```javascript
this.props.dispatch(
    loginAction.login({login, password})
).then(result => {
   // например редирект а личный кабинет
});
```

##Роутинг {#routing}

Роутингом определяется, какой контейнер отображать при соответствующем адресе в браузере. 
Также роутингом определяется вложенность контейнеров, например в центральную область контейнера 
админки вставляются контейнеры подразделов админки. Тем самым контейнер админки определяет единый 
каркас для всех подразделов, базовый адрес и, например, логику с проверкой прав доступа. 

Роутинг реализуется библиотекой react-router. На текущий момент 4 версия. Описание роутинга 
выполняется в рендере контейнеров (JSX) соответствующими компонентами библиотеки роутинга. 
Желательно, чтобы в контейнере с роутингом был только роутинг для четкого разделения ответственности.

В отличие от предыдущих версий react-router, роутинг на всё приложение не описывается в одном месте 
(в одном файле/контейнере), а декомпозируется в соответствии с вложенностью контейнеров. 
Например, в корневом контейнере описан роутинг на главную, страницу входа, регистрации, 
личный кабинет. А роутинг на разделы личного кабинета уже определяется в контейнере личного кабинета.