# Api
Більшість свого часу програмісти прцюють із кодом проектів які зачастую розроблялися кимось до них або ж він створювався достатньо давно.  Тому часто коли приходиться пчинати проект з нуля часто виникає ряд питаня. З чого почати? Як це все заставити працювати? Чому я це повининен робити? 
В даній статті я хотів поділитися тим як в кілька кроків настроїти boilerplate для API невеличкого forum-like application based on express

## API for the forum-like application. Where should I begin?
Перш за все слід обдумати функціональні вимоги поставленні перед вашим додатком. В залежності від них структура може мінятися. Наприклад чи потрібний у вашому додатку механізм авторизації чинію. Чи буде шттеграція із іншими сервісами чи ні. Всі інші додаткові функціональні вимоги типу генерації звітів (pdf, xml, csv) відправка емейлів тощо.

- функціональні вимоги вашого додатку
- дані якими він оперуватими 
- методи розробки

## Can boilerplate be perfect?  
Перш за все потрібно розробити структуру вашого boilerplate. Це можна робити вручну створюючи кожен файл. Можна використати утиліти який в інтерненті є придестатньо та змінити його для сфоїх потреб. Ми підем другим шляхом і використаємо утиліту [express-generator](http://expressjs.com/uk/starter/generator.html). Отже приступимо до розробки boilerplate for API of forum-like application. 
__Крок 1__ Генерація структури проекту
Установіть express-generator та створіть дерикторію вашого проекту.
```
npm install express-generator -g
express <your-project-name>
```
Оскільки express-generator генерує структуру яка не зовсім задовільняє наші потреби її слід дещо змінити. Нище показано наглядний приклад того які зміни слід зробити. 
![](https://github.com/VolodymyrTymets/articles/blob/express-boilerplate/express-boilerplate/img/fig1.png?raw=true)

Злівої сторони наведено boilerplate зшгенерований а з правої той який слід зробити. Давайте розглянемо структуру проекту детальніше
 - `controllers` - тут слід зберігати всі ваші Api endpoints
 - `models` - тут слід зберігати всі моделі даних 
 - `api` -  тут знаходиться взаємодія ваших даних з вашими Api endpoint (детальніше нижче) 
 - `utils` -  тут знаходиться весь допоміжний код додатку (відправка емейлів генерування pdf ets)
 -  `middleware` -  тут знаходиться всі express middleware додатку
 -  `mongo or db or <yourDatabaseName>` -  тут знаходиться уся робота з вашою бащою даних
 -  `config or .env` -  усі налаштування вашого додатку краще зберігати в окремому файлі (це може бути як .js .json or .env)
 
Дана структура дозволить вам зберігати ваш код в логічному порядку і дозволить швидко зорієнтуватися.
Що ж далі? З чого слід почати перш за все? Оскільки оснавна задача апі оперувати даними то з них слід і почати. Багато хто недооцінює важливість важливість правильної організації даних та починають розробку одразу ж. І даремно. Неправильна структура ваших даних (ващої бази даних) може значно ускладнити вам життя в подальшому. Тому краще потратити більше часу і розробити хорошу структуру даиних ніж потому мати зайві проблеми.

## Dealing with data
__Крок 2__ Оголошення моделей
Кожне апі зазвичай працює з якимись даними. І наше майбутнє апі не авиключення.  Ми почнемо розробку апі з оголошення структури даних. У нашому випадку це робота із схемами та моделями.
Щоб краще розуміти і осмислювати дані якими вам слід оперувати краще мати їх структуру і взаємодії у вигляді певної графічної схеми. Так ви краще будете осмислювати що до чого. Нижче наведена приблизна структура бази даних для  forum-like application.

![](https://github.com/VolodymyrTymets/articles/blob/express-boilerplate/express-boilerplate/img/fig2.png?raw=true)

Відповідно до цієї структури ми оголошуємо наші моделі. Оскільки для зберігання даних ми використовуємо [mongo](https://www.mongodb.com/) базу даних то побудуємо наші моделі за допомогою [mongoose](http://mongoosejs.com/docs/populate.html).

Деректорія models виглядатиме наступним чином.

![](https://github.com/VolodymyrTymets/articles/blob/express-boilerplate/express-boilerplate/img/fig3.png?raw=true)

- `schema.js` - це схема моделі детальніше [тут](http://mongoosejs.com/docs/schematypes.html)
Приблизна реалізація схеми наведена нище. Все досить просто. Ви оголошуєте поля та типи ваших моделей. Також можна додавати додаткові перевірки і валідації но це вже при необхідності можна поситати в документації. 
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const ObjectId = Schema.Types.ObjectId;

const schema = new Schema({
  title: {
    type: String,
    required: [true],
  },
  description: {
    type: String,
    required: [true],
  },
  questionId: {
    type: ObjectId,
    ref: 'Question',
  },
  createdAt: {
    type: Date,
    required: [true],
  },
  createdBy: {
    type: ObjectId,
  }
});

module.exports = { schema };
```

Оголосивши схему ми її просто експортуємо для того щоб використати в 'model.js'.

- `model.js` - це саме обєднання mongoose mмоделі та схеми. Це зроблено в окремих файлах тому що mongoose надає можливість розширяти ваші моделі різними методами і хуками. Тримати усе це + схема, яка сама по собі здатна розростатися чимало, призведе до того що ваш файл займатиме 500+ строчок. А це не дуже добре.
```
// models/answers/model.js
const mongoose = require('mongoose');
const { schema } = require('./schema');
// add hooks here
schema.pre('save', function() {
  return doStuff().
    then(() => doMoreStuff());
});
const Answer = mongoose.model('Answer', schema);
module.exports = { Answer };
```
Як можна було замітити тут оголошено hook `pre('save')`. Даний код буде викликатися при кожному збереженні вашої моделі. Це  [Middleware](https://mongoosejs.com/docs/middleware.html) їх використовують якщо потрібно виконати якість складі операції (відправка листа складана перевірка ш так далі). Проте зауважте що це не зовсім для валідації моделі для цього краще аикористовувати [Built-in Validators](https://mongoosejs.com/docs/validation.html#built-in-validators). 
За аналогічною схнмою оголошуємо усі інші наші моделі та схеми для них.  

## What about routes?
__Крок 3__ Оголошення контроллерів
Тепер перейдем до роутів або контроллерів. Контроллери також слід поділити у відповідності до ваших моделей так як зображено нижче.

![](https://github.com/VolodymyrTymets/articles/blob/express-boilerplate/express-boilerplate/img/fig4.png?raw=true)

Таким чином у деректорії `controllers` кожна підерикторія представлятиме певну модель. Кожен файл відповідно представлятиме single Api endpoint.
![](https://github.com/VolodymyrTymets/articles/blob/express-boilerplate/express-boilerplate/img/fig5.png?raw=true)

Такий підхід дозволить максимально розділити код вашого апі і уникнути безглуздого зберігання усіх Api endpoints в одному файлі. Ось як виглядатиме приклад реалізації  `GET /questions` та `GET /questions:_id` Api endpoints.
![](https://github.com/VolodymyrTymets/articles/blob/express-boilerplate/express-boilerplate/img/fig6.png?raw=true)

Як можна було замітити у `./list.js` першим параметром приходить `Question`. Це mongooose модель з якою тепер можна просто працбювати. Також можна даний обєкт містить всі моделі у нашому апі. Таким чином  можна зручно використовувати любу модель в любому контроллері не імпортуючи її. Завдяки цьому код також виглядає компактніші і дозволяє уникнути помилок при імпортах. Як усе це реалізувати показано у наступному розділі.  

## Routes and data together. How to not get lost
__Крок 4__ Реєстрація моделей та контроллерів
Ми розглянули як створити моделі та контроллери у нашому апі. також було показано як використовувати моделі в середені контроллера тепер слід розглянути як повязати моделі і контроллери разом. Усе відбувається у файлі `.src/api/index.js`.
![](https://github.com/VolodymyrTymets/articles/blob/express-boilerplate/express-boilerplate/img/fig7.png?raw=true)

Тут відбувається три основні речі.
1. Імпорт та реєстрація контроллерів.
2. Імпорт комбінування моделей та передача їх у контроллери.
3. Реєстрація обробника помилок, який перехвачуватиме помилки з усіх контроллерів (його реалізація показана в наступному розділі)

Таким чином, по мірі розростання проекту, ви реєструєте нові моделі та контроллери в обному місці. 

Останнім крокоам буде створення точки входу у наше апі. Ним виступатиме  `app.js`. Своєрідний клеєм який дозволить звязати усі файли описувати додатковий функціонал та реєструвати стастичні файли при неоюхідності. Його реалізація наведенна нище. 

```
const express = require('express');

const config = require('./config');
const api = require('./src/api');
const { mongoManager } = require('./src/mongo');

const app = express();
mongoManager.connect();

// api routes v1
app.use('/api/v1', api(config));

module.exports = app;
```
Отже у нас тут відбуваються три основні речі
- імпорт configs `const config = require('./config');`
- підключення до нашої бази даниї  `mongoManager.connect();`
- реєстрація усіх контролерів в  express `app.use('/api/v1', api(config));`. 

Тепер express перенаправлятиме всі запити по шляху `/api/v1` у фаул `api.js` (крок 4) а він у свою чергу перенаправлятиме на controllers (крок 3) а контроллери використовуватимуть моделі (крок 2). Ось так ми звязала усі наші розкидані файли в одну робочу програму. Тепер залишається тільки розширяти наше апі слідуючи даним правилам. Для запуску вашого додатку достаттно інстаювати пакети `npm i` запустити `node ./bin`. Або дописати дану команту в npm script. 

## Additional recommendations you can't omit
__Крок 5__ Додатковий функціонал
 - how to handle errors/bugs
Обробляти усі помилки краще в одному місці вашого додатку. Оскільки це зручно коли вам аинекне необхідність щось змінити (додати логер, обробляти dev/profuction mode ets.). В нашому випадку таким місце являється './midlleware/error-handler.js'.
```
const { APIError, InternalServerError } = require('rest-api-errors');
const { STATUS_CODES } = require('http');

const errorHandler = (err, req, res, next) => {
  const error = (err.status === 401 ||
    err instanceof APIError) ? err : new InternalServerError();

  if (process.env.NODE_ENV !== 'production') { 
    // do something here
  } 
  if (['ValidationError', 'UserExistsError'].includes(err.name)) {
    // if it special error
    return res.status(405).json(err);
  }
  // log error if needed 
  // logger.info('API error', { error: err });

  return res // return 500 for user
    .status(error.status || 500)
    .json({
      code: error.code || 500,
      message: error.message || STATUS_CODES[error.status],
    });
};

module.exports = { errorHandler };
```
Оскільки ми зареєстрували даний обробник у `.src/api/index.js`  тепер достатньо викликати  `next(new Error())` у контроллері і дана помилка попада в наш обробник.

 - how to implement additional features for the app.
 
Ще одне завдання яке стоїть перед розробником забезпечити максимально зручний запуск вашого проетку. Також було б добре мати так званий `hot-reload` вашого додатку. У  node js це можна реалізувати за допомогою [nodemon](https://nodemon.io/). 
Ви можете написати простий npm script `nodemon --watch src --exec 'npm run start` чи sh script. Проте інколи виникає потреба запускати кілка процесів одночасно. Наприклад 1. запуск бази даних (mongo) 2. запуск api 3. запуск UI (react-script). Для таких цілей краще підходить [gulp](https://gulpjs.com/). Нижче наведений невеличкий приклад gulp файлу для запуску монго апі та react-script як єдиного проекту однією командою.
![](https://github.com/VolodymyrTymets/articles/blob/express-boilerplate/express-boilerplate/img/fig7.png?raw=true)

Тепер достатньо створити npm script dev ` "./node_modules/.bin/gulp run:dev",` та просто запустити ваш проект `npm run dev`
- advices
  1. Завжди документуйте ваше апі 
 Оскільки ваши апі користуватимуться інші розробники тому детальна документація кожного api point є життєво необхідною. Вона також убереже вас від багатьох питань типу 'А як мені дістади спимок ...', 'Яким чином мені змінити ...'  
    2. Документуйте усі ваші `utils`
  Оскільки ці речі використовуватимуться у всьому проекту і всіма розробниками добре мати невелиский опис того що робить ваша утиліта і невеликий приклад її використання
    3. Тримайте Readme в належному стані
 Тут варто описати якісь основні речі типу як запустити проект його структуру процес розробки і так далі для інших розробників. 

## Sumarry
Отже в данній статті я коротко показав як можна з нуля створити boilerplate for forum-like application based on express. Вона є дещо загальною і підхоить як основа для любого express application. Проте вона не строгим правилом а лиш рекомендаціями для покращення структури вашого апі. Звісно кожен проект вимагатиме певних змін в залежності від його специфіки і це нормально.  