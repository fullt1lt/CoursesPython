# Лекция 32: REST аутентификация. Авторизация. Permissions. Фильтрация. (3 часа)

![](image/REST_authentication.png)

## Идентификация, аутентификация и авторизация в API

Когда мы работаем с `Django REST Framework`, система аутентификации немного отличается от классического входа через форму. Здесь уже нет обычного логина и пароля в `HTML`-форме, вместо этого — запросы к `API`, заголовки, токены. Но суть процессов остаётся той же: и в `API`, и в веб-интерфейсе всё строится на трёх этапах — аутентификация, идентификация, авторизация.

### Аутентификация

Аутентификация в контексте `REST API` — это проверка того, что клиент действительно тот, за кого себя выдаёт. В отличие от веб-интерфейса, где пользователь вручную вводит логин и пароль, в `API` чаще всего используется заголовок авторизации, где передаётся токен или пара логин-пароль (в зависимости от метода).

В DRF за аутентификацию отвечают классы, подключаемые в `DEFAULT_AUTHENTICATION_CLASSES`. Они получают данные из запроса, проверяют их, и если всё в порядке — возвращают пользователя.

Если аутентификация не проходит, клиент получает ошибку `401 Unauthorized`.

Таким образом, в `API` мы не вызываем функцию `authenticate()` напрямую — всё происходит автоматически на уровне middleware и класса аутентификации.

Пример: клиент отправляет заголовок
`Authorization: Token abcdef123456`

`DRF` находит пользователя по этому токену и передаёт объект пользователя в `request.user`. Если токен невалидный или отсутствует — пользователь считается неаутентифицированным.

### Идентификация

После успешной аутентификации `DRF` определяет, какой именно пользователь делает запрос — и сохраняет его в объекте request.user. Это и есть идентификация. В отличие от аутентификации, которая отвечает на вопрос «можно ли доверять этим данным», идентификация связывает этот запрос с конкретным объектом пользователя в системе.

Можно сказать так:
— Аутентификация проверила, что токен валиден.
— Идентификация определила, что этот токен принадлежит пользователю с `id=5` и именем **manager**.

Дальше во всём `API` можно обращаться к `request.user`, чтобы получить этого пользователя.

Идентификация всегда следует после аутентификации. Если пользователь не прошёл аутентификацию, `request.user` будет установлено в `AnonymousUser`.

### Авторизация

Последний этап — авторизация. Она определяет, имеет ли данный пользователь право выполнять конкретное действие. Например:
 - Может ли он просматривать список всех заказов?
 - Разрешено ли ему редактировать чужой товар?
 - Доступен ли ему эндпоинт только для админов?

В DRF за авторизацию отвечают классы `permission_classes`, например `IsAuthenticated`, `IsAdminUser`, `IsAuthenticatedOrReadOnly` и другие.

**Пример:**

```python
from rest_framework.views import APIView
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response

class ProfileView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({"username": request.user.username})
```

Здесь авторизация работает так: если пользователь аутентифицирован — ему разрешён доступ к методу `get`, иначе — будет возвращена ошибка `403 Forbidden`.

Весь процесс целиком Если обобщить:
 - Сначала DRF извлекает данные из заголовка — это этап идентификации.
 - Далее проверяется валидность этих данных — это аутентификация.
 - Затем применяется логика разрешений — это авторизация.

Если все этапы прошли успешно — клиент получает доступ к ресурсу. Если где-то возникает проблема — система автоматически возвращает ошибку с нужным статусом (`401` или `403`).

## Виды аутентификации в Django REST Framework

В `Django REST Framework` можно подключить разные способы аутентификации. Каждый из них отвечает на один и тот же вопрос — можно ли доверять этому запросу, — но работает по-разному. Выбор способа зависит от того, с кем работает ваш API: с браузером, мобильным приложением, внешними сервисами или всем сразу.

Рассмотрим пять основных вариантов, которые чаще всего используются на практике:
 - `SessionAuthentication`
 - `BasicAuthentication`
 - `TokenAuthentication`
 - `JWT`
 - `OAuth2`

### SessionAuthentication

Это стандартный способ аутентификации, который встроен в `Django`. Он работает через `сессию` и `cookies`.

Когда пользователь проходит аутентификацию (например, через форму логина), Django сохраняет его `ID` в сессии. При каждом следующем запросе браузер автоматически отправляет `cookie` с `ID` сессии, и `Django` узнаёт пользователя. Такой способ удобен для браузерных клиентов и админок, где есть формы и `cookies`.

Для `DRF` это означает следующее: если мы используем `SessionAuthentication`, то клиент должен сначала аутентифицироваться в браузере (или через форму), а потом делать запросы — и только тогда `request.user` будет доступен.

Как подключить:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
    ]
}
```

**Важно:** этот способ зависит от `cookies` и `CSRF`, поэтому он плохо подходит для `API`, которыми пользуются мобильные клиенты, боты или сторонние сервисы.

### BasicAuthentication

Это самый простой способ — логин и пароль передаются в заголовке Authorization.

Клиент шлёт заголовок
`Authorization: Basic base64encoded(username:password)`

`DRF` получает его, расшифровывает, ищет пользователя и проверяет пароль. Если всё хорошо — пользователь считается аутентифицированным.

Как подключить:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.BasicAuthentication',
    ]
}
```

`BasicAuthentication` подходит для тестирования или очень простых внутренних сервисов. Использовать в продакшене — только через `HTTPS`.

Если нет `HTTPS`, логин и пароль передаются в открытом виде, и их можно перехватить.

### TokenAuthentication

Это один из самых распространённых способов в `REST API`. Суть простая:
- После входа пользователь получает уникальный токен.
- При каждом запросе он передаёт этот токен в заголовке.
- DRF проверяет токен и узнаёт пользователя.

Преимущество:
 - не нужно работать с сессиями и cookies;
 - клиент может быть любым — браузер, мобильное приложение, внешний сервис.

Чтобы использовать токены, нужно установить модуль authtoken(Если djangorestframework уже устанвовлен, то этот модуль уже находится в его составе):

```bash
pip install djangorestframework
pip install djangorestframework-authtoken
```
Затем добавить его в `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    ...
    'rest_framework.authtoken',
]
```

После этого нужно сделать миграцию, чтобы создать таблицу для токенов:

```bash
python manage.py migrate
```

Затем в настройках проекта нужно указать, что мы будем использовать токен для аутентификации:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ]
}
```

Теперь в модели пользователя будет поле `auth_token`, где хранится токен для каждого пользователя.

Чтобы пользователь получил токен, можно:
- сгенерировать его вручную в Django shell,
- или подключить стандартный view:

```python
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path('api-token-auth/', obtain_auth_token),
]
```
Теперь у нас есть эндпоинт `/api-token-auth/`, куда можно отправить POST-запрос с логином и паролем, чтобы получить токен.
Теперь можно отправлять POST-запрос с логином и паролем, и получать токен:

```json
POST /api-token-auth/
{
    "username": "admin",
    "password": "12345"
}
```

Ответ:

```json
{
    "token": "17cc41c7e5d90a0a023..."
}
```

После этого клиент может делать любые запросы с заголовком:

```makefile
Authorization: Token 17cc41c7e5d90a0a023...
```

### JWT (JSON Web Token)

`JWT` — это более современный способ аутентификации. Токен не просто указывает на пользователя в базе, он содержит зашифрованные данные: `id` пользователя, срок действия и т.д.

Преимущества:
- токен можно валидировать без обращения к базе;
- можно настроить срок жизни;
- можно хранить дополнительные данные.

Для подключения чаще всего используют библиотеку `djangorestframework-simplejwt`.

Установка:

```bash
pip install djangorestframework-simplejwt
```

Затем нужно добавить её в `INSTALLED_APPS`:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ]
}
```

Добавить `URL’ы`:

