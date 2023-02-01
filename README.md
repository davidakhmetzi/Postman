Данный репозиторий показывает мои практические навыки работы в <b>Postman</b>. Вы можете импортировать коллекцию и переменное окружение в Postman и проверить меня.
Для примера использован публичный API для создания временных аккаунтов почты (10 Minute Mail API https://docs.mail.gw). Логика сервиса заключается в том, что пользователю выдается аккаунт почты, который блокируется через 10 минут (удобно для прогонов кейсов с авторизацией, чтобы не использовать свой аккаунт).
Ниже представлены запросы с небольшим описанием логики и пояснения к частичной  автоматизации процессов.
***
+ <b>GET Получить список доменов, доступных для регистрации аккаунта почты</b>
<br>В теле ответа будет видны доступные домены для регистрации. Это самый первый запрос в цепочке запросов, поэтому для него прописан скрипт проверки соответствия статус кода (а также для всех последующих GET запросов). Скрипт сравнивает фактический и ожидаемый статус код. Принимается, что ожидается 200 статус код.
<br><i>pm.test("Status code is 200", function () {
<br>    pm.response.to.have.status(200);
<br>});</i>

+ <b>POST Зарегистрировать новый аккаунт почты</b>
<br>Для этого запроса необходимо прописать в теле запроса <i>{{$randomUserName}}</i>, для того, чтобы домен почты при каждом новом запросе был автоматически разным (через 10 минут почта становится неактивной). Также необходимо прописать в Environment переменное окружение под названием Email, где будет хранится текущая почта. Чтобы не прописывать значение Email каждый раз вручную для последующих запросов, нужно перейти во вкладку Test и прописать скрипт, который будет вносить в переменное окружение Email значение автоматически:
<br><i>var jsonData = pm.response.json ();
<br>pm.environment.set("Email", jsonData.address);</i>
<br>Для упрощения пароль будет статичен и одинаков.

+ <b>POST Залогиниться в систему с получением токена</b>
<br>Для этого запроса необходимо прописать в Environment переменное окружение под названием AuthToken, где будет храниться токен, выданный в теле ответа для текущего аккаунта почты. Чтобы не прописывать значение AuthToken каждый раз вручную, нужно перейти во вкладку Test и прописать скрипт, которые будет вносить в переменное окружение AuthToken значение автоматически:
<br><i>var jsonData = pm.response.json ();
<br>pm.environment.set("AuthToken", jsonData.token);</i>
<br>В теле запроса также прописывается <i>{{Email}}</i>, значение которого берется из Environment.

+ <b>GET Проверить аккаунт почты в системе с токеном авторизации</b>
<br>Во вкладке Authorization в поле токен прописать <i>{{AuthToken}}</i>, для того, чтобы проверка совершалась именно для текущего аккаунта с токеном, который автоматически задался через скрипт в предыдущем запросе на логин. В последующих запросах вкладка Authorization будет иметь аналогичное значение.

+ <b>GET Проверить новые сообщения</b>
<br>Для проверки корректности этого запроса можно отправить на созданный аккаунт сообщение, содержание которого будет видно в теле ответа.

+ <b>DEL Удалить полученное сообщение</b>
***
