API для работы с сервисом Яндекс.Фотки
======================================

Функционал
----------
- [Получение Fimp-токена](https://github.com/dmkuznetsov/php-yandex-fotki/wiki/%D0%9F%D0%BE%D0%BB%D1%83%D1%87%D0%B5%D0%BD%D0%B8%D0%B5-Fimp-%D1%82%D0%BE%D0%BA%D0%B5%D0%BD%D0%B0) по логину/паролю
- Получение OAuth-токена [->](http://api.yandex.ru/fotki/doc/overview/oauth-token.xml)
- [Получение сервисного документа](https://github.com/dmkuznetsov/php-yandex-fotki/wiki/%D0%9F%D0%BE%D0%BB%D1%83%D1%87%D0%B5%D0%BD%D0%B8%D0%B5-%D1%81%D0%B5%D1%80%D0%B2%D0%B8%D1%81%D0%BD%D0%BE%D0%B3%D0%BE-%D0%B4%D0%BE%D0%BA%D1%83%D0%BC%D0%B5%D0%BD%D1%82%D0%B0)
- Загрузка изображения [->](http://api.yandex.ru/fotki/doc/concepts/add-photo.xml)
- [Постраничная выдача коллекций](http://api.yandex.ru/fotki/doc/operations-ref/collection-partial-lists.xml)
- [Получение данных альбома](http://api.yandex.ru/fotki/doc/operations-ref/album-get.xml)
- Добавление нового альбома [->](http://api.yandex.ru/fotki/doc/operations-ref/albums-create.xml)
- Редактирование альбома [->](http://api.yandex.ru/fotki/doc/operations-ref/album-edit.xml)
- Удаление альбома [->](http://api.yandex.ru/fotki/doc/operations-ref/album-delete.xml)
- [Получение данных фотографии](http://api.yandex.ru/fotki/doc/operations-ref/photo-get.xml)
- Редактирование фотографии [->](http://api.yandex.ru/fotki/doc/operations-ref/photo-edit.xml)
- Удаление фотографии [->](http://api.yandex.ru/fotki/doc/operations-ref/photo-delete.xml)
- Получение данных тега [->](http://api.yandex.ru/fotki/doc/operations-ref/get-tag.xml)
- Добавление тега [->](http://api.yandex.ru/fotki/doc/operations-ref/add-tag.xml)
- Редактирование тега [->](http://api.yandex.ru/fotki/doc/operations-ref/edit-tag.xml)
- Удаление тега [->](http://api.yandex.ru/fotki/doc/operations-ref/delete-tag.xml)
- [Получение данных коллекции альбомов](http://api.yandex.ru/fotki/doc/operations-ref/albums-collection-get.xml)
- Добавление нового альбома [->](http://api.yandex.ru/fotki/doc/operations-ref/albums-collection-create.xml)
- [Получение данных коллекции фотографий](http://api.yandex.ru/fotki/doc/operations-ref/album-photos-collection-get.xml)
- Загрузка изображения в альбом [->](http://api.yandex.ru/fotki/doc/operations-ref/album-photos-collection-add.xml)
- [Получение данных общей коллекции фотографий](http://api.yandex.ru/fotki/doc/operations-ref/all-photos-collection-get.xml)
- Загрузка изображения в общую коллекцию фотографий [->](http://api.yandex.ru/fotki/doc/operations-ref/all-photos-collection-add.xml)
- Получение данных коллекции тегов [->](http://api.yandex.ru/fotki/doc/operations-ref/tag-collection-get.xml)
- Получение данных коллекции фоторафий тега [->](http://api.yandex.ru/fotki/doc/operations-ref/tag-photos-collection-get.xml)
- Получение коллекции новых интересных фотографий [->](http://api.yandex.ru/fotki/doc/operations-ref/interesting-photos-get.xml)
- Получение коллекции популярных фотографий [->](http://api.yandex.ru/fotki/doc/operations-ref/top-photos-get.xml)
- Получение коллекции "Фото дня" [->](http://api.yandex.ru/fotki/doc/operations-ref/day-photos-get.xml)


Примеры
-------

```php
$api = new \Yandex\Fotki\Api($login);

// Загрузка сервисного документа, в котором ссылки на коллекции
// пункт необязательный, но в Яндексе написано, что ссылки могут
// когда-нибудь измениться
$api->loadServiceDocument()

// Аутентификация
// Принимает пароль от аккаунта или fimp-token
// Внимание! При получении Fimp-токена - Яндекс часто отвечает 502 ошибкой
try {
    $api->auth($passwordOrToken);
} catch(\Yandex\Fotki\Exception\ServerError $ex) {
    // Яндекс ответил 502. Повторите попытку снова. Как правило, раза с 5 удается получить токен
} catch(\Yandex\Fotki\Exception\Api\Auth $ex) {
    // Что-то с самой авторизацией (см. $ex->getMessage())
}

// Сохраните токен куда-нибудь. По документации - время жизни токена неограничено
$token = $api->getAuth()->getToken();

// Получение коллекции альбомов
$collection = $api->getAlbumsCollection()
    ->setLimit(5) // лимит на загрузку 5 альбомов
    ->load();
$collection->getList();

// Загрузка всей коллекции
$collection = $api->getAlbumsCollection()->loadAll();

// Постраничная загрузка коллекции
try {
    $collection = $api->getAlbumsCollection()->load()->next()->next()->next();
} catch(\Yandex\Fotki\Exception\Api\StopIteration $ex) {
    // Окончание постраничной навигации
}

// Список альбомов в коллекции
$albums = $collection->getList();

$album = null;
// Проходимся по альбомам коллекции (берем первый попавшийся)
foreach($albums as $id => $item) {
    $album = $item;
    break;
}

// Работа с альбомом - это работа с коллекцией фотографий
// и мета-информацией об альбоме.
// Загружаем все фотки альбома
$photos = $album->loadAll()->getList();
```

По всему коду прописаны php-doc комментарии, поэтому в IDE должны быть нормальные подсказки по методам у объектов.

### Кэширование

Для объектов API реализован интерфейс Serializable, что позволит вам сохранять полученные объекты в текстовом представлении.

```php
$api = new \Yandex\Fotki\Api($login);
$api->auth($token);

// Загружаем все альбомы в коллекцию
$collection = $api->getAlbumsCollection()->loadAll();

// Сериализуем коллекцию (и можем сохранить в кэш, например)
echo serialize($collection);
```