```python
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

Теперь клиент может отправить логин и пароль:

```json
POST /api/token/
{
    "username": "admin",
    "password": "12345"
}
```

И получить:

```json
{
    "access": "eyJ0eXAiOiJKV1QiLCJh...",
    "refresh": "eyJ0eXAiOiJKV1QiLCJh..."
}
```

`Access`-токен — для авторизации, действует ограниченное время (например, 5 минут). `Refresh`-токен — для получения нового `access`-токена.

### OAuth 2.0

Этот способ используется, когда вы хотите разрешить вход через внешние сервисы — например, `Google`, `Facebook`, `GitHub`. Он основан на токенах и требует отдельной настройки через `django-allauth` или `django-oauth-toolkit`.

На практике чаще всего `OAuth` применяют, когда ваш `API` — это часть большого проекта с внешними поставщиками авторизации.

### Пример настройки с `TokenAuthentication`

1. Установите необходимые пакеты:

```bash
pip install djangorestframework
pip install djangorestframework-authtoken
```
2. Добавьте `rest_framework` и `rest_framework.authtoken` в `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
]
```
3. Настройте `REST_FRAMEWORK` в `settings.py`:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}
```

4. Создайте токены для пользователей.

4.1 Это можно сделать через Django shell:

```bash
python manage.py shell
```

```python
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token
user = User.objects.get(username='admin')  # Замените 'admin' на нужного пользователя
token, created = Token.objects.get_or_create(user=user)
print(token.key)  # Вывод токена
```
4.2 Или через Postman, отправив POST-запрос на `/api-token-auth/` с логином и паролем:

```json
POST /api-token-auth/
{
    "username": "admin",
    "password": "12345"
}
```

![](image/REST_authentication_token.png)


5. Если добавить в заголовок запроса `Authorization: Token <ваш_токен>`, то запросы к эндпоинту  будут успешными, и вы получите данные о товарах.
Сделаем GET-запрос к `/api/products/` с заголовком.

![](image/REST_authentication_token_success.png)

## Permissions в Django REST Framework

После того как пользователь прошёл аутентификацию и система уже знает, кто он `(request.user)`, наступает следующий этап — проверка прав. Это и есть `permissions`.
`Permissions` определяют, имеет ли конкретный пользователь право выполнять конкретное действие. Например:
- можно ли ему редактировать товары;
- разрешено ли ему удалять заказы;
- имеет ли он право видеть список всех пользователей.

### Как это работает

В `DRF` у каждого представления (`APIView`, `ViewSet` и т.д.) можно указать список `permission_classes`. Это список проверок, которые должны пройти до выполнения запроса.

Если хотя бы один `permission`-класс вернёт `False` — запрос не будет выполнен, и пользователь получит ошибку `403 Forbidden`.

**Простой пример**

```python
from rest_framework.permissions import IsAuthenticated

class ProductViewSet(ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]
```

Здесь мы говорим: `"Разрешить доступ только аутентифицированным пользователям"`. Остальные получают `401`.

[](image/REST_authentication_token_error.png)

### Встроенные классы permissions

DRF предоставляет несколько готовых классов:

#### AllowAny

Позволяет доступ вообще всем — даже незалогиненным пользователям. Это поведение по умолчанию, если вы ничего не указали.

```python
permission_classes = [AllowAny]
```

#### IsAuthenticated

Требует, чтобы пользователь прошёл аутентификацию.

```python
permission_classes = [IsAuthenticated]
```

Если пользователь не вошёл — он получит ошибку `401`.

#### IsAdminUser

Только для пользователей, у которых `is_staff=True`.

```python
permission_classes = [IsAdminUser]
```

Обычные зарегистрированные пользователи будут получать ошибку `403`.

#### IsAuthenticatedOrReadOnly

Позволяет читать (GET, HEAD, OPTIONS) — всем,
а изменять (POST, PUT, DELETE) — только аутентифицированным пользователям.

```python
permission_classes = [IsAuthenticatedOrReadOnly]
```

Очень удобно, когда нужно сделать публичное чтение, но защитить изменение данных.

### Как работает проверка

Каждый `permission`-класс должен реализовывать метод:

```python
def has_permission(self, request, view):
    ...
```

Если вы работаете с объектом (например, `retrieve`, `update`, `destroy`), можно также реализовать:

```python
def has_object_permission(self, request, view, obj):
    ...
```

**Пример: Только автор может редактировать объект**

Допустим, у нас есть модель `Product`, и мы хотим разрешить редактировать и удалять только тем пользователям, которые являются администратораами сайта.

