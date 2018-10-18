## Node-red how start to build your own pallets with dinamic ui with React.
Не так давно я мав можливість ознайомитися із доволі новою технологією розробки програмних продуктів на Node js - Node Red. Якщо коротко то це технологія яка дозволяє зьирати вашу прогруму із готових частин  так званних pallets як конструктор. Із їхнього сайту. 
> Node-RED is a programming tool for wiring together hardware devices, APIs and online services in new and interesting ways.
> It provides a browser-based editor that makes it easy to wire together flows using the wide range of nodes in the palette that can be deployed to its runtime in a single-click.

Це як дитина складає із маленьких конструкторів лего робота чи що лего сіті так ви скаладаєте вашу програму. Перевагою тако підходу являється те що навіть людина без серйьозних навиків програмування може сісти і скласти свою програму.

Цим це корисно для біснесу? Це корисно якщо вам необхідно надти вашим клієнтам можливіст динамічно програмувати певно поведінку на певних частин вашої програми. Допустимо ваш сервіс оперує звітністю і вам необхідно надати можливість різним компаніям вашого сервісу самим задавати порядок обробки цих звітів. Одна компанії хоче відпрвляти листи інша отримувати локальні сповіщення це інша зберігати чи видаляти ці зівти і так далі. В таких випадках ви можете надати зручний інтервейс для step by steps actions.
Програмування в node red це складання pallets step by step. Він надає велику кількість цих палеті. Також бібліотека палетів постійно поповнюється іншими розробниками. Проте не всі випадки чи задачі мож на покрити ними. Особливо якщо це якась спецефічна задача для вашого бізнесу. У такхи випадках вам прийдеться писати свої [палети](https://nodered.org/docs/creating-nodes/first-node). В цілому програмування під node-red зводиться до розробки палетів. Саме про такий підхід у розробці я хотів поділитися з вами у даній статті:
    - як розробити N кількість палетів і в одному проекті
    - як підключити та настроїти ваш проект до Node red
    - як організувати вашу серверну та UI частину
    - ?
##  Постановка бізнес проблеми
Під час написання ціє ї статті я уявив собі наступну ситуацію. Користувачам певного сервісу слід розсилати каталог товарів на наступний тиждень. Каталог містить опис товару і саме головне його картинку. Тому нас слід розробити функціональ, який дозволить переглянути актуальний каталог товарів вибрати певний і сказати нашому сервісу відправляти саме цей товар. 
Для простоти давайте зосередимося тільки на роботі із зображеннями. Спростимо нашу задачу до трьох кроків: 
1. Вибір зображення із каталогу
2. Редагування зображення переп відправкою
3. Розсилка картинок.
![](https://github.com/VolodymyrTymets/articles/blob/master/node-red/img/moc.png?raw=true)

>> ??

## Вирішення

>> ?
### Настройка React Boilerplate wiht Node js
Оскільки нові палети в node red [підключаються](https://nodered.org/docs/creating-nodes/first-node) як npm пакети є два вибори. Реєструвати кожен палет як окремий пакет або ж реєструвати набір палетів як один пакет. Це залежить від того розробляєте ви комплекс із кількох палетів чи один окремий. Другий випадо на мою думку більш типовий і я вирішив взяти за основу його. Тому розробив приблизну структруру проекту:

```
|_ pallets <- place whree all node-red pallets saved
    |_ <categoryName>-<palletName> <- pallet folder
        |_ icons <- icon of pallet
        |_ ui 
           |_ index.js <- register component for pallets
           |_ build/main.js <- builded pallet teplate code
        |_ <categoryName>-<palletName>.html
        |_ <categoryName>-<palletName>.js 
|_ src
   |_ client/containers <- react components here (paller ui templates)
   |_ server <- all server code here
     |_ pallete-managers <- all code to manage server side of pallet
        |_ <palletName> 
   |_ utils <- code to use on client and server
|_ test
   |_ unit
```
> Повний код проекту доступний [тут](https://github.com/VolodymyrTymets/node-red-react-example)

Отже тепер можна приступати до написання нашого проекту. Будемо слідувати крокок за кроком і детально роглядати кожен з них.


### Створення першого mode red pallet
Перш за все потрібно ініціалізувати ваш репозиторі `npm init`. Далі необхідно оприділитися із назвами палетів. Оскільки ваші палети можуть використуваватися із іншими не варто називати їх просто `Cropper`. Краще придумати неймспейс яки обєднюватиме ваші палете. В мому випадку `Apiko`. Тепер наш палет називатиметься `apiko-image-cropper`.
Згідно [туторіал](https://nodered.org/docs/creating-nodes/first-node) кожен палет повинен містити: 
- `package.json`
- `lower-case.js`
- `lower-case.html`

`package.json` - у нас уже є тепер лишилося створити 
- `pallets/apiko-image-cropper/apiko-image-cropper.js` <- server side code
- `pallets/apiko-image-cropper/apiko-image-cropper.html` <- Ui or client side code

Після чого зареєструємо на палет для node-red в `package.json`
```
// package.json
{
    "name" : "node-red-contrib-react-example",
    ...
    "node-red" : {
        "nodes": {
            "apiko-image-cropper":"/pallets/apiko-image-cropper/apiko-image-cropper.js",
        }
    }
}
```
`apiko-image-cropper.js` - цей фал node red використовуватиме для реєстрації вашого палету та для обробки серверної поведінки. Тут нас чікавить тіки `node.on('input', () => ...` callback який викликатиметься тоді коли нам прийдуть дані із попередніх палетів. Детальніше [тут](https://nodered.org/docs/creating-nodes/node-js). 
```
// /pallets/apiko-image-cropper/apiko-image-cropper.js
const  MODULE_NAME = 'apiko-image-cropper';

module.exports = function(RED) {
  'use strict';
  function nodeGo(config) {
    RED.nodes.createNode(this, config);
    const node = this;
    node.on('input', (msg) => {
        msg.payload = { newData: {  } }; // extend payload for next pallet
        this.send(msg)
    });
  }

  RED.nodes.registerType(MODULE_NAME, nodeGo);
};
```
### Реєстрація палету в node red

### Node Red Server Side more professional
### Add React for UI
#### Create our Component
#### Build React
#### Add Build react for node red
#### Add reat time update

# conclusion

