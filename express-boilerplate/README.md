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


3. Dealing with data
We're planning to show here the example of how you can get access to the data, using MongoDB.
4. What about routes? 
Routes structuring. Organization, documentation, and description tips and tricks.
5. Additional recommendations you can't omit
 - how to handle errors/bugs
 - routes and data together. How to not get lost
 - how to implement additional features for the app.