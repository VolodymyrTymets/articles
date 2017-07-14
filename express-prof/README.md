# Express Api професійно

[Express js](http://expressjs.com/) це чудовий фрейморк для розробки Rest Api на мові програмування java script. Він працює під управління [node js](https://nodejs.org/en/), який прекрасно підходить для додаткі обробки потоків вводу/виводу. Або ж  [rest server](https://en.wikipedia.org/wiki/Representational_state_transfer). Про те як почати писати на express js уже написано достатньо статей уроків і туторіалів. В цій статті я б хотів поділитися своїм досвідом у організаці express rest api додатку та відповісти на кілька намтупних запитань.

- Як організувати структуру rest Api?
- Як обробляти помилки та виключення?
- Як організувати rest запити?

## Структура rest api
Розробка rest api це посуті опис шляхів (routs) за якими до вашого сервера буду звертатися клієнти а можливо інші сервіси. Тому їх організація та інтуїтивно зрозумілий опис є надзвичано важливими. Великий rest api серрвер може виконувати безліч операцій.  Велика помилка яку часто роблять розробники це створення окремих роутів на кожну операцію. Це призводить до неконтрольованого росту роутів. Але ж про ці всі роути слід знати (на що здатен сервер) іншим розробникам. Це добре якщо кожний роут задукоментований. А якщо ні це може призвести до великої путанини і великиз затрат часу на розуміння іншими  вашого api.

Щоб уникнути цього та мінімізувати кількість роутів у вашому api старайтеся слідувати логіці rest. У rest є 4 основні види запитів __GET__, __POST__, __UPDAE__, __DELETE__. Старайтеся позиціонувати кожну операцію їз цими запиами. Наприклад якщо вам слід оновти якусь сутність використувуйте запити __UPDATE__ якщо удалити __DELETE__.
Також старайтеся позиціонувати кожеш ваш роут з моделлю бізнес логіки. Нижче наведена таблиця і 5 роутів для обробки одніїє моделі бізнес логіки.

| Route                   |  HTTP Verb | Description               |
|-------------------------|------------|---------------------------|
|`/api/v1/models/`        |__GET__     | Get list of models        |
|`/api/v1/models/:modelId`|__GET__     | Get  model item           |
|`/api/v1/models/`        |__POST__    | Crete model Item          |
|`/api/v1/models/:modelId`| __PUT__    | Update model Item         |
|`/api/v1/models/:modelId`| __DELETE__ | Remove model item         |

У таблиці наведено варіанти роутів та відповідні методи, щоб забезпечити CRUD (create, read, update, delete) над певною моделлю. Дортримуючись даного підходу для кожної моделі і не створюючи додаткових роутів ви значно скоротете ваше Api та покращете його розуміння дл інших розробників.
#### Params
__А як же додаткові параметри?__ Припустим нам необхідно вернути список моделей відсорованих та пофільтрованих. Для цього не варто створювати додатковий роут типу `/api/v1/models/sorted/:param`. Достатньо використати __qery__ параметри. І роут матиме наступний вигляд `/api/v1/models?sort=name`. Таких параметрів можне буте набагато більше. Вони є не обовязковими тому їх використання дозволить вам обробити кілька операцій на одному роуті. Можна подумати що таку стрічку набагато важче формувати на клієнті проте ні. Як це зробити коректно я покажу нижче.
### Code organization
Кілька прорад для розробників по оргацізації коду.
- починайте кожен ваш роут `/api/v1/`
 можливо у майбутньому прийдеться переписати ваше api зберігши функціональність старого на деякий час.
- Розділяйте моделі та контроллери
  Моделі це обєкти бізнес логіки їх слід зберігати у паці models. Контроллери містять бізнес логіку роутів пака controllers.
- Не збивайте роути однієї модделі в один файл.
  створіть окрему папку `modelName` для кожної сутності та для кожної операції GET, POST, DELETE створюйте окремий файл. Наприклад файл `controllers/notifications/index.js` міг мати наступний вигляд.
 ```
const fetchNotifications = require('./fetch-notifications');
const updateNotification = require('./update-notification');
const removeNotification = require('./remove-notification');
/**
* Provide Api for Investors (user with articles)
 **/

module.exports = ({ config, db }) => {
  api.get('/', authenticate, fetchNotifications({ config, db }));
  api.put('/:notificationId',authenticate, updateNotification({ config, db }));
  api.delete('/:notificationId', authenticate, removeNotification({ config, db }));
  return api;
};
```

- Документуйте ваші контроллери.
- Виность додатковий код  у відповідні пакпки 'utils', 'helpers'

#### Documentation
Один з найважливіших елементів розробки rest api є його документація. Оскільки таким api можуть користуватися як інші розробники (front-end, movile developres), так і інші сервіси тому їм необхідно знати як з ним взаємодіяти. Тому дотримуйтеся наступних порад.
- __Ввизначіть для чого це Api і хто ним буде користуватися.__
Від цього залежить конкретний тип документації. Як що це внутрішнє api проекту тоді можна документувати ваші роути як у файлах README в середені вашого репозиторію, так і у самих фйлах `controllers/model/index.js`. Нижче наведено невеликий приклад документації.
```
/**
 Provide Api for Model

  Model list  GET /api/v1/models
  @header
         Authorization: Bearer {token}
  @optionalQueryParameters
         param1 {String} - description
         param2 {String} - description

  Model create  POST /api/v1/models
  @header
         Authorization: Bearer {token}
  @params
         param1 {string}
         param2 {boolean}
**/
```
Якщо ж це api з яким будуть взаємодіяти інші сервіси, слід задуматися над створення повної, відсортованої та зручної у користуванні. Ось один приклад чудової документації [google calendar api](https://developers.google.com/google-apps/calendar/v3/reference/).

- __Документуйте кожен свій rout.__
  Який смисл в існуванні роута якщо про нього не знають інші. Якщо користувач вашого api не знатиме про існування того чи іншого роута він просто вирішить що у вас не має такого функціоналу. Тому документуй кожен свій роут.
- __Документуйте роут у момент його створення__
 Не відкладайте на потім, щоб не забути про нього чи його параметри.


## Обробка помилок
У rest серрвера є набір загальноприйнятих [помилок](http://www.restpatterns.org/HTTP_Status_Codes), тому не варто вигадувати свої. Краще один раз написати свій обробник подій і використовувати його ніж кожному роуті займатися цим. У цьому може допомогти пакет [rest-api-errors](https://www.npmjs.com/package/rest-api-errors). Спершу слід написати загалтний обробник:

```
const { APIError, InternalServerError, Unauthorized } = require('rest-api-errors');
const { STATUS_CODES } = require('http');
const winston = require('winston');
module.exports = (err, req, res, next) => {
  let error = err instanceof APIError ? err : new InternalServerError();

  if(process.env.NODE_ENV !== 'production') {
    winston.log('error', '-----> Unknown server error...');
    winston.log('error', err);
  }
  res
    .status(error.status || 500)
    .json({
      code: error.code || 500,
      message: error.message || STATUS_CODES[error.status],
    });
};
```

та використати його в exoress
```
const errorHandler = require('./utils/error-handler');
const app = express();
app.use(errorHandler);
```

Тепер у любому контроллері можна просто ініційовувати помилку
```
const { MethodNotAllowed } = require('rest-api-errors');

api.use('/account', (req, res, next) => {
....
if(!user) {
  throw new Unauthorized(401, 'Unauthorized');
 }
});

```

### Permission checker
Не рідко виникає необхідність обмежувати користувачів у певних діях. Наприклад менеджери можуть мати набагато більше прав ніж прості користувачі. Далі показано як написати пройти та компачтний permission checker ось такого вигляду.

```
 import { hasPermissionTo, actions } from '../'
 api.use('/account', (req, res, next) => {
     const user = getUser();
     if (!hasPermissionTo(actions.GET_USER, user)) {
        throw new MethodNotAllowed();
     }
});
```

> де взяти обєкт user це вже залежить від типу авторизації яку ви використали детальніше [тут](http://passportjs.org/docs).

Спершу опишем усі доступні дії (їх можна буде доповнювати в майбутньому). `utils/permission-checker/constants.js`
```
module.exports = {
  ALL_RIGHT: 'ALL_RIGHT',
  GET_USER: 'GET_USER',
};
```
Далі присвоїм ці дії конкретним користувачам `utils/permission-checker/permissions.js`.
```
const actions = require('./constants');
module.exports = {
  owner: [
    actions.GET_USER,
  ],
  user: [],
  admin: [
    actions.ALL_RIGHT,
  ],
};
```
Далі створимо сам метод hasPermissionTo `utils/permission-checker/index.js`
```
const _ = require('lodash');
const permissions = require('./permissions');
const actions = require('./constants');

const hasPermissionTo = (action, user) => {
  const isRoleHasPermission = action => role => _.includes(permissions[role], action);
  // get map user rolse into true false array
  const userPermissions = user.roles.map(isRoleHasPermission(action));

  if (!(userPermissions && _.includes(userPermissions, true))) {
    return false
  }

  return true;
};

module.exports = {
  hasPermissionTo,
  actions,
};
```
> В даному прикладі користувач має поле roles ['owner', 'user'], яке вказує до якої ролі він належить. І коли користувач входить до ролі owner яка має дозвіл GET_USER тоді hasPermissionTo верне true. Ви можете описати свою логіку метода hasPermissionTo.


## Запити на клієнті
Запити з клієнта можна оргпнізувати різними способами і за домомогою різних бібліотек. Але б хотілося дати кілька порад по органцізації цього процесу.

- використовуйте розроблені бібіліотеки для цього такі як [axios](https://www.npmjs.com/package/axios)
- зберігайте роути відділено від коду
  Велика помилка прописувати шлях запиту в самому коді наприклад.
```
axios.get('api/v1/users', {
    params: {}
  })
```
Проблема такого підходу в тому що коли шось поміняється в Api буде тяжко шукати усі ці шляхи у коді. Краже написати певне сховиша для всіх шляхів і зберігати їх в одному місці `utils/api/urls.js`
```
export default {
  USERS: '/api/v1/amazonS3/',
};
```
Тереп можна використати цю константу одразу для 4 запитів
```
import ApiAdressses = from './utils/api/urls.js';
axios.get(ApiAdressses.USERS);
axios.post(ApiAdressses.USERS);
axios.put(ApiAdressses.USERS);
axios.delete(ApiAdressses.USERS);
```
- __напишіть допоміжний функціонал для взаємоді з api__

Зазвичай більшість запитів крім даних містьть заголовки для авторизації, абож  qury параметри. І прописувати все це у вашому коді кожен раз не найкраща ідея.
```
axios({
      url: `ApiAdressses.USERS?limit=${limit};soty${sort}`,
      method: 'GET',
      headers: {
        'Authorization': 'Bearer token'
     }
    });

//    or

axios({
      url: `ApiAdressses.USERS`,
      method: 'POST',
      headers: {
        'Authorization': 'Bearer token'
        'Content-Type' : 'multipart/form-data'
     }
    });
```

Краще написати чотири методи використання яких виглядатиме наступним чином
```
import { get, post, put, remove, query, APIAddresses } from './utils/api';

const response = await get(`APIAddresses.USERS${query({ limit, sort })}`);
const response = await post(`APIAddresses.USERS`, formData);
const response = await put(`APIAddresses.USERS/${userId}`, formData);
const response = await remove(`APIAddresses.USERS/${userId}`);
```

Методи get, post, put, remove обгортки одного метода `makeRequest` створенне ні для зручності використання. Метод query формує стрічку query з вжідних параметрів.
```
import axios from 'axios';
import APIAddresses from './urls';

const makeRequest = async (type, url, data) => {
  try {
  const response =  await axios({
      url,
      data,
      method: type,
      headers:
        'Authorization': getTokenHeaderValue(),
        'Content-Type' : 'multipart/form-data'
      }
    });
    return response;
  } catch (error) {
    if(process.env.NODE_ENV !== 'production') {
      console.log(`Error: [${type}] - ${url}`);
      console.log(error);
    }
    return error;
  }
};

const get = url =>  makeRequest('get', url, null);
const post = (url, data) => makeRequest('post', url, data);
const put = (url, data) =>  makeRequest('put', url, data);
const remove = url => makeRequest('delete', url, null);
const query = data => `?${Object.keys(data).map(key => `${key}=${data[key]}`).join('&')}`;

export { APIAddresses, get, post, put, remove, query, postFile };

```
> APIAddresses імпортований і експортований для зручності використання що імпортувати все з одного джерела

Організувавши логіку запитів таким чином ви позбудетеся лишнього коду та покращете розуміння вашого коду для інших розробників. Також при зміні чи переході на нову версію api вам достатньо змінити шляхи  в одному файлі.

# Підсумок

Кожен проект може бути різним і у кожного свої потреби. Не завжди полуситться в ідеальному дотримуватися якигось шаблонів, проте старайтеся дотримуватися порад поданій у цій статті.

- Дотримутесф логіки rest
- Організуйте ваше api (роути, обробку подійб права доступу)
- Докумнтуйте ваше api
- Організуйте ваші запити для уникнення дублюваннякоду.

Організація вашоко rest api скорить час розробки, покращить розуміння іншими розробниками того що ви робити та значно спростить супровід.
