# Api
Більшість свого часу програмісти прцюють із кодом проектів які зачастую розроблялися кимось до них або ж він створювався достатньо давно.  Тому часто коли приходиться пчинати проект з нуля часто виникає ряд питаня. З чого почати? Як це все заставити працювати? Чому я це повининен робити? 
В даній статті я хотів поділитися тим як в кілька кроків настроїти boilerplate для API невеличкого forum-like application based on express

## API for the forum-like application. Where should I begin?
> This part refers to the preparation processes and where a developer should start when building a forum-like app.

Перш за все слід обдумати функціональні вимоги поставленні перед вашим додатком. В залежності від них структура може мінятися. Наприклад чи потрібний у вашому додатку механізм авторизації чинію. Чи буде шттеграція із іншими сервісами чи ні. Всі інші додаткові функціональні вимоги типу генерації звітів (pdf, xml, csv) відправка емейлів тощо.

- функціональні вимоги вашого додатку
- дані якими він оперуватими 
- методи розробки
## Can boilerplate be perfect?  
> We plan to unveil here code organization and folders structuring best practices. Tips (based on personal experience) with examples which others might find useful.

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
 
Дана структура дозволить вам зберігати ваш код в логічному порядку і дозволить швидко зорієнтуватися. своєрідним клеєм який дозволить звязати усі файли виступатими файл `app.js`

```
const express = require('express');

const config = require('./config');
const api = require('./src/api/index');
const { mongoManager } = require('./src/mongo');

const app = express();
mongoManager.connect();

// api routes v1
app.use('/api/v1', api(config));

module.exports = app;
```
Тепер для запуску вашого додатку достаттно інстаювати пакети `npm i` запустити `node ./bin`. Або дописати дану команту в npm script. 

## Dealing with data
> We're planning to show here the example of how you can get access to the data, using MongoDB.

__Крок 2__ Оголошення моделей
Кожне апі зазвичай працює з якимись даними. І наше майбутнє апі не авиключення. Нижче наведена приблизна структура бази даних для  forum-like application.

![](https://github.com/VolodymyrTymets/articles/blob/express-boilerplate/express-boilerplate/img/fig2.png?raw=true)

Відповідно до цієї структури ми оголошуємо наші моделі. Оскільки для зберігання даних ми використовуємо [mongo](https://www.mongodb.com/) базу даних то побудуємо наші моделі за допомогою [mongoose](http://mongoosejs.com/docs/populate.html).

Деректорія models виглядатиме наступним чином.

![](https://github.com/VolodymyrTymets/articles/blob/express-boilerplate/express-boilerplate/img/fig3.png?raw=true)

- `schema.js` - це схема моделі детальніше [тут](http://mongoosejs.com/docs/schematypes.html)
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
За аналогічною схнмою оголошуємо усі інші наші моделі та схеми для них.  
## What about routes? 
> Routes structuring. Organization, documentation, and description tips and tricks.

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

Таким чином, по мірі розростання проекту, ви реєструєте нові моделі та контроллери в обному місці. А даний файл виконується в `app.js` як показано в на кроці 1.

## Additional recommendations you can't omit
__Крок 5__ Додатковий функціонал
 - how to handle errors/bugs
 - how to implement additional features for the app.

## Sumarry