Создаём `permission`-класс:

```python
from rest_framework.permissions import BasePermission
class IsAdminOrReadOnly(BasePermission):
    """
    Разрешает доступ только администраторам.
    Остальным пользователям доступ только на чтение.
    """

    def has_permission(self, request, view):
        # Разрешаем доступ на чтение всем
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        # Разрешаем изменение только администраторам
        return request.user and request.user.is_staff
```

Теперь подключаем его к представлению:

```python
from rest_framework.viewsets import ModelViewSet
from .models import Product
from .serializers import ProductSerializer
from .permissions import IsAdminOrReadOnly   

class ProductViewSet(ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAdminOrReadOnly]
```
Теперь, если пользователь не администратор, он сможет только читать данные о товарах, но не сможет их изменять или удалять. Если он попытается сделать POST, PUT или DELETE запрос, получит ошибку `403 Forbidden`.

### Комбинирование permissions

Можно указать несколько классов `permissions`, и они будут проверяться по очереди. Если хотя бы один из них вернёт `False`, доступ будет запрещён.

```python
permission_classes = [IsAuthenticated, IsAdminUser]
```

В этом случае пользователь должен быть аутентифицирован и администратором, чтобы получить доступ.

### Пример с несколькими permissions
Допустим, у нас есть API для управления заказами, и мы хотим:
- Позволить всем пользователям видеть список заказов.
- Разрешить создавать заказы только аутентифицированным пользователям.
- Разрешить редактировать и удалять заказы только администраторам.
Для этого можно использовать `IsAuthenticatedOrReadOnly` для списка заказов и `IsAdminUser` для действий редактирования и удаления.

```python
from rest_framework.viewsets import ModelViewSet
from rest_framework.permissions import IsAuthenticatedOrReadOnly, IsAdminUser
from .models import Order
from .serializers import OrderSerializer

class OrderViewSet(ModelViewSet):
    queryset = Order.objects.all()
    serializer_class = OrderSerializer

    def get_permissions(self):
        if self.action in ['list', 'retrieve']:
            return [IsAuthenticatedOrReadOnly()]
        elif self.action in ['create', 'update', 'partial_update', 'destroy']:
            return [IsAdminUser()]
        return super().get_permissions()
```

В этом примере:
- Для действий `list` и `retrieve` используется `IsAuthenticatedOrReadOnly`, что позволяет всем пользователям видеть заказы.
- Для действий `create`, `update`, `partial_update` и `destroy` используется `IsAdminUser`, что разрешает эти действия только администраторам.

## Фильтрация данных в API

Когда вы создаёте `API`, вы почти всегда работаете с большим количеством объектов — товаров, заказов, пользователей. И клиенту редко нужно всё сразу. Гораздо чаще — *"все товары этой категории"*, *"все заказы пользователя"*, *"все продукты дешевле определенной цены"*.

Вот тут и нужна фильтрация.

### Как это работает

Фильтрация в `Django REST Framework` — это механизм, который позволяет через параметры URL запроса вернуть только нужные записи из базы данных. Например:

```bash
GET /api/products/?category=3&price=100
```

Такой запрос должен вернуть все продукты из категории с `id=3` и с ценой `100`.

### Библиотека django-filter

`DRF` не реализует фильтрацию *"из коробки"*, но позволяет очень просто подключить стороннюю библиотеку `django-filter`, которая идеально подходит для этого.

#### 1. Устанавливаем django-filter

```bash
pip install django-filter
```

#### 2. Добавляем в INSTALLED_APPS

```python
INSTALLED_APPS = [
    ...
    'django_filters',
]
```

#### 3. Настраиваем DRF
В `settings.py` нужно указать, что мы будем использовать фильтрацию:

```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
    ],
}
```

#### 4. Простейший вариант фильтрации — filterset_fields

Если у нас есть модель `Product` со связью к `Category`, мы можем разрешить фильтрацию по определённым полям:

```python
from django_filters.rest_framework import DjangoFilterBackend

class ProductViewSet(ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAdminOrReadOnly]
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ["category", "price"]
```

Теперь можно делать запросы:

