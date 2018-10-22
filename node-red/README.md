# Node-red how start to build your own pallets with dinamic ui with React.
Не так давно я мав можливість ознайомитися із доволі новою технологією розробки програмних продуктів на Node js - Node Red. Якщо коротко то це технологія яка дозволяє зьирати вашу прогруму із готових частин  так званних pallets як конструктор. Із їхнього сайту. 
> Node-RED is a programming tool for wiring together hardware devices, APIs and online services in new and interesting ways.
> It provides a browser-based editor that makes it easy to wire together flows using the wide range of nodes in the palette that can be deployed to its runtime in a single-click.

Це як дитина складає із маленьких конструкторів лего робота чи що лего сіті так ви скаладаєте вашу програму. Перевагою тако підходу являється те що навіть людина без серйьозних навиків програмування може сісти і скласти свою програму.

Цим це корисно для біснесу? Це корисно якщо вам необхідно надти вашим клієнтам можливіст динамічно програмувати певно поведінку на певних частин вашої програми. Допустимо ваш сервіс оперує звітністю і вам необхідно надати можливість різним компаніям вашого сервісу самим задавати порядок обробки цих звітів. Одна компанії хоче відпрвляти листи інша отримувати локальні сповіщення це інша зберігати чи видаляти ці зівти і так далі. В таких випадках ви можете надати зручний інтервейс для step by steps actions.
Програмування в node red це складання pallets step by step. Він надає велику кількість цих палеті. Також бібліотека палетів постійно поповнюється іншими розробниками. Проте не всі випадки чи задачі мож на покрити ними. Особливо якщо це якась спецефічна задача для вашого бізнесу. У такхи випадках вам прийдеться писати свої [палети](https://nodered.org/docs/creating-nodes/first-node). В цілому програмування під node-red зводиться до розробки палетів. Саме про такий підхід у розробці я хотів поділитися з вами у даній статті:
    - як розробити N кількість палетів і в одному проекті
    - як підключити та настроїти ваш проект до Node red
    - як організувати вашу серверну та UI частину
    - як використовувати React with node red
##  Постановка бізнес проблеми
Під час написання ціє ї статті я уявив собі наступну ситуацію. Користувачам певного сервісу слід розсилати каталог товарів на наступний тиждень. Каталог містить опис товару і саме головне його картинку. Тому нас слід розробити функціональ, який дозволить переглянути актуальний каталог товарів вибрати певний і сказати нашому сервісу відправляти саме цей товар. 
Для простоти давайте зосередимося тільки на роботі із зображеннями. Спростимо нашу задачу до трьох кроків: 
1. Вибір зображення із каталогу
2. Редагування зображення переп відправкою
3. Розсилка картинок.
![](https://github.com/VolodymyrTymets/articles/blob/master/node-red/img/moc.png?raw=true)

Досить проста задача але як для прикладу більшої і не потрібно. 

## Вирішення
Провівши буквально декілька годин за розробкоюможна побудувати досить непоганий UI для такої задачі. Використовуючи node red можна значно скоротити цас розробки подібних задач. Адже node-red надає нам більшу частину функціоналу. Нижче наведено приклад додакку на який у мене пішло буквально пів дня.

![](https://github.com/VolodymyrTymets/articles/blob/master/node-red/img/node-red-res.gif?raw=true)
Далі розглянемо покроково як настроїти і розробити даний проект використовуючи node-red + React.

## Настройка React Boilerplate wiht Node js
Оскільки нові палети в node red [підключаються](https://nodered.org/docs/creating-nodes/first-node) як npm пакети є два вибори. Реєструвати кожен палет як окремий пакет або ж реєструвати набір палетів як один пакет. Це залежить від того розробляєте ви комплекс із кількох палетів чи один окремий. Другий випадо на мою думку більш типовий і я вирішив взяти за основу його. Тому розробив приблизну структруру проекту:

```
|_ pallets 
    |_ <categoryName>-<palletName> 
        |_ icons
        |_ ui 
           |_ index.js
           |_ build/main.js
        |_ <categoryName>-<palletName>.html
        |_ <categoryName>-<palletName>.js 
|_ src
   |_ client/containers 
   |_ server 
     |_ pallete-managers 
        |_ <palletName> 
   |_ utils
|_ test
   |_ unit
```
Нижче наведено пояснення до дерикторій та файлів проекту:
![](https://github.com/VolodymyrTymets/articles/blob/master/node-red/img/fig2.png?raw=true)
> Повний код проекту доступний [тут](https://github.com/VolodymyrTymets/node-red-react-example)

Отже тепер можна приступати до написання нашого проекту. Будемо слідувати крокок за кроком і детально роглядати кожен з них.


## Створення першого mode red pallet
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

## Реєстрація палету в node red
Тепер коли ми створили наш перший палет можна додати його в node red і запустити його. Перш за все слід [встановити node red](https://nodered.org/docs/getting-started/). Далі слід підключити ваш [пакет локально](https://nodered.org/docs/getting-started/adding-nodes).
```
cd $HOME/.node-red
npm install <npm-package-name>
```
> node red зберігає всі свої файли в .node-red дерикторії. Ви можете переглянути .node-red/pacakge.json і моніторити усі пакети (палети) які ви підключаєте

І тепер все що нам залишилося це запустити `node-red` і перевірити чи усе працює.
## Node Red Server Side more professional
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
// /pallets/apiko-image-cropper/apiko-image-cropper.js
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

## Add React for UI
Інша річ яка мене розчарувала це те що у всіх прикладах node red пропонує нам використовувати html + jq. Якщо нам потрібно просто розробити просту форму для вводу даних тоді цього думаю достатньо. Але ж у нашуму випадку це вже не проста форма. Нам слід показати список картинок у зрочному представленні і додати можливість обрізаці вибрану картінку. За допомогою React це можна реалізувати за допомогою двох пакетів:
- [react-photo-gallery](https://www.npmjs.com/package/react-photo-gallery);
- [react-image-crop](https://www.npmjs.com/package/react-image-crop)
 
За допомогою jq - я і не досліджував це питання. Все ж на React View js or Angular набагато простіше писати ui. Тому я вирішив що у мому випадку доцільніше буде використати React.

#### Create our Component
Ну що ж попробужм створити наші перші React компоннти у дерикторії `/src/client/containers`створимо наш компонет і контейнер

```
// /src/client/containers/ImageCropper/Container.js
import React from 'react';
import { compose, branch, renderComponent, withHandlers, withState } from 'recompose';

import Component from './Component';

const enhancer = compose(
  ...
  withState('crop', 'setCrop', { x: 10, y: 10, width: 80, height: 80 }),
  withHandlers({
    onSelectFile: props => e => {
      ...
    },
  }),
);

export default enhancer(Component);

// /src/client/containers/ImageCropper/Container.js
....
import ReactCrop, { makeAspectCrop } from 'react-image-crop'
import 'react-image-crop/dist/ReactCrop.css'

import './style.css';

const ImageCropper = ({ node, src, crop, onSelectFile, onImageLoaded, onCropComplete, onCropChange }) => (
  <div className="apiko-image-cropper">
    <h3>Image Cropper</h3>
    <div>
      {src && (
        <ReactCrop
          src={src}
          crop={crop}
          onImageLoaded={onImageLoaded}
          onComplete={onCropComplete}
          onChange={onCropChange}
        />
      )}
    </div>
    <div className="form-row">
      <label htmlFor="node-input-name"><i className="fa fa-tag"></i> Name</label>
      <input type="text" id="node-input-name" placeholder="Name" defaultValue={node.name}/>
      <input type="text" id="node-input-cropX" value={crop.x} className="hide" />
      ...
    </div>
    ...
  </div>
);

export default ImageCropper;
```
Я упустив деякі деталі щоб не розтянувати данну статтю. Усе вище згадане це простий Ract + Recompose. Думаю на цьому не варто зупинятися. Повний код доступний [here](https://github.com/VolodymyrTymets/node-red-react-example/tree/master/src/client/containers/ImageCropper)
Як і уже заначалося вище занчення інтпутів `<input type="text" id="node-input-cropX"` передаватимуться на сервер. Тобто якщо користувач введе значення 3 node-red знатиме що змінна 'cropX===3'. В нашому випадку ці змінні проставлє сам реакт. Поля приховані що б не збивати з пантелику користувача.
Також тут хотілося б відзначити змінну `window.Apiko.constants.lastSelectedUrl.value`. На жаль в node-red немає механізму щоб передавати змінні з ондного палета в інший (на клієнті). Це мене трохи розчарувало адже досить часто приходиьтся робити UI основі значень вибраних в попередньому палеті. Тому приходиться використовувати ось такі уловки. 

#### Build React
Ну щож тепер у нас є наші React components. Як же нам їх помістити в наш node-red pallet? У файлі `/pallets/apiko-image-cropper/apiko-image-cropper.html` у нас є елемент ` <div id="apiko-image-cropper">` Відповідно в нього нам слід і рендерити наш компонент.  Отже створимо новий файл і `/pallets/apiko-image-cropper/ui/index.js` який просто hєструє наш компонет. 
```
import ImageCropper from '../../../src/client/containers/ImageCropper';
const onImageCropper = (node) =>
  ReactDOM.render(<ImageCropper node={node} />,
    document.getElementById('apiko-image-cropper'));
```
Тпер на треба якимось чином це файл підключити до node red. можна подумати що достатньо його просто підключити до `apiko-image-cropper.js`. Але ні. Даний файл виконуться на сервері а нам потрібний клієнт. Хм тоді моджe `apiko-image-cropper.html`? І знову ні. Даний файл просто підправляється на клієнт у чистому вигляді без будь якої компіляції. У тут і починається найцікавіше:
1. Слід збілдити наш React у виконуваний файл
2. Слід описати Api endpoint щоб викликати наш файл з сервера
2. Слід виконати виконуваний фал під час відображення нашого палету

Усе по порядку.
##### 1. React build
 Ну тут на допомого прийде старий добрий [webpack](https://webpack.js.org/). Весь конфіг доступний [here](https://github.com/VolodymyrTymets/node-red-react-example/blob/master/webpack.config.js). Єдини на що я б звернув увагу це: 
```
 const { PALLET_NAME } = process.env;
 ...
 output: {
    path: path.resolve(__dirname, `./pallets/${PALLET_NAME}/ui/build`)
  }
```
Тепер можна викликати команду `PALLET_NAME=apiko-image-cropper webpack --mode development`. Це потрбіно для того щоб не копіювати файл в репозиторі. Але ви можете збілдити і скопіювати в ручну у дерикторі `./pallets/apiko-image-cropper/ui/build`.

##### 2.Api endpoint
Тепер слід якимось чином віддати наш виконуваний файл з сервера. На щасття node red надає можливітсть описувати свої [api points](https://nodered.org/docs/api/runtime/api) У файлі `apiko-image-cropper.js`: 
```
// /pallets/apiko-image-cropper/apiko-image-cropper.js
const { ImageCropperPalletManager } = require('../../src/server/pallet-managers/image-cropper');
const  MODULE_NAME = 'apiko-image-cropper';

module.exports = function(RED) {
  ...
  RED.nodes.registerType(MODULE_NAME, nodeGo);

  RED.httpAdmin.get(`/${MODULE_NAME}/js/*`, (req, res) => {
    res.sendFile(req.params[0], {
      root: __dirname + '/ui/build', // <- get code form this dir
      dotfiles: 'deny'
    });
  });
};
```
Отже ми зареєстрували новий api point `/apiko-image-cropper/js/` тепер по запиту GET `/apiko-image-cropper/js/main.js` на код шукатиме файл `main.js` у `pallets/apiko-image-cropper/ui/build/main.js`. Тепер залишилося тільки викликати наш запит з клієнта. 

##### 3. Виклик main.js from server 

Node red надає нам два [калбеки](https://nodered.org/docs/creating-nodes/node-html) які ми можемо викристати для цих цілей.  `oneditprepare` - called when the edit dialog is being built (кожен раз коли ми відкриваємо вікно) та `onpaletteadd` - alled when the node type is added to the palette. Тому у фалі `apiko-image-cropper.js`: 
```
 oneditprepare: function () {
      var node = this;
      const fileName = '/apiko-image-cropper/js/main.js';
      $.getScript(fileName) // <- get React code form server part of pallet
        .done(function() {
          window.Apiko.onImageCropperLoad(node);
        })
        .fail(function(jqxhr, settings, exception ){
          console.log('Fail of code load from:[' + fileName + ']', exception)
        });
    },
```
> `oneditprepare` - чому саме? тому що це зручно при розробці кожен раз отримувати новий код не перезагружаючи сторінку. В продакшен можна змінити на `onpaletteadd`.

Отже тепер нам лишилося запустити наш node red і перевірити чи відрендирився наш компонент. 

#### Add real time update
Осатннє що б хотілося показати як додати real time update у подібний проект. 
Перше з чим я стикнувся що мені слід білдити не один палет а кілька. Ми ж розробляли два палети `imageCropper` та `imageGallery`. Тому мені кожен раз би приходилося запускати два скріпти `PALLET_NAME=apiko-image-cropper webpack --mode development && PALLET_NAME=apiko-image-gallery webpack --mode development`. Я пішов лекшим шляхом і нписав невеличкий скріпт `build.sh`.
```
# /build.sh
for PALLET_NAME in "$@"
do
 echo "Build fo $PALLET_NAME "
 PALLET_NAME=$PALLET_NAME npm run build ./pallets/$PALLET_NAME/ui
done
```
Що він робить так це проходиться по масиву відних параметрів і білдить react. вхідними параметрами виступатимуть назви деректорій у `./pallets/`. Тепер можна просто написати npm script
```
// package.jscon
"build-all": "sh build.sh apiko-image-cropper apiko-image-gallery",
```
І останні крок це додати виклик цього скріпта після будь якої зміни вашого коду. В цьому на поможе [nodemon](https://www.npmjs.com/package/nodemon). А оскільки ваші палети ветчать новий `main.js` з кожним відкриттям модалки ви отримаєте новий код. Отже додамо наступний скріпнт в `package.jscon`.
```
// package.jscon
  "start": "nodemon --watch pallets --watch src/client --exec 'npm run build-all'"
```
Ну і звичайно вам ще треба буде запустити `node-red`.
## Conclusion
Отже у даній статті показано розробку невеликого проетку з використанням технології node-red. Показано яким чином настроїти і організувати як невеличкий так і досить значний проект. Також показано як використовувати React для ui частини node-red. Подібним чином можна використовувати і інші ui фрйворки як View js, Angular etc.

