# Websm/Framework/Router
Класс Router является «оберткой» над стандартными функциями PHP по работе с HTTP запросами и представляет удобный инструментарий для их обработки сервером.
## Запросы протокола HTTP
Протокол HTTP определяет множество запросов, которые указывают, какое желаемое действие выполнится для данного ресурса. Так как эти запросы, являются подмножеством протокола, для любого бэкенда, будь он на Python, PHP, NodeJS, Ruby или C# они являются абсолютно идентичными. Вот некоторые из них:
* GET;
* POST;
* PUT;
* DELETE;
* PATCH.

Обращаю Ваше внимание, что это неполный список запросов, определяемый протоколом HTTP. С полный списком, а также с их спецификацией Вы можете посмотреть по ссылке: https://developer.mozilla.org/ru/docs/Web/HTTP/Methods.

## Суперглобальные переменные PHP

Язык PHP имеет в своем стандартном функционале суперглобальные переменные. Их отличие от обычных глобальных переменных заключается в том, что они видны в любой области видимости кода, а также в том, что они зарезервированы языком и их нельзя переопределить. Суперглобальные переменные начинаются с $_ Простым языком, мы может из любого места в коде обратится к этим переменным и получить их значение. Например, если в каком-то месте кода, мы хотим узнать IP адрес клиента, с которого происходит запрос на сервер, мы можем обратится к глобальной переменной $_SERVER. Суперглобальная переменная $_SERVER представляет собой, по сути, ассоциативный массив (массив пар ключ – значение). А значит, обращаясь к ключу REMOTE_ADDR, мы можем получить IP – адрес клиента:
```php
$ip = $_SERVER['REMOTE_ADDR'];
```
HTTP запросы, которые мы обсуждали ранее хранятся в суперглобальных переменных языка PHP. Для запроса GET определена суперглобальная переменная $_GET, для запроса POST - $_POST. Для остальных запросов из списка суперглобальных переменных не определено, но их значение мы можем их получить, обращаясь к переменной SERVER по ключу REQUEST_METHOD:
```php
$method = $_SERVER['REQUEST_METHOD'];
```
С полным списком суперглобальных переменных PHP, вы можете ознакомится по ссылке https://www.php.net/manual/ru/reserved.variables.server.php 
Особого внимания заслуживает метод GET. Метод GET, по сути, это то, что присутствует в адресной строке браузера при обращении к серверу. Именно он используется PHP для определения того, какую страницу нужно вернуть клиенту, то есть для маршрутизации. Метод GET в отличии от других вопросов не может иметь тела, то есть мы не можем послать с ним дополнительную информацию на сервер в виде массива XML или JSON. Оставшиеся запросы не представляют существенных различий в работе для класса Router. Название указывает на то, что собирается сделать, тот или иной запрос. Традиционно, GET используется для получения данных с сервера. PUT, POST – для отправки, DELETE – для удаления, PATCH – для изменения. Что, однако, не мешает программисту добавлять данные с помощью DELETE, а удалять с помощью PUT, что естественно считается плохой практикой при написании кода.
## Маршрутизация
_Маршрутизация_ – это набор правил, по которым клиент взаимодействует с сайтом. Такими правилами выступают маршруты.
_Маршрут_ – заранее определенный обработчик запроса клиента, которым он может пользоваться. Маршруты объединяются в группы маршрутов, как правило по признаку работы с тем или иным классом/сущностью PHP. Группы объединяются в маршрутизатор. Маршрутизатор – это класс PHP, который объединяет в себе все правила маршрутизации сайта. В Websm/Framework маршрутизатором является класс Router. Давайте разберем принцип его работы.
Если бы такого класса не существовало, то мы бы получали маршрут следующим образом:
```php
if (isset[$_GET]){
	echo "Маршрут: ".$_GET;
}
```
Это означает буквально следующее: если переменная $_GET определена, то есть внутри что-то действительно есть, тогда выводим её значение. Это справедливо и для других вариантов запросов. Именно этот функционал с некоторыми изменениями и спрятан внутри класса Router. Программисту не нужно писать этот код каждый раз, когда он хочет обработать маршрут. Рассмотрим работу класса Router и его методы сразу на конкретном примере и приведем графическое представление.
Для того, чтобы воспользоваться классом Router его нужно инициализировать. Традиционно это делается в файле index.php. Для инициализации используется метод init(). Инициализировать Router можно с параметром url, параметр url – это главный домен например www.domain.ru, по умолчанию init() всегда инициализируется с текущей переменной $_SERVER['REQUEST_URI']. Init является статичным методом и возвращает экземпляр класса Router.
```php
//index.php
use Websm\Framework\Router;

$router = Router::init();
```
Эта запись аналогична, простому созданию класса:
```php
$router = new Router();
```
Точнее она была бы аналогична, если бы в классе Router присутствовал конструктор. Но его нет, поэтому данная запись не сработает корректно. Причину, по которой конструктор отсутствует, я объясню позже.
Чтобы маршрутизатор начал следить за маршрутами, нужно вызвать метод start. Данный метод объединит в себе в себе маршруты и подмаршруты.
```php
//index.php
use Websm\Framework\Router;

$router = Router::init();
$router->start();
```
Давайте посмотрим, что представляет собой маршрутизация в данном случае.
![](https://github.com/knsroo/websm/blob/master/Router/dia.png?raw=true)
Теперь у нас есть Router, но он пуст. Давайте наполним его правилами. Предположим, у нас есть некий класс Controller в файле Controller.php, который реализует какой-то функционал и ему нужно добавить маршрутизацию. Для этого ему нужно создать функцию, традиционно её называют getRoutes(), которая вернет набор правил для работы с данным классом.
```php
//Controller.php
use Websm\Framework\Router;

class Controller
{
public function getRoutes(){}
}
```
Для того, чтобы вернуть набор правил надо создать экземпляр класса Router. В этот раз мы сделаем это с помощью метода group:
```php
//Controller.php
use Websm\Framework\Router;

class Controller
{
    public function getRoutes(){
        $group = Router::group()
        return $group;
    }
}
```
В данном случае переменная group, это такой же экземпляр, что мы создавали с помощью метода init с единственным различием. Когда мы создаем экземпляр класса Router с помощью метода group он никуда не привязывается по умолчанию, а просто существует. Это сделано для того, чтобы программист мог в будущем привязать маршрут туда, куда ему нужно. Этим объясняется отсутствие конструктора класса – способ, которым можно его создать не единственный.
Оговорка: Данный функционал можно было реализовать и с помощью конструктора, тогда бы в конструктор пришлось бы добавлять переменную, которая покажет, каким именно образом мы хотим его создать, а в самом конструкторе реализовывать условие и возвращать то или иное, в зависимости от его значения, что не совсем удобно как практически так и синтаксически.
После создания группы, мы должны ее возвращать, чтобы в будущем её можно было привязать.
Теперь мы можем добавлять маршруты. Маршруты добавляются с помощью методов класса Router:
* addGet;
* addPost;
* addPut;
* addDelete.

Данные методы принимают на вход строку – значение GET, и функцию которую нужно вызвать, при совпадении значения. Допустим, мы хотим, чтобы по маршруту pages, выполнялась функция getPages. Реализуем это:
```php
//Controller.php
use Websm\Framework\Router;

class Controller
{
    public function getRoutes(){
        $group = Router::group();
        $group->addGet('/pages', [$this, 'getPages']);
        return $group;
    }
    public function getPages(){}
}

```
Посмотрим на структуру маршрутизации:
![](https://github.com/knsroo/websm/blob/master/Router/dia1.png?raw=true)
Но если мы сейчас попробуем получить доступ к www.domain.com/pages
GET вернет 404, так как на данный момент Router, который находится в index.php, назовём его главным, не подозревает о существовании Controller. Для того, чтобы определить это нужно воспользоваться методом mount класса Router. Метод mount привяжет все методы нашей группы к главному маршруту
```php
//index.php
use Websm\Framework\Router;
use Controller;

$router = Router::init();
$router->mount(‘/’, (new Controller)->getRoutes());
$router->start();
```
Посмотрим на нашу структуру теперь:
![](https://github.com/knsroo/websm/blob/master/Router/dia2.png?raw=true)
Теперь у нас есть список правил:
Правило | Обработчик
---- | ----
www.domain.com/pages | getPages()
Добавим еще правило. Допустим, мы хотим чтобы отображалась главная страница. Создадим класс Index по аналогии с Controller и посмотрим на структуру маршрутизации.
```php
//IndexClass.php
use Websm\Framework\Router;

class Index
{
    public function getRoutes(){
        $group = Router::group();
        $group->addGet('/', [$this, 'getIndex']);
        return $group;
    }
    public function getIndex(){}
}

```
```php
//index.php
use Websm\Framework\Router;
use Controller;
use Index;

$router = Router::init();
$router->mount(‘/’,(new Controller)->getRoutes());
$router->mount(‘/’,(new Index)->getRoutes());
$router->start();
```
Структура:
![](https://github.com/knsroo/websm/blob/master/Router/dia3.png?raw=true)
Список правил:
Правило | Обработчик
---- | ----
www.domain.com | getIndex()
www.domain.com/pages | getPages()
Добавим пару запросов другого типа. Предположим, что у нас есть функционал добавления новой страницы (addPage) и удаления существующей страницы (delPage). Замечу, если маршруты имеют одинаковые имя правила, но разный метод, они воспринимаются маршрутизатором как разные маршруты. Добавим эти маршруты на правило pages.
```php
//Controller.php
use Websm\Framework\Router;

class Controller
{
    public function getRoutes(){
        $group = Router::group();
        $group->addGet('/pages', [$this, 'getPages']);
        $group->addPost('/pages', [$this, 'addPage']);
        $group->addDelete('/pages', [$this, 'delPage']);
        return $group;
    }
    public function getPages(){}
    public function addPage(){}
    public function delPage(){}
}
```
Структура:
![](https://github.com/knsroo/websm/blob/master/Router/dia4.png?raw=true)
Список правил:
Правило | Метод | Обработчик
---- |----| ----
www.domain.com | GET | getIndex()
www.domain.com/pages | GET | getPages()
www.domain.com/pages | POST | addPage()
www.domain.com/pages | DELETE | delPage()
Как быть если необходимо добавить запрос, метод на который отсутствует в классе? Например, PATCH. Здесь следует воспользоваться методом add класса Router, который добавит любой требуемый метод. При этом необходимо указать его название:
```php
//Controller.php
use Websm\Framework\Router;

class Controller
{
    public function getRoutes(){
        $group = Router::group();
        $group->addGet('/pages', [$this, 'getPages']);
        $group->addPost('/pages', [$this, 'addPage']);
        $group->addDelete('/pages', [$this, 'delPage']);
        $group->addDelete('/pages', [$this, 'delPage']);
        $group->add('PATCH', '/pages', [$this, 'patchPage']);
        return $group;
    }
    public function getPages(){}
    public function addPage(){}
    public function delPage(){}
    public function patchPage(){}
}
```
## Деструктуризация
Предположим, необходимо добавить страницы личного кабинета и новостей, в которых много функционала. Поскольку $group это экземпляр класса Router мы можем деструктурировать схему. Создадим еще два класса Lk и News и добавим им обработчики getDefault. Затем, с помощью функции mount привяжем их к классу Index.
```php
//Lk.php
use Websm\Framework\Router;

class Lk
{
    public function getRoutes(){
        $group = Router::group();
        $group->addGet('/lk', [$this, 'getDefault']);
        return $group;
    }
    public function getDefault(){}
}
```
```php
//News.php
use Websm\Framework\Router;

class News
{
    public function getRoutes(){
        $group = Router::group();
        $group->addGet('/', [$this, 'getDefault']);
        return $group;
    }
    public function getDefault(){}
}
```
```php
//IndexClass.php
use Websm\Framework\Router;
use News;
use Lk;

class Index
{
    public function getRoutes(){
        $group = Router::group();
        $group->addGet('/', [$this, 'getIndex']);
        $group->mount('/', (new Lk)->getRoutes());
        $group->mount('/news', (new News)->getRoutes());
        return $group;
    }
    public function getIndex(){}
}
```
Структура:
![](https://github.com/knsroo/websm/blob/master/Router/dia5.png?raw=true)
Список правил:
Правило | Метод | Обработчик
---- |----| ----
www.domain.com | GET | getIndex()
www.domain.com/lk | GET | getDefault()
www.domain.com/news | GET | getDefault()
www.domain.com/pages | GET | getPages()
www.domain.com/pages | POST | addPage()
www.domain.com/pages | DELETE | delPage()
www.domain.com/pages | PATCH | patchPage()
Таким образом можно деструктурировать файлы и создавать сколь угодно длинные иерархии.
## Параметры обработчика
Предположим, у нашей страницы новости будут дополнительные маршруты в виде самих новостей. Но новости – динамическая сущность, их может быть сколько угодно и их нужно идентифицировать. Для этого в обработчик функции подается специальная переменная. Традиционно, она называется $req, но назвать можно как угодно. В этой переменной будет хранится результат того, что обозначено специальным шаблоном - :. Попробуем создать маршрут с шаблоном в классе News, и добавим переменную в обработчик:
```php
//News.php
use Websm\Framework\Router;

class News
{
    public function getRoutes(){
        $group = Router::group();
        $group->addGet('/', [$this, 'getDefault']);
        $group->addGet('/:new', [$this, 'getDefault']);
        return $group;
    }
    public function getDefault($req){
        $chpu = $req['new'];
    }
}
```
Теперь всё, что после двоеточия в маршруте будет восприниматься как переменная. Таких переменных может быть сколь угодно. Если после переменной добавить ещё и знак +, то все последующие маршруты (/) также будут попадать в обработчик.
Ранее мы говорили, что если запросы разные, то если их присваивать одному и тому же маршруту, они будут восприниматься как разные. Но что, если запросы будут одинаковыми? Тогда выполнится тот, который будет первый в очереди.
```php
$group->addGet('/lk', [$this, 'getFirst']);  // Это выполнится
$group->addGet('/lk', [$this, 'getSecond']); // Это нет
```
Но что, если необходимо, чтобы оба они выполнились? Смоделируем такую ситуацию. Допустим, чтобы попасть на страницу личного кабинета, нужно сначала проверить авторизован ли пользователь. Добавим данный функционал в класс Lk:
```php
//Lk.php
use Websm\Framework\Router;

class Lk
{
    public function getRoutes(){
        $group = Router::group();
        $group->addGet('/lk', [$this, 'isAuth']);
        $group->addGet('/lk', [$this, 'getDefault']);
        return $group;
    }
    public function isAuth(){}
    public function getDefault(){}
}
```
Пока что маршрут видит только isAuth. Чтобы после проверки авторизации, выполнилась getDefault, в запрос необходимо передать переменную $next (так она называется традиционно, название может быть любым) и вызвать её. Но обязательно, нужно это сделать после переменной $req, так как $next принимается строго вторым параметром. Эта переменная и будет содержать следующий по списку маршрут. Если мы хотим, чтобы данная функция применилась ко всем типам запросов, целесообразно использовать метод addAll класса Router:
```php
//Lk.php
use Websm\Framework\Router;

class Lk
{
    public function getRoutes(){
        $group = Router::group();
        $group->addAll('/lk', [$this, 'isAuth']);
        $group->addGet('/lk', [$this, 'getDefault']);
        return $group;
    }
    public function isAuth($req, $next){
    	if (auth) $next();
    }
    public function getDefault(){}
}
```
Посмотрим на нашу структуру:
Список правил:
![](https://github.com/knsroo/websm/blob/master/Router/dia6.png?raw=true)
Правило | Метод | Обработчик
---- |----| ----
www.domain.com | GET | getIndex()
www.domain.com/lk | GET | isAuth() -> getDefault()
www.domain.com/news | GET | getDefault()
www.domain.com/pages | GET | getPages()
www.domain.com/pages | POST | addPage()
www.domain.com/pages | DELETE | delPage()
www.domain.com/pages | PATCH | patchPage()
## Именованные маршруты
Допустим в ходе выполнение операции нам необходимо обращаться к какому-либо маршруту чтобы, что-нибудь с ним сделать. Тогда, целесообразно указывать маршруту имя, чтобы в дальнейшем можно было к нему обратится. Имя маршрута указывается через метод setName класса Router:
```php
...
    public function getRoutes(){
        $group = Router::group();
        $group->addGet('/', [$this, 'getDefault']);
        $group->addGet('/:new', [$this, 'getDefault'])
        	->setName('new');
        return $group;
    }
    public function getDefault($req){
        $chpu = $req['new'];
    }
...
```
Тогда, где-нибудь в другом месте, мы можем обратится к маршруту с помощью byName и получить от него, к примеру, его url:
```php
...
$url = Router::byName('new')->getUrl();
...
```
## Что ещё можно делать?
Класс Router также содержит в себе следующие методы:

Метод |	Входные параметры |	Входные параметры по умолчанию |	Результат
---- | ---- | ---- | ----
instance | name	| null | 	Возвращает экземпляр класса Router по имени
normalizeUrl |	url |	'' |	Возвращает нормализованный url (без двойных слешей и т.п.)
setUrl |	url |	'' |	Устанавливает url для маршрутизатора
addAjax	| Pattern, call, options |	'', null, [] |	Добавляет Ajax запрос
route	| Pattern, options	| '', [] |	Добавляет маршрут без указания типа запроса
where	| Vars, exp |	Required, null |	Устанавливает параметры строки правила
getParent ||| Возвращает родителя экземпляра
getBasePath	|||	Возвращает путь маршрута, вместе с родителем
getAbsolutePath	|||	Возвращает абсолютный путь маршрута
getOrigin	|||	Возвращает строку домен
getURL ||| Возвращает полный URL