```bash
GET /api/products/?category=2
GET /api/products/?price=199
GET /api/products/?category=2&price=199
```

**Фильтрация по ForeignKey**

Если category — это `ForeignKey`, то всё равно фильтрация будет работать по id категории. То есть:

```bash
/api/products/?category=5
```

вернёт все продукты, у которых `product.category_id == 5`.

#### 5. Кастомная фильтрация через `FilterSet`

Если нужно больше гибкости — можно создать отдельный фильтр-класс.

```python
import django_filters
from .models import Product

class ProductFilter(django_filters.FilterSet):
    price_min = django_filters.NumberFilter(field_name="price", lookup_expr='gte')
    price_max = django_filters.NumberFilter(field_name="price", lookup_expr='lte')

    class Meta:
        model = Product
        fields = ['category', 'price_min', 'price_max']
```

Подключаем:

```python
class ProductViewSet(ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_class = ProductFilter
```
Теперь можно фильтровать по цене с помощью параметров `price_min` и `price_max`:

Теперь можно фильтровать так:

```bash
/api/products/?category=2&price_min=100&price_max=300
```

**Когда это особенно полезно**
 - В интернет-магазинах (по цене, бренду, категории)
 - В админках (по дате создания, статусу)
 - В любом списке с большим количеством записей


## Поиск и сортировка в API

После того как мы реализовали фильтрацию, следующим шагом становится удобство для пользователя — возможность быстро найти нужный объект по части текста и отсортировать результаты по дате, цене, имени и т.д.

Оба механизма — поиск и сортировка — встроены в `Django REST Framework`. Они не требуют дополнительных библиотек и легко подключаются через `filter_backends`.

### Поиск

**Поиск** — это способ находить объекты, содержащие определённый текст в одном или нескольких полях. Например: "все товары, где в названии или описании есть слово 'чехол'".

#### Как подключить

В `ProductViewSet` добавляем `backend SearchFilter` и указываем поля, по которым можно искать:

```python
from rest_framework.filters import SearchFilter

class ProductViewSet(ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend, SearchFilter]
    filterset_class = ProductFilter
    search_fields = ['name', 'description']
```
Теперь можно делать запросы с параметром `search`:

```bash
GET /api/products/?search=чехол
```

И получим все продукты, где `name` или `description` содержит *"чехол"* (регистр не важен).

#### Поддержка поиска по связям

Если `Product` связан с `Category`, можно искать по полям из связанных моделей:

```python
search_fields = ['name', 'category__title']
```
Теперь запрос `GET /api/products/?search=электроника` вернёт все продукты, у которых в названии или в названии категории есть слово *"электроника"*.


### Сортировка

Сортировка позволяет вернуть данные в нужном порядке — по цене, дате добавления, названию и т.д.

#### Как подключить

Добавим `OrderingFilter` в список `filter_backends`:

```python
from rest_framework.filters import OrderingFilter

class ProductViewSet(ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_class = ProductFilter
    search_fields = ['name', 'description']
    ordering_fields = ['price', 'created_at']
    ordering = ['price']  # сортировка по умолчанию
```

Теперь можно сортировать результаты по полям `price` и `created_at`. Параметр `ordering` позволяет указать, по какому полю сортировать.

```bash
GET /api/products/?ordering=price
GET /api/products/?ordering=-created_at
```

 - `ordering=price` — сортировка по цене по возрастанию
 - `ordering=-price` — по убыванию
 - `ordering=-created_at` — сначала новые

### Всё вместе

Вот как будет выглядеть полноценный `ProductViewSet` с фильтрацией, поиском и сортировкой:

```python
from rest_framework.viewsets import ModelViewSet
from rest_framework.filters import SearchFilter, OrderingFilter
from django_filters.rest_framework import DjangoFilterBackend
from .models import Product
from .serializers import ProductSerializer
from .filters import ProductFilter

class ProductViewSet(ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_class = ProductFilter
    search_fields = ['name', 'description']
    ordering_fields = ['price', 'created_at']
    ordering = ['price']
```

Теперь у вас:
 - фильтрация: `/api/products/?price_min=1000`
 - поиск: `/api/products/?search=чехол`
 - сортировка: `/api/products/?ordering=-price`