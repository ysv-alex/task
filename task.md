# Реализация части функционала автомойки

по данной диаграмме, построить базу
https://drawsql.app/alexxx/diagrams/laravel

простая линия это связь один ко одному
линия с ветками это связь один ко многим

varchar, в laravel используем string
is_send_notify - по умолчанию поставить false
status - по умолчанию поставить 0

## В таблицах

    при создании миграции, по умолчанию будет поле
    timestamps
    оно добавляет 2 поля
    created_at, updated_at

в таблице orders, настроить soft delete
```php
$table->softDeletes();
```
расширить модель, добавив трейт, пример из документации
https://laravel.su/docs/8.x/eloquent#soft-deleting
это касается только таблицы **orders**

## Дизайн

Марка автомобиля, автомобили, категории услуг, услуги - можно сделать как у меня в интерфейсе, т.е на примере posts\categories

Клиенты, машины клиентов, работники, заказы - сделать в новом шаблоне, я пример накидал как должно быть
Index page - вывод таблицы, с фильтрами, и с кнопкой создать
Edit page, Create page - это просто страница с формами по созданию и удалению
![enter image description here](https://i.gyazo.com/8b62489b0e2c0d2e12b4bd2c6a178b36.png)

## Фильтры

**Страницы:**

 - customers - по всем полям
 - customer cars - по всем полям, кроме car, image
 - employees - по всем полям, кроме image
 - orders - по всем полям, кроме customer_car, если сможете, то сделайте
   и по ней
   image в таблицах выводить не нужно

## Загрузка файлов

везде где есть поле image, нужно загрузить изображение
в интерфейсе, в теге form, в атрибуте enctype используем multipart/form-data
http://htmlbook.ru/html/form/enctype

пример из документации
https://laravel.su/docs/8.x/requests#files
https://laravel.su/docs/8.x/filesystem#file-uploads

нужно ввести команду
```php
php artisan storage:link
```
## Enum поле

в таблице orders
есть поле status
0 - означает создан
1 - означает завершен

в приложении enum, делают по разному
в бд либо хранят текст (created, completed) либо цифры (0, 1), у нас будут цифры

пример, в проекте в папке App создайте папку Enums и там сам enum, Status.php
```php
enum Status: int
{
    case CREATED = 0;
    case COMPLETED = 1;
}
```
это пример со string, но вы используйте вариант с int
```php
enum Status: string
{
    case CREATED = 'created';
    case COMPLETED = 'completed';
}
```
в php enum появился не давно, поэтому чтоб получить значение enum поля, нужно вызвать value, автоматической сериализации нет
пример:
```php
Status::COMPLETED->value
```
## Страница orders

по связям вывести все данные, т.е в таблице не должно быть service_id и т.д, а реальные значения
(смотреть мой пример посты-категории)

в orders реализованы soft delete, поэтому должен быть фильтр для вывода удаленных заказов,
у модели нужно вызвать метод
```php
onlyTrashed
```

## Страница customer cars
по связям вывести все данные, т.е в таблице не должно быть car_id и т.д, а реальные значения
(смотреть мой пример посты-категории)

В таблице вывести цену, цену взять из услуги


## Фронтенд
в laravel встроен webpack для сбора всей статики (js,css)
в моих примерах используется 
https://tailwindcss.com/docs/
можно почитать  документацию, там в сайдбаре есть примеры utility классов
для запуска webpack  используем команду

    sail npm watch
запустится ватчер который будет следить за файлами и автоматически билдить статику

кому такой подход не нравится и где-то нужно будет добавить свои стили в BEM, 
то пишем css в /public/app.css
соо-о sail npm watch запускать не нужно

в сайдбаре можно закоментировать
посты, категории, авторов, книги, чтоб не занимали место

под новые сущности иконки можно брать отсюда
https://heroicons.com/

## Логика
у одного клиента может быть много машин
я упростил создание заказа в интерфейсе, если посмотреть связи, то при создании заказа нужна именно машина клиента
поэтому в UseCase, OrderService при создании заказа, находим клиента и берем его машину, рандомную или 1ую

---

под все сущности создаем UseCase - сервисы и работаем через них, кода в контроллерах должно быть минимум

---

под все действия по созданию\редактированию создаем свои Request
там же делаем валидацию (смотрите примеры в коде)

---

DTO создавать не обязательно, можно взять пример, когда массив возвращается из объекта Request, только назвать нормально, по имени сущности
```php
public function getPostCreateDto(): array
{
    return [
        'title' => $this['title'],
        'slug' => $this['slug'],
        'excerpt' => $this['excerpt'],
        'content' => $this['content'],
    ];
}
```
эти данные передаем в сервисы

---

**brands, cars, service_categories**
с этими сущностями можно не создавать Request, а писать валидацию в контроллерах, т.к там всего одно поле, и также можно не создавать под них сервисы в UseCase, в общем update,delete,create записи сделать в контроллерах

---

при создании заказа, если у клиента is_send_notify - true
то отправить письмо, что заказ создан

в самом заказе должна быть кнопка завершить заказ, по которому order обновляется и статус меняется на завершен

---

для **brands, cars, service_categories**, да и для других сущностей можно использовать фабрики по созданию, примеры есть в коде и в презентации
https://laravel.su/docs/8.x/seeding

---

реализовать проверки на удаления сущностей
