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
`apiko-image-cropper.htms` - цей [фал](https://github.com/VolodymyrTymets/node-red-react-example/blob/master/pallets/apiko-image-cropper/apiko-image-cropper.html) node red використовуватиме для реєстрації UI частини майбутнього палету. Із ним дещо складніше адже тут слід зареєструвати кілька налаштуваня для палету. Весь їх список перелічений [here](https://nodered.org/docs/creating-nodes/node-html). Самі важливі із них наведені нижче:
![](https://github.com/VolodymyrTymets/articles/blob/master/node-red/img/fig3.png?raw=true)

- 1 - категорія палету (можна використовувати як існуючі так і пвказувати власі) таким чином ми групуємо палети за приналежністю до чогось
- 2 - Колір палету
- 3 - Іконка палету
- 4 - label палету (користувача в майбутньому зможе його міняти а по дефолту вказуєм назву тут)
- 5 - Список полів які отримуватиме ваша серверна частина в `msg.payloads`. Якщо нижче у пункті 7 буде `<input type="text" id="node-input-name"` node red знатиме що значення цього поля слід передати на сервер. Це дежу важливий нюанс детальніше можна почитати [here](https://nodered.org/docs/creating-nodes/properties)
- 6 - зазначаємо скільки точок зєднання з іншими палетами на вході і на виході.
- 7 - Контент який відображатиместь в UI
> у пункті 7 міститься простий `<div >` його ми використаємо для react в майбутньому поки можна просто додати `<p>hellow world</p>`

### Реєстрація палету в node red
Тепер коли ми створили наш перший палет можна додати його в node red і запустити його. Перш за все слід [встановити node red](https://nodered.org/docs/getting-started/). Далі слід підключити ваш [пакет локально](https://nodered.org/docs/getting-started/adding-nodes).
```
cd $HOME/.node-red
npm install <npm-package-name>
```
> node red зберігає всі свої файли в .node-red дерикторії. Ви можете переглянути .node-red/pacakge.json і моніторити усі пакети (палети) які ви підключаєте

І тепер все що нам залишилося це запустити `node-red` і перевірити чи усе працює.
### Node Red Server Side more professional
Однією з речей що мені не сподобалася в node-red це те що вся обробка інформації від попередніх палетів відбувається в одному .js файлі а конкретно в методі `node.on('input',`. Якщо це кілька строк коду то це нормально. проте якщо вам прийдеться писати якусь складну логіку зберігати це все діло в одному файлі не бажано. На щастя усе це запускається під node js і можна перенесту нашу логіку в кись обробник.

Отже в дерикторії `src/server/pallet-managers` зберігатиметься логіка обробки сереверного коду наших палетів. Отже створимо клас `ImageCropperPalletManager` для нашого палету `apiko-image-cropper` який імплементитеме один обовязковий метод `onInput`. Усе решта це ваша додаткова логіка.
```
// /src/server/pallet-managers/image-cropper/ImageCropperPalletManager.js
class ImageCropperPalletManager {
  constructor(RED, palletConfig, node) {
    this._self.palletType = 'apiko-image-cropper'; 
     this._self.palletType = palletConfig.someProps...// pass payload for this
    this.onInput = this.onInput.bind(this._self);
  }
  onInput(msg) {
    try {
      const { palletType, someProps } = this; // get payload data
      this.send(msg);
    } catch (error) {
     // catch error here
    }
  }
}

module.exports = { ImageCropperPalletManager };
```
Тепер просто зареєструєм наш клас у  `apiko-image-cropper.js`.
```
const { ImageCropperPalletManager } = require('../../src/server/pallet-managers/image-cropper');

module.exports = function(RED) {
  'use strict';
  function nodeGo(config) {
    RED.nodes.createNode(this, config);

    const node = this;
    const palletManager = new ImageCropperPalletManager(RED, config, node);
    // register onInput callback here
    node.on('input', palletManager.onInput);
  }
  ...
};
```
>  Повний код даного класу [here](https://github.com/VolodymyrTymets/node-red-react-example/blob/master/src/server/pallet-managers/image-cropper/ImageCropperPalletManager.js) дещо відрізняється від наведеного вище але це всього лиш нюанси реалізації.

Тепер можна працювати як із любим іншим node проектом. Ми відділили логіку node red від бізнес логіки.
Ще один нюанс на який би хотілося звернути увагу це [стату палету](https://nodered.org/docs/creating-nodes/status) під час виконання яким можна управляти. Досить викликати `this.status({ fill:"red", shape:"dot", text: "some error"});` щоб показати помилка в UI під час виконання. Це досить коорисно коли слід показати якись процес який займає певний час чи для відображення якигось помилок виконання.

### Add React for UI
#### Create our Component
#### Build React
#### Add Build react for node red
#### Add reat time update

# conclusion

