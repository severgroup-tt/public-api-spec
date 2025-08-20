Инструкция описывает, как получить доступ к API, имея под рукой только браузер и freeware-приложения. Инструменты для выполнения данных действий вы можете выбирать на свое усмотрение и личный опыт.

1. Сначала нужно создать приложение с нужными правами в админке. 
    *  Откройте панель администратора, выберите пункт “Приложения” и нажмите “Создать приложение”. 
    *  Укажите название приложения
    *  Определите, какие доступы нужны данному приложению. Выберите скоупы (права) для данного приложения. dp - цифровой профиль, team - команды, user - пользователи, tasks - цели, checkpoints - отчёты; read - права на чтение, write - права на запись, delete - права на удаление. 
    *  Если предполагается использование Oauth 2.0, поставьте флаг “подключить openID, укажите нужную ссылку для редиректа . 
    *  Нажмите “создать приложение”, появится окно с приватным ключом. 
    *  Нажмите “Копировать” и сохраните приватный ключ (скопируйте в блокнот).
    ![image](https://user-images.githubusercontent.com/48784397/193005985-0c7cc3cb-5c1d-4049-8bfd-8d204ed95082.png)

**Redirect_url** https://archivist.talenttech.ru/app/me

2. Скачайте puttygen https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html  , сформируйте из приватного ключа публичный, скопируйте в блокнот сформированный публичный ключ, они оба скоро понадобятся.
Для оболадателей MacOS можно выполнить в консоле следующую команду для получения публичного ключа из приватного: 
```
openssl rsa -in путь_к_приватному_ключу -pubout > путь_к_публичному_ключу
```
4. Теперь создайте запрос, который вернет вам авторизационный токен для дальнейшей работы. Срок действия токена составляет 7 дней.
    *  Скачайте postman https://www.postman.com/downloads/ , установите, 
    *  нажмите File > Import > Raw text, 
    *  вставьте туда текст curl запроса ниже, 
    *  нажмите Continue и Import. 
      ```
      curl --location --request POST 'https://api.public-api.talenttechlab.com/core/v1/auth/authorize' \
      --header 'accept: application/json' \
      --header 'Content-Type: application/json' \
      --data-raw '{
        "token": ""
      }'
      ```
4. Отобразится добавленный вами запрос. Переключитесь на вкладку Body.
![image](https://user-images.githubusercontent.com/48784397/193006588-a5a71d4e-4b66-486a-beec-4e4a5ea744f3.png)
5. Теперь нужно создать токен, истекающий через 30 секунд.  
![image](https://user-images.githubusercontent.com/48784397/193007223-5a9c57fe-d435-4deb-9117-ac12497b847f.png)
    *  Откройте сайт https://dinochiesa.github.io/jwt/  ,  
    *  в выпадающем меню Algorithm выберите RS-256. 
    *  В поле PAYLOAD: DATA введите 
      ```
      {
        "iss" : "5b6b2470-d1b0-0139-a07d-7be4b2c2fab",
        "exp" : 1627462923,
        "alg" : "RS256"
      }
      ```
    *  В поля для ввода приватного и публичного ключа(VERIFY SIGNATURE) введите свои ключи в том же формате, как находящиеся там примеры.
    *  Откройте созданное приложение в админпанели Платформы, 
    *  скопируйте его ID из адресной строки и замените им id, указанный в поле PAYLOAD: DATA под заголовком iss
    ![image](https://user-images.githubusercontent.com/48784397/193007326-885ce4f8-cfd4-4cc9-9dd7-5b5ea1aac708.png)
    
6. Откройте сайт https://www.epochconverter.com/ , найдите строку “The current Unix epoch time is “, 
    *  скопируйте число из этой строки (**с этого момента у вас есть 30 секунд на дальнейшие действия**)
    *  вернитесь на [http://jwt.io](https://dinochiesa.github.io/jwt/)  и вставьте число в поле PAYLOAD: DATA вместо числа под заголовком “exp”. 
    *  Отредактируйте вставленное число, прибавив к нему 30. Например, если на сайте epochconverter было число 1627463000, то его нужно изменить на 1627463030. Делается это, чтобы срок действия созданного нами токена составил 30 секунд от текущего момента. 
7. Токен должен сгенерироваться автоматически и появиться в поле “Encoded”. 
    *  Скопируйте его, 
    *  переключитесь на Postman, 
    *  вставьте токен в кавычки (см. положение курсора в п.4) . 
    *  Нажмите Send.
8. В поле Body придет ответ. Это нужный нам токен сроком действия 7 дней. 
![image](https://user-images.githubusercontent.com/48784397/193007577-14ee78c4-b59b-4d66-95f3-ce364f909302.png)
9. Чтобы убедиться, что он работает, запросим список команд. Для этого импортируйте еще один запрос аналогично описанному в п.3. В --header 'Authorization: Bearer укажите токен, полученный в п.8
    ```
    curl --location --request GET 'https://api.public-api.talenttechlab.com/core/v1/team' \
    --header 'accept: application/json' \
    --header 'Content-Type: application/json' \
    --header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhbGciOiJSUzI1NiIdImFwcF9pZCI6ImUyOWY4MGMwLThhMTMtNDEwNC04ODE5LTdlNDRkMmNiZjBlOSIsImV4cCI6MTY1NzI4NzU0MCwiaXNzIjoiZTk4NGFhYjAtMjZkNy00NzkyLWFkMmItZWViYzM1N2U3ODcxIn0.mDnmLZLttwUdSo0El3UnQrfTmhdWPemwWupl1_C5t_0IoBAPCU760Ek53eXoo9_hsywVdJ8lcycVOIsQOw6LcujhneZni0zfbDMDyxAoQbMYAjUlXiq8bL42-hPPwPxuQwkmbbhyRpMdZslP4U3J5B2u11QvqWqHEN4_YbKi2jpbDwxhjNjctn4AeQ2jBfs2daWBV68RyJTJQzZ8d5XqTitOLYSssKdl-8lXpEUVsv3jk-CY8cGmFiq1j66iomXMQbFx96NR5nLWA-Mob_h2aUvVB8paZs0XM7ABQBJ6OQem6AeWvsNbbUO5LdReQqIGKum3kRSA2UWEpDsIgrLNs-Ed4FHM6rLM-rSOWPP4HsR7ZTRLIbhAPURFcqlTAF3uVYlGu_P6kw9AiGA9WOEjoeVNnLdud1U5n93IVGJBEELtGFkv8awJkjZQpiHOTJO2Q4Yqc3JSYqga4UzygVv8s8_UxuR3xROhgMNEUODbx2601DcoaPF8ind_aPKW2zZiON8beUmxA6x-IFIQjItlFtK0xWfX0fmKBo8XglUVaphoQ1FK78w2VlhMrIa6VO5RPGo4Svog5AHdOXk08eMSrBehuqYiHwgN7M9VmeOMYtZAbEhiTNqzU_5sOiTFOquvDPekEdXHQ3wcmQ9ox2LztRazw_8Aq3p3gsNqCvvqUus' \ 
    ```
    ![image](https://user-images.githubusercontent.com/48784397/193007813-5f5fa3b4-adf7-450a-89af-be352fca64b7.png)
10. Остальные запросы к Public API авторизуются тем же токеном. Через 7 дней нужно будет получить новый токен по аналогичному сценарию.
11. Ваши разработчики могут реализовать автоматическое получение токена раз в 7 дней написав скрипт согласно данной инструкции и запуская его автоматически по расписанию.
