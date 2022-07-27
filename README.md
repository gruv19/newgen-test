# Тестовое задание

## Исходный код для анализа
```javascript
class UserService {
    var username; 
    var password; 
  
    constructor(username, password) {
      this.username = username;
      this.password = password;
    }
  
    get username() { 
      return UserService.username; 
    }  
  
    get password() { 
      throw "You are not allowed to get password";
    }
  
    static authenticate_user() {
      let xhr = new XMLHttpRequest();
      xhr.open('GET', 'https://example.com/api?username=' + 
      UserService.username + '&password=' + UserService.password, true);
      xhr.responseType = 'json';
  
      const result = false;
      
      xhr.onload = function() {
        if (xhr.status !== '200') {
          result = xhr.response;
        } else {
          result = true;
        }
      };
  
      return result;
    }
   }
  
   $('form #login').click(function() {
    var username = $('#username'); 
    var password = $('#password');
  
    var res = UserService(username, password).authenticate_user();
  
    if (res == true) {
      document.location.href = '/home';
    } else {
      alert(res.error);
    }
   });
```

## Комментарии

```javascript
$('form #login').click(function() { ... })
```
> Выполняется обращение по селектору id, т.к. он должен быть уникальным на странице достаточно использовать $('#login').
> Если данные отправляются из формы, то корректнее использовать обработку события submit

```javascript
var username = $('#username'); 
var password = $('#password');
```
> var устаревшая конструкция, т.к. значения username, password не изменяются дальше в коде, лучше использовать const.
> Эти конструкции вернут объект jquery, необходимо использовать методы этого объекта, чтобы получить конкретное значение. Например, $('#username').val()

```javascript
var res = UserService(username, password).authenticate_user();
```
> var устаревшая конструкция, т.к. значения res не изменяются дальше в коде, лучше использовать const.
> Выполнение приведет к ошибке, т.к. конструктор класса, UserService не может быть вызван без ключевого слова new. Статический метод класса можно вызвать без создания экземпляра класса, например UserService.authenticate_user(), но, чтобы это сработало, необходимо менять реализацию метода authenticate_user.

```javascript
if (res == true)
```
> Необходимо использовать строгую проверку равенства "==="

```javascript
if {
  ...
} else { 
  alert(res.error); 
}
```
> Метод authenticate_user() может вернуть значение xhr.response, у которого нет свойства error.

```javascript
class UserService { 
  var username; 
  var password;
  ...
}
```
> Использование ключевого слова var перед свойством объекта выдаст ошибку

```javascript
get username() 
get password()
```
> Геттеры не должны иметь одинаковое имя со свойствами класса, в текущей реализации при обращении, например, objectUserService.password сработает не геттер, а обращение напрямую к свойству объекта.

```javascript
get username() { 
  return UserService.username; 
} 
```
> Даже при корректном именовании геттера вернет undefined

По логике именования класса UserService, предосатвляет некоторые функции для пользователей, но необходимость хранить данные пользователя (тем более пароль) в объектах этого класса отсутствует. 
Корректнее было бы исключить из данного класса все кроме статического метода authenticate_user(), т.е. привести класс к следующему виду:
```javascript
class UserService {
  static authenticate_user(username, password) { ... }
}
```
Рассмотрим текущую реализацию метода authenticate_user
```javascript
xhr.open('GET', 'https://example.com/api/user/authenticate?username=' 
  + UserService.username + '&password=' + UserService.password, true);
```
> Необходимо использовать метод POST для передачи пароля, т.к. при передачи данных через метод GET, они отобразятся в адресной строке браузера, что не безопасно.
> В методе отсутствует команда отправки запроса xhr.send(), т.е. события load, обрабочик которого описан дальше не будет инициировано.

```javascript
if (xhr.status !== '200')
```
> xhr.status имеет числовой тип

## Исправленный код
```javascript
class UserService {
  static authenticate_user(username, password) {
    const xhr = new XMLHttpRequest();
    const result = {};
    const body = new FormData();

    // в зависимости от реализации backend-а можно использовать другой способ формирования body, например JSON
    body.append('username', username);
    body.append('password', password);

    xhr.open('POST', 'https://example.com/api', true);
    xhr.send(body)

    xhr.onload = function (e) {
      if (xhr.readyState === 4) {
        (xhr.status === 200)
          ? result = { auth: true, message: xhr.statusText }
          : result = { auth: false, message: xhr.statusText };
      }
    };
      
    return result;
  }
}

// Считаю, что корректнее либо использовать jquery как в классе, так и в основном коде,
// либо не использовать нигде, чтоб было всё в едином стиле. 
document.querySelector('#login').addEventListener('submit', (e) => {
  const username = document.querySelector('#username').value;
  const password = document.querySelector('#password').value;

  const res = UserService.authenticate_user(username, password);
  if (res.auth) {
    document.location.href = '/home';
  } else {
    alert(`Произошла ошибка: ${res.message}`);
  }
});
```