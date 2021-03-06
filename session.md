# Управление сессией (авторизацией)

Авторизованные запросы выполняются с передачей токена (идентификатора сессии). 
В зависимости от АПИ, токен может передаваться в http заголовке, url параметре, куке. 

Фронт получает токен при успешной авторизации. Иногда сразу после успешной регистрации 
(после регистрации можно послать запрос авторизации, если пароль задавался пользователем). 

Если сервер присылает токен в куке, а принимает в заголовке, то достаточно настроить библиотеку 
[axios], указав название куки и заголовка (для кроссдоменных запросов не забыть опцию `withCredentials:true`) 

```javascript
const Http = axios.create({
  baseURL: CONFIG.baseUrl,

  // `xsrfCookieName` is the name of the cookie to use as a value for xsrf token
  xsrfCookieName: 'XSRF-TOKEN', // default

  // `xsrfHeaderName` is the name of the http header that carries the xsrf token value
  xsrfHeaderName: 'X-XSRF-TOKEN', // default
});
```
 
В ином случаи реализуется хранение токена самостоятельно в `localStorage`. За сроком действия токена 
и другие параметры безопасности (например, сверка IP) ответственен сервер. Поэтому сохраненный токен 
используется пока позволяет сервер.

Пользователь считается авторизованным, если имеется токен и профиль аккаунта (минимально 
необходимые данные). В последующем сервер может сообщить о недействительности токена, тогда из 
состояния фронта удаляется и токен и профиль. 

Начальное состояние `store.account`
```javascript
{
	token: undefined, // Ещё неизвестно есть токен или его точно нет
  user: {} // не null, чтобы не проверять на null при обращении к свойствам
}
```    
        
- При undefined токена показывать спинер загрузки приложения в корневом контейнере, не выводя 
вложенные контейнеры (роутеры). Им бессмысленно работать, так как неизвестно состояние аккаунта 
(авторизован или нет). Чтобы не делали лишних редиректов на форму входа из-за временного отсутствия 
признаков авторизованности.
- В корневом контейнере при его монтировании делается запрос "вспомнить" или, другими словами: 
"получить профиль по токену". Сервер должен вернуть профиль аккаунта и, при необходимости, новый 
токен. Запрос делается с текущим токеном. В типовой логике необходимости проверять наличие токена 
до запроса нет. 
- Профиль и токен сохраняются в store.account. Если токен в куке, то в store его значение можно не 
сохранять, а присвоить true (токен есть)
-Если профиль не получен, то авторизоваться по текуну не удалось, и в store.account его нужно 
установить в null. Считается значение токена определенно, но оно null (допустимо false).
- При значении токена !== undefined уже выводятся вложенные контейнеры. Если они должны работать 
только для авторизованного пользователя или с проверкой роли, то именно в них эта проверка 
выполняется в `componentWillMount()` и в `componentWillReceiveProps()`. Если доступ запрещается, 
то делается редирект на страницу авторизации.
- При успешной авторизации по паролю (из формы входа) делать редирект на необходимую страницу. 
Возможно, предыдущую, главную или конкретную, например личный кабинет. Зависит от ТЗ. 
Если форма входа в модалке, то скорее всего, просто закрывать её. Суть в том, что редирект 
осуществляется непосредственно после входа через форму, а не из-за изменения store.account 
- При выходе делается запрос на сервер, чтобы тот удалил у себя токен. Если токен приходит по куке, 
то сервер должен ответить с удалением куки. Если управление токеном на фронте своё, то после 
успешного запроса нужно установить токен в null в store.account. И очистить `localStorage`.
