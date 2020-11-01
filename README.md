# Cifro
Проект команды <b>FromStack0verFlow</b> на хакатоне Лидеры Цифровой Трансформации по теме Автоматизированный анализ архивов фото и скан-копий технических и контрактных документов. 

# Описание задачи
Распознать множество скан-копий технических и контрактных документов с определением их структурных элементов, классифицировать по заданному набору признаков. Результаты распознавания организовать в виде базы данных с возможностью полнотекcтового поиска и фильтрации по параметрам. Предоставить пользовательские интерфейсы как для работы с полученными результатами, так и для последующего самостоятельного распознавания документов, возможность построения аналитических панелей и отчетов, а также REST API для бесшовной интеграции.

# Кластеризация

Для решения задачи классификации мы используем кластеризацию, что значительно увеличивает возможности масштабируемости.
Данный подход позволяет автоматически делать классы страниц, даже тех существование которых было неизвестно.
Используем для векторизации латентный слой свёрточного автоэнкодера на TensorFlow Keras.

По целевым страницам результат следующий: 10% ошибки первого рода, 3% ошибки второго рода.

![class](class.png)

/DL/pdf2img.ipynb преобразует PDF файлы в картинки по страницам и складывает в папку img.

/DL/pdf2text-mult.ipynb мультипоточно распознаёт все страницы Tesseract OCR из PDF и складывает в папку txt.

/DL/sort.ipynb кластерезует картинки из папки img и складывает из в папку sort по классам.


# Распознавание символов и объектов в изображениях
С помощью алгоритмов компьютерного зрения нам удалось распознать для такого класса документа как свидетельство об утверждении архитектурно-градостроительного решения следующие атрибуты:

* Номер

* Административный округ

* Район

* Адрес

* Наименование объекта

* Функциональное назначение объекта

![photo_2020-11-01_01-28-55](https://user-images.githubusercontent.com/42088646/97798742-ea076980-1c39-11eb-8ef7-e129e02b3daa.jpg)

Полученные данные были добавлены в базу данных sqlite.

recognition_table/documents/dataset_img/class_1 - изображения данного класса

recognition_table/LCT.db - база данных

recognition_table/class1.py - распознование структуры таблицы и извлечение нужных атрибутов


# Application programming interface
**REST API** разработан с целью бесшовной интеграции к другим сервисам системы автоматизированного анализа архивов фото и скан копий технических и контрактных документов.
**API** представлен в альфа версии и сопровождается ~~User-Frendly документацией написанной с помощью фреймворка **Swagger**.<br>Ресурс: **/api/docs**~~ 

### Авторизация и аутентификация
**JSON Web token + Basic auth**

**Basic auth** - это метод для агента пользователя HTTP, который предоставляет имя пользователя и пароль при выполнении запроса.
Шифрование пароля осуществляется по стандарту **PBKDF2** с алгоритмом **SHA512** в 200 000 итераций.

**JSON Web Token (JWT)** — это JSON объект, который определен в открытом стандарте RFC 7519. Он считается одним из безопасных способов передачи информации между двумя участниками. Для его создания необходимо определить заголовок (header) с общей информацией по токену, полезные данные (payload), такие как id пользователя, его роль и т.д. и подписи (signature). 

**Для получения такена аутентификации вам нужно:**

Авторизоваться по логину и паролю обратившись к ресурсу /api/auth
```bash
curl -u Testuser:password -X POST http://127.0.0.1:5000/api/auth
```
Результатом успешной авторизации будет токен:
```json
{
  "Access token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhdXRoIiwiZXhwIjoxNjAzNTk3NzYwLjAsImlhdCI6IjFkQ1cyZ2NBZGg1VjJDcTE5aHVsNyIsImp0aSI6MTYwNjE4OTc2MC4wfQ.qBMk0N9c9PI4jI_nPWf8zsu3-s_Fm9SqsoyyAaJPItnkJnGOqo0LQe_2e0y8ZdhsYcPlT3CkYOBmwcv0vrRu9g"
}
```

Используйте токен в HTTP заголовке:
```x-access-tokens: token```

### Используемые методы HTTP (Соглашения Restful) для ресурсов: 
| Ресурс | POST | GET |	DELETE | PUT PATCH |
|--------|------|-----|-------|:------------:|
| api/documents | Добавление нового документа(ов) |	Получение всех документов | **DROP** | **DROP** 
| api/documents/**ident** | **DROP** | Получение сведений для документа **ident** |	Удаление документа **ident**|	**DROP** 

**Использование фильтров**

| Метод | Ресурс | Фильтр | Описание
|:-----:|--------|----------|-------|
|GET| api/documents | ?type=type | Получение списка документов по заданному типу {type: str} 
