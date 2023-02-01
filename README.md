Данный репозиторий показывает мои практические навыки работы в Postman. Вы можете импортировать коллекцию и переменное окружение в Postman и проверить меня.
Для примера использован публичный API для создания временных аккаунтов почты (10 Minute Mail API https://docs.mail.gw). Логика сервиса заключается в том, что пользователю выдается аккаунт почты, который блокируется через 10 минут (удобно для прогонов кейсов с авторизацией, чтобы не использовать свой аккаунт).
Ниже представлены запросы с небольшим описанием логики и пояснения к частичной  автоматизации процессов.

GET Получить список доменов, доступных для регистрации аккаунта почты
В теле ответа будет видны доступные домены для регистрации. Это самый первый запрос в цепочке запросов, поэтому для него прописан скрипт проверки соответствия статус кода (а также для всех последующих GET запросов). Скрипт сравнивает фактический и ожидаемый статус код. Принимается, что ожидается 200 статус код.
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

POST Зарегистрировать новый аккаунт почты
Для этого запроса необходимо прописать в теле запроса {{$randomUserName}}, для того, чтобы домен почты при каждом новом запросе был автоматически разным (через 10 минут почта становится неактивной). Также необходимо прописать в Environment переменное окружение под названием Email, где будет хранится текущая почта. Чтобы не прописывать значение Email каждый раз вручную для последующих запросов, нужно перейти во вкладку Test и прописать скрипт, который будет вносить в переменное окружение Email значение автоматически:
var jsonData = pm.response.json ();
pm.environment.set("Email", jsonData.address);
Для упрощения пароль будет статичен и одинаков.

POST Залогиниться в систему с получением токена
Для этого запроса необходимо прописать в Environment переменное окружение под названием AuthToken, где будет храниться токен, выданный в теле ответа для текущего аккаунта почты. Чтобы не прописывать значение AuthToken каждый раз вручную, нужно перейти во вкладку Test и прописать скрипт, которые будет вносить в переменное окружение AuthToken значение автоматически:
var jsonData = pm.response.json ();
pm.environment.set("AuthToken", jsonData.token);
	В теле запроса также прописывается {{Email}}, значение которого берется из Environment.

GET Проверить аккаунт почты в системе с токеном авторизации
Во вкладке Authorization в поле токен прописать {{AuthToken}}, для того, чтобы проверка совершалась именно для текущего аккаунта с токеном, который автоматически задался через скрипт в предыдущем запросе на логин. В последующих запросах вкладка Authorization будет иметь аналогичное значение.

GET Проверить новые сообщения
Для проверки корректности этого запроса можно отправить на созданный аккаунт сообщение, содержание которого будет видно в теле ответа.
DEL Удалить полученное сообщение
