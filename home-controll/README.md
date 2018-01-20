# Управляння будинком з java script | Java Script everivere 
   Уявіть собі ситуацію, що у вас дома є акваріум з рибками. Щоб зробити його красивішим ви посадили в ньому водорості. Але от біда, виявляється їм потрібне світло. І не просто світло а світло у певні години доби. Допустимо з 9 ранку по 4 вечора. А у вас робочий графік з 8 до 6. Було б зручно мати додаток на своєму телефоні який би дозволим вмикати чи вимикати світло прямо з робочого місця. Або ж настроїти його вмикання вимикання просто вказавши певні години і зберігши їх.
Чи скажімо у вас електричне опалення дому. А ви цілий день на роботі або у частих розїздах. І вам би хотілося виключати опалення після виходу з дому і скажімо за дві години до того як прийдте. Аналогічний додаток був би також корисним.
У цій статті я бхотів навести невеличкий приклад контролю будинком з використання різних технологій та мови java script. Спробувати показати як можна, написати невеликий мобільний додаток для контролю за виключення включенням живлення у одній із резеток вашого дому.

# Основна ідея

Основна ідея полягає в тому що існує певний контроллер у вашому домі, який може може управляти електрекою, пристроями чи просто стежити за вашим домомо. Та певний мобільний додаток за допомогою ви можете управляти цим контроллером (система розумний будинок). Схема зображена нижче.
![](https://github.com/VolodymyrTymets/articles/blob/master/home-controll/home-controll.PNG?raw=true)
   
У якості контроллера можна може виступати [Raspberry Pi](https://www.raspberrypi.org/). Це однокристальний міні компютер, яктй містить усі переваги та функціонал звичайного компютера проте набагато менший і дозволяє керувати технікою. У якості пристрою взаємоді з користувачем може виступати звичайний телефон. Таким чином отримає досить зручне рішення для керування домомо. Для прикладу я використав звичайну плату Raspberry Pi і телефон на базі андроїд для віддаленого управління одного із джерел живлення дому.

## Технічне забезпечення 
Перш за все слід ззабезпечити технічний аспект майбутнього проекту. Cписок технічного забезпечення аиглядатиме наступним
- джерело живлення (резетка, чи переноска)
- [Raspberry Pi](https://www.raspberrypi.org/) 
- флеш память micro cd
- wifi адаптер (при наявності) або enternet cabel
-  [Relay](https://en.wikipedia.org/wiki/Relay)
-  смартфон

Оскільки raspberry py містить тільки 5v gpio виходи, для контролю технікою потрібне додаткове живлення на 220v.  

### Raspberry
Raspberry py це однокристальний міні компютер який використовують для управлянн різного роду технічного забезпечення в основному з віддаленийм керуванням. Невеличкий список hardware які може використовувати ця маленька плата.
- GPIO controllers (для управляння схемотхнікою)
- Ethernet Wifi
- Bluetooth
- Sound, Microphone, Camera
- GPS cordinates
- [Difference sensors](https://www.whiteboxes.ch/?gclid=EAIaIQobChMIos-W2YXm2AIVCIuyCh2fuA7VEAAYASAAEgJZTvD_BwE) - temperature, pollution

Можливості застосування у бізнес проектах також чималі. Ось тільки невеликий список тих про які я чув.

- Системи розумний будинок
- Робототехніка
- Відстеження публічного транспорту (GPS treking)
- Системи візуального моніторингу (передача Sound, Microphone, Camera скажімо на сервер чи мобільний)
- Системи моніторингу навколишнього середовища у аграрному чи науковому секторі (temperature, pollution sensors)
- У медицині

### 220 Power Controll
Для управління подачею живлення можна викорисати реле зєднане із джелом живлення та raspberry. Нижче наведено два типи даних пристроїв.
![](https://github.com/VolodymyrTymets/articles/blob/master/home-controll/0117181801.jpg?raw=true)
Принцип їх дії дуже простий. При подачі струму 3 - 5v дані пристрої замикають елекричне коло підєднане до виходу. Детальніше можна глянути [тут](https://forum.arduino.cc/index.php?action=dlattach;topic=496516.0;attach=223340). Я використав другий тип оскільки він дешевший, ну і тому що перший тип мені продали в неробочому стані. Але для енегроємких пристроїв краще використовувати перший тип.

### пристрій взаємодіЇ з користувачем
У якості пристрою взаємоді з користувачем може виступати звичайний телефон на базі android чи ios. Або ж інший персональний компютер (якщо в цьому є потреба). Для даної статті я розробив не складний дадаток із таким інтерфейсом.
![](https://github.com/VolodymyrTymets/articles/blob/master/home-controll/QuickMemo+_2018-01-15-22-21-39.png?raw=true)

### Усе разом

Для наочного прикладу я використав звичайну новорічну гірлянду щоб показати наочно можливості управління додом з мобільного пристрою. Все тому що мені її було не жалко і вона була під рукою. Все що я зробив це розрізав один із проводів які ішли від вилки і підїднав їх до реле, як показано на рисунку нижче. Таким чином розімкнув колою. Тако ж підєднав реле до вихлдів Raspberry.

> Три входи GND ground (green), UCC 5v power (read) IN 3v programed pin (yelow) [карта виходів](http://webofthings.org/wp-content/uploads/2016/10/pi-gpio.png)  

![](https://github.com/VolodymyrTymets/articles/blob/master/home-controll/0117181915.jpg?raw=true)

> Таким же чином ви можете зєднати будь яке електричне коло чи джерело живлення у вашому домі. Взявши навіть ту саму переноску. Та отримаєте повністю контролюоване та програмоване джерело живлення для будь яких пристроїв. 

Усе що тепер залишилося це підєднати нашу герльнду в резетку і перевірити чи все працює. Що в мене вийшло наведено нижче.
![](https://github.com/VolodymyrTymets/articles/blob/master/home-controll/res.gif?raw=true)

## Програмна реалызація 
Оскільки контроль  здійснюється через raspbery gpio а сама raspberry контролюється через телефон тому слід було забезпечити належну комунікацію між пристроями. Найшвидний спосіб написати невеличкий REST Api сервер який працюватиме на raspberry і приймтимен команди від телефона. Нижче наведено список техологій
 - node js express - технологія розробки server api на javacsript
 - react native  - технологія розробки native mobile app

![](https://github.com/VolodymyrTymets/articles/blob/master/home-controll/home-controll-pz.PNG?raw=true)

Проте існують і інші способи взаємодії з Raspberry
- ethernet 
- через віддалений сервер (в такому випадку ви розгортаєте ваші api десь у хмарі посилаєте йому команди а він вже пересилає ці команди разбері)
- через блютус протокол. Детальніше [тут](https://www.hackster.io/inmyorbit/build-a-mobile-app-that-connects-to-your-rpi-3-using-ble-7a7c2c)

### Mobele Client
Оскільки в якості клієнта виступає мобільний пристрій, тому було написано невеликий мобільний додаток.  Для його написання використано технологію React native.


Для початку створенно додаток за допомогою [expo](https://expo.io/). Як це зробити детально наведено [тут](https://expo.io/learn). Виконавши усі інструкції ви зможете побачити стартове вікно вашого першого додатку. Звісно його слід замінити на потрібне нам. Тож я створив деректорію де розмістив скрін із потрібною нам логікою та виглядом у `src/view/views/LightControl`(детальніше про структуру папок можна прочитати [тут](https://medium.com/the-react-native-log/organizing-a-react-native-project-9514dfadaa0)). Даний скрін представлятиме логіку нашого додатку. Нижче наведено фрагмент коду розмітки згаданого вище скріна
```
const LightControl = ({ onPress, onSelectDate, time }) => (
  <View style={[styles.container]}>
    <View style={[styles.margin]}>
      <Button
        raised
        large
        onPress={onPress('on')}
        buttonStyle={[styles.colorBlue]}
        textStyle={{textAlign: 'center'}}
        title='Light on' />
    </View>
    <View style={[styles.margin]}>
      <Button
        raised
        large
        onPress={onPress('off')}
        buttonStyle={[styles.colorBlack]}
        textStyle={{textAlign: 'center'}}
        title='Light Off' />
    </View>
    ...
  </View>
);
```
> У даній статті я вирішив упустити детальні пояснення особливостей тої чи іншої технологіЇ щоб показати загальну картину в цілому. детальніше про react natime можна почитати (силка на якусь нау статтю про react native із нашого блогу)

Наведений вище фрагмент коду відповідає за відображення дох кнопок для включення і виключення світла.
Оскільки мобільний клієнт шле запити до нашого сервера логічно було написати вевний класс для взаємодї із сервером `src/utils/api.js`.
```
const BASE_URL = 'http://{ip}:3000/api/v1/';
class Api {
  _callApi(url, options = {}) {
  ...
  }
  lightOn() {
    return this._callApi('light/on', {method: 'POST'});
  }
  lightOff() {
    return this._callApi('light/off', {method: 'POST'});
  }
}
export default new Api();
```
> `ip` в `BASE_URL` слід замінити на ip вашої Raspverry. Введіть `ifcongig` пыдключившись до вашої raspberry

Уся бізнес логіка описана за домогою [recompose](https://github.com/acdlite/recompose). Обробка кліків по згаданих вище кнопках наведена нижче. Весь код доступний у репозиторії.
```
withHandlers({
    onPress: props => type => () => {
      if (type === 'on') {
        return api.lightOn().then(console.log('-> light on')).catch(logError);
      }
      if (type === 'off') {
        return api.lightOff().then(console.log('-> light off')).catch(logError);
      }
      ...
    },
    ....
})
```
Отже таким чином, за кільк агодин однією людиною може бути написаний невеличкий додаток який дозволить взаємодяйти та контролювати техніку у вашому домі. Також цей додаток може буде запросто запущений на платформах Android та ios що значно зекономить час розробки подібних біснес проектів. 

> Весь код розробленого додатку можна найти [тут](https://github.com/VolodymyrTymets/home-controll)

### Rasspberry server
Перш за все на нову raspberry потрібно встановити операційну систему. Як це зробити детальніше наведено ось [тут](http://thisdavej.com/beginners-guide-to-installing-node-js-on-a-raspberry-pi/). Далі слід встановити зєднання із raspbery та персональним компютером. це можна зробити через ethernet. Детальнше можна почитати [тут](https://stackoverflow.com/questions/16040128/hook-up-raspberry-pi-via-ethernet-to-laptop-without-router). Таким чином ви зможете управляти raspberry через ssh.  Оскільки в весь проект розроблявся на javascript то відповідно серверна частина написана на node js із використання express js.

Для генерації epress проекту можна використати [express-generator](https://www.npmjs.com/package/express-generator). Він сфорує для вас потрібну структуру проекту і все що залишться це розширяти його дописуючи функціонал. Тепер мжна приступати до розроки нашого контроллера. І почнеми із управління технікою. 
Для управляння різного роду технікою Raspberry pi використовує GPIO. Для управління ним на node js можна використати бібліотеку [onoff](https://www.npmjs.com/package/onoff).  Чудова бібліотека із чудовою документаціїю.
Для управлянні подачею електрики запрограмавоно один із виходів GPIO. Контроль здійснюватиметься за допомогою класу `GpioController` в `utils/GpioController.js`.
```
var Gpio = require('onoff').Gpio;
class GpioController {
  constructor(lightNumber = 17) {
    this._led = new Gpio(lightNumber, 'out');
  }
  lightOn() {
    return new Promise((resolve,reject) => {
        this._led.write(1, (err, value) => {
          if (err) return reject(err);
          return resolve(value)
        });
    });
  }
  lightOff() {
    return new Promise((resolve,reject) => {
      this._led.write(0, (err, value) => {
        if (err) return reject(err);
        return resolve(value)
      });
    });
  }
}
module.exports = GpioController;
```
Даний клас містить два методи `lightOn` та `lightOff`, за допомогою яких можна конролювати подчу живлення на один із виходів.
>  Номер виходу не відповідає порядковому номеру на самій платі номер pin можна знайти [тут](http://webofthings.org/wp-content/uploads/2016/10/pi-gpio.png)

Оскільки Raspberry прийматиме REST запит було створенно два роути для управляння pin виходом.
```
 Light on  POST /api/v1/light/on
 Light off  POST /api/v1/light/off
```
Реалізація одного з таких роутів наведено нижче.
```
const GpioController = require('../../utils/GpioController');
const offLight = async (req, res, next) => {
  try {
    console.log('light off');
    const lightController = new GpioController();
    await lightController.lightOff();
    res.status(200).end();
  } catch (error) {
    console.log('Gpio not detected');
    return next(error);
  }
};
module.exports = offLight;
```
Також як було видно наш клієнт містиь інтерфейс для встановлення часу роботи якогось пристрою у вашому домі автоматично, з по певні години. Для реалізації цієї логіки можна використати пакет [node-cron](https://www.npmjs.com/package/node-cron). Відповідна задача була реаізована у `tascks/`.
```
const moment = require('moment');
const GpioController = require('../utils/GpioController');
const switchLight = async () => {
  try {
    if (global.time && global.time.from) {
      if (moment().isBetween(moment(global.time.from), moment(global.time.to))) {
        const lightController = new GpioController();
        await lightController.lightOff();
      } else {
        const lightController = new GpioController();
        await lightController.lightOff();
      }
    }
  } catch (error) {
    console.log(error)
  }
};
const cron = require('node-cron');
cron.schedule('30 * * * * *', switchLight);
```
> у даному коді використано константу глобальну змінну `global.time.from`, що не рекомендується робити у серьозних проектах. Оскільки це тестовий проект я згрішив щоб скороти час розробки.

Також слід прописати новий роут щоб зберегти час роботи введеного із клієнта.
```
 Light on  POST /api/v1/time/set
 Light off  POST /api/v1/time/unset
```
Реалізація одного з таких роутів наведено нижче.
```
const moment = require('moment');
const setTime = async (req, res, next) => {
  try {
    const { from, to } = req.body;
    global.time =  global.time || {};
    global.time.from = moment().hour(from.hour).minute(from.minute).toDate();
    global.time.to = moment().hour(to.hour).minute(to.minute).toDate();
    res.status(200).end();
  } catch (error) {
    return next(error);
  }
};
module.exports = setTime;
```
Ось так у кілька кроків можна написати робоче Api для контролю Raspberry py через вашу локольну мережу. Написаний вами сервер можна залити через ваш git репозеторій абож засобами ssh локально. Та запустити звичайною командою `npm start`. І якщо вам потрібно що вас сервер продовжував роботу при включенні raspberry можна використати [npm forever](https://www.npmjs.com/package/forever).

> Весь код розробленого додатку можна найти [тут](https://github.com/VolodymyrTymets/home-controll)


## Висновки 
У даній статті я показав як за досить невеликий термін, одніїю людиною та на одній мові програмування може бути написаний невеликий додаток, який дозволить контролювати ваш будинок. Тако ж хотів показати наскільки наскільки розвинулася технологія мови javascript. Ажде спершу вона задумувалася як просто мова для браузерної сторінки, для управління простими сайтами і не більше.
Одна мова програмування дозволить вам зекономити час розробки комерційних додатків такого типу. Адже вам не потрбно наймати кількох спеціалістів (Raspbery py спеціаліст, Android Developer, Ios Developer). Одна спеціаліст зможе конролювати весь процес розробки проекту. Це значно зекономитьт час розробки та кошти потрачені на проект.
І взагальному можливості розробки з використанням raspberry py + mobile виходять далеко за межі управляння домом. Якщо в думатися в список пристроїв якеми може оперувати ця маленька плата, можна багато чого корисного розробити.
