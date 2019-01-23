# How to work with sound in javascript
Не так давно я мав можливість попрацювати із звуком в межах одного проекту. Для цього прийшолся дослідити дане питання дещо глибше. Завдяки цьому я дізнався багато нового та цікавого про те які насправді процеси відбуваються коли ми просто слухаємо музику. Тому я вирішив узагальнити нові знання і поділитися ними із іншими людьми. Можливо комусь також пригодяться дані знання. А почнем ми із невеличкої теоріх і плавно перейдем до прикладів того що можна робити у реальних проектах із звуком.
## Sound nature in computing
> В даному розділі буде наведено те

Перш завсе слід вдатися до початків і згадати а що таке власне звук? n physics, sound is a vibration that typically propagates as an audible wave of pressure, through a transmission medium such as a gas, liquid or solid (from [wiki](https://en.wikipedia.org/wiki/Sound)). Якщо коротко то це прості коливання повітря які вловлює нші вуха. Якщо прдставити звук грфічно то це хвиля яку можна позначети як f(t) де t - часовий інтервал.
![](https://github.com/VolodymyrTymets/articles/blob/master/sound-in-js/img/fig1.png?raw=true)
Далі виникає нуступне логічне питання яким же чином наші пристрої відтворють цю хвилю. Для цього використовують [Digital audio](https://en.wikipedia.org/wiki/Digital_audio) - спосіб зберігання звуку у формі цифрового сигналу. Оскільки звук це форма хвилі в момент часу тому ці моменти можна виділити і зберігати у вигляді [samples](https://en.wikipedia.org/wiki/Sampling_(signal_processing)) (числових значень форми хвилі на кожен момент часу).
![](https://github.com/VolodymyrTymets/articles/blob/master/sound-in-js/img/fig2.png?raw=true)
Кожен семпл є набором бітів (значень 0 або 1). Зазвичай використовуються 16-бітні або 24-бітні сигнали. Кількість семплів на секунду в цифровому сигналі визначається частотою дискретизації (sample rate), яка вимірюється в герцах. Чим вища частота дискретизації — тим вищі частоти може містити звуковий сигнал.
Якщо вже дуже упрости ти то можна уявити собі звук як великий (дуже великий) масив значень звукових коливаня (як у байтах так і у числових значннях -N< 0 >N після декодування). Наприклад [0, -0.018, 0.028, 0.27, ... 0.1]. Довжина такого масиву залежить від частоти дискретизації вказаної при записі наприклад частота дискритезації 44400 означатиме що довжина такого масиву складатиме 44400 елементів за 1 секунду запису. Такрж існуж двохканальна запис при якій таких масмивів у вас два.
### How sound is saving on devices
Після того як розібралися із звуковою хвилею настав час розібратися яким же чином на зберігати її на носієві. Для цього викоритсовують [цифровий аудіо формат](https://en.wikipedia.org/wiki/Audio_file_format).  Це всім нам відомі `.wav .mp3 ...`. Але яка що вони із себе представляють?
Кожен аудіо файл складається із двох части data and header. Data - це саме наша звукова хвиля наш масив даних такж відоий як `.raw` формат. Heder - це додаткова інформація для декодування наших даних вона мість інформацію про частоту дескритизації кількість каналів запису та іншу користну нагрузку типу автора альбом дату запису  і так далі.
![](https://github.com/VolodymyrTymets/articles/blob/master/sound-in-js/img/fig3.png?raw=true)
Різниця між `.wav` та `.mp3` в тому що `.mp3` це зжатий формат !{силка}
Від теорії перйдем до практики. Для даної статті всі прикладви зробленні Express + React прое основні підходи закладені у них не привязані до якось конкретного фреймфорка.

## How to load from the server
Перш за все слід взяти якись файл для роботи (для прикладу візьмем `.wav` file).  Тут заргрузити його з клієнта або ж сервера. З клієнта це зробити досить просто використавши [file input element](https://developer.mozilla.org/ru/docs/Web/HTML/Element/Input/file). Яж хотів більше детальніше показати як вигрузити файл із сервера.  Нижче наведений невеликий приклад реалізації на express js. {!повний код}
```
...
const express = require('express');
const app = express();
const api = express();

api.get('/track', (req, res, err) => {
  // generate file path
  const filePath = path.resolve(__dirname, './private', './track.wav');
  // get file size info
  const stat = fileSystem.statSync(filePath);

  // set response header info
  res.writeHead(200, {
    'Content-Type': 'audio/mpeg',
    'Content-Length': stat.size
  });
  //create read stream
  const readStream = fileSystem.createReadStream(filePath);
  // attach this stream with response stream
  readStream.pipe(res);
});

//register api calls
app.use('/api/v1/', api);

const server = http.createServer(app);
server.listen('3001',  ()  => console.log('Server app listening on port 3001!'));
```
В цілому тут 3 основні кроки. Читання файл і інформації про нього. Вказання заголовка `audio/mpeg` into response. Вигрузка самого файлу. Подібним чином ви можете вигружати будь які файли audio video pdf etc. Все що нам залишається це звернутися за адресою `api/v1/track`.

## What we can do with sound on client
Тепер коли ми знаємо як вигружати файли із сервера, наступним кроком буде отримати наш фай на клієнті. Якщо ми просто Виконаємо `GET` !{силка} запит у браузері ми отримаємо на файл. Просте хотілося б використати його на нашій сторінці якимось чином. Найпростіший спосіб це зробити використати елемент [audio](https://www.w3schools.com/html/html5_audio.asp) наступним чином. See example [here](https://sound-in-js.herokuapp.com/example1)
```
  <audio controls>
      <source src="/api/v1/track" type="audio/mpeg" />
    </audio>
```
### Work with sound in background
Звісно чудово що browse api надає нам такі прості елементи із коробки. Проте хотілося б мати більший контроль над звуком всередині нашого кодую. Щоб мати можливість насити свій власний Audio control та кастомізувати його так як вам потрібно. Наприклад такий.  See example [here](https://sound-in-js.herokuapp.com/example2)
![](https://github.com/VolodymyrTymets/articles/blob/master/sound-in-js/img/fig4.png?raw=true)

В цьому може допомогти [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) - це набір інстурментів для роботи із звуком у браузері. Отже з чого слід почати? А почне і звідповіді на кілька запитань.

### Як прогати аудіо файл?
Перш за все слід загрузити його з сервера. Для цього можна скористатится методом [fetch](https://developer.mozilla.org/ru/docs/Web/API/Fetch_API) або ж іншими бібліотеками. Я наприклад викристовую [axios](https://www.npmjs.com/package/axios).

```
 const response = await axios.get(url, {
     responseType: 'arraybuffer', // <- important param
   });
```
> слід вказати `responseType: 'arraybuffer'` into header щоб ваш браузер знав що він грузить саме `buffer` а не `json`

Далі щоб програти файл слід створити клас екземпляр класу [AudioContext](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext). Будь які подальші дії із звуко без нього неможливі.
```
const getAudioContext =  () => {
  AudioContext = window.AudioContext || window.webkitAudioContext;
  const audioContent = new AudioContext();
  return audioContent;
};
```
> Тут є досить важливий нюанс! Деякі браузери дозволюь використовувати AudioContext тільки після взаємодії користувача із сторінкою. Тобто якщо користувач не клікну на жодну кнопку чи інше мусце на сторінці у вас буде помилка. Саме тому `getAudioContext`.

Щоб програти аудіо потрібно створити `BufferSource` для цього в AudioContext є метод [createBufferSource](https://developer.mozilla.org/en-US/docs/Web/API/BaseAudioContext/createBufferSource). Після чого `BufferSource` потребує `audioBuffer` а візьмем ми його із нашого вайлу скориставшись методом [decodeAudioData](https://developer.mozilla.org/en-US/docs/Web/API/BaseAudioContext/decodeAudioData). Разом усе це виглядатиме наступним чином:
```
   // load audio file from server
   const response = await axios.get(url, {
     responseType: 'arraybuffer',
   });
   // create audio context
   const audioContext = getAudioContext();
   // create audioBuffer (decode audio file)
   const audioBuffer = await audioContext.decodeAudioData(response.data);

   // create audio source
   const source = audioContext.createBufferSource();
   source.buffer = audioBuffer;
   source.connect(audioContext.destination);

   // play audio
   source.start();
```
Після чого залишається тіки викликати метод `source.start()`
### Як зупинити програвання?
Просто виклечіть метод метод `source.stop()`. Також вам слід зберегти час коли ви нажали stop. Це необхідно якщо вам прийдеться відновити програвання із місця зупинки. В такеому випадку прийдеться викликати `source.start()` уже з параметром.
```
// start play
let startedAt = Date.now();
let pausedAt = null;
source.start();

// stop play
source.stop();
pausedAt = Date.now() - startedAt;

// resume from where we stop
source.start();
startedAt = Date.now() - pausedAt;
source.start(0, audionState.pausedAt / 1000);
```

### Як відобразити процес програвання?
Тут модна піти двома шляхами скоритсатися методом [createScriptProcessor](https://developer.mozilla.org/en-US/docs/Web/API/BaseAudioContext/createScriptProcessor) та відповідно його callback `onaudioprocess`.
```

    const audioBuffer = await audioContext.decodeAudioData(response.data);
     // create progress source
    const  scriptNode = audioContext.createScriptProcessor(4096, audioBuffer.numberOfChannels, audioBuffer.numberOfChannels);
    scriptNode.connect(audioContext.destination);
    scriptNode.onaudioprocess = (e) => {
         const rate = parseInt((e.playbackTime * 100) / audioBuffer.duration, 10);
    };
```
Для того щоб відобразити скільки відсодкі пісні програло нам потрібно дві речі тривалість пісні `audioBuffer.duration`  та поточний час програвання
`e.playbackTime` а далі чиста математика.

Недоліком є те щопід час виклику `source.stop()` вам прийдеться онуляти даний callback.

Інший спосіб зберезти час початку відтворення і запустити оновслення скажим кодну секунду.
```
const audioBuffer = await audioContext.decodeAudioData(response.data);
...
const startedAt = Date.now();
const duration = audioBuffer.duration;
source.start();

setInterval(() => {
      const playbackTime = (Date.now() - startedAt) / 1000;
      const rate = parseInt((playbackTime * 100) / duration, 10);
  },1000)
```
### Як перемотати до певеного місця?
Тут митуація дещо зворотя. Спершу слід визначити `rate`  а же на його основі вирахувати `playbackTime`. Щоб визначити час `rate` можна взяти за 100 довжину елемента progress та позицію мишки відносно місця куди користувач клікнув.
```
 onProgressClick: (e) => {

      const rate = (e.clientX * 100) / e.target.offsetWidth;
      const playbackTime = (audioBuffer.duration * rate) / 100;

      source.stop();
      source.start(o, playbackTime);
      // dont foger change startedAt time
      // startedAt = Date.now() - playbackTime * 1000;
}
```
> тут також важливо не забути змінити `startedAt` інакше ваш прогрес буде відображатися невірно

### Як управляти гучністю?
Для цього потрібно створити [gainNode](https://developer.mozilla.org/en-US/docs/Web/API/GainNode) викликавши метод `audioContext.createGain();`. Після чого досить легко написати метод `setVolume`.
```
 const gainNode = audioContext.createGain();
 ...
  source.connect(gainNode);
  const setVolume = (level) => {
     gainNode.gain.setValueAtTime(level, audioContext.currentTime);
   };
  setVolume(-1); // mute
  setVolume(2); // speek
```
Ну от  тепер ви знаєте усе щоб написати свій власний компонет для програвання аудіо файлів. Далі справа за тільки в кастомізації і функціональних можливостях які ван необхідні такі як час відтворення назва файлу перехід на наступний трек і так далі.

### A litle bit of math
Тепер коли ви знаєте як програвати аудіо файли як на разунок того щоб генерувувати власні звуки? Хотів показати якмжна у кілтка строк коду генерувати звук різної частотити. Невеликикий приклад можете переглянути [тут](https://sound-in-js.herokuapp.com/example1). повний код можете переглянути [тут](https://github.com/VolodymyrTymets/sound-in-js/tree/master/client/src/components/Example3). Для того щоб генерувати звук різної частити слід методом [createOscillator](https://developer.mozilla.org/en-US/docs/Web/API/BaseAudioContext/createOscillator).

```
const getOscillator = (startFrequency) => {
  const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  const oscillator = audioCtx.createOscillator();
  oscillator.type = 'square';
  oscillator.frequency.setValueAtTime(startFrequency, audioCtx.currentTime);
  oscillator.connect(audioCtx.destination);

  const start = () => oscillator.start();
  const stop = () => oscillator.stop();
  const change = frequency =>
    oscillator.frequency.setValueAtTime(frequency, audioCtx.currentTime); // value in hertz

  return { start, stop, change };
};

let frequency = 100;
const oscillator = getOscillator(frequency);
oscillator.start();

const interval = setInterval(() => {
  frequency = frequency + 100;
  oscillator.change(frequency);
}, 100);
setTimeout(() => {
  clearInterval(interval);
  oscillator.stop();
}, 2000);
```
## How to use microphone
Використання мікрофону пристрою браузером зараз досить поширена практика. На часть зараз це досит просто зробити. Для цього слід просто скористатися викликаом метода `getUserMedia` обєктва [window.navigator]( https://developer.mozilla.org/en-US/docs/Web/API/Window/navigator).  Після чого ви отримаєте доступ до обєкта  `stream` sp якого за допомогою  `AudioContext` сожна зробити `audioSource`. Для отримання chank of data from mic вожна скористатися `createScriptProcessor` тай його методом `onaudioprocess`. Нижче наведено невеликий код як усе це реалізувати разом.
```
// get permission to use mic
window.navigator.getUserMedia({ audio:true }, (stream) => {
    const audioContext = new AudioContext();
    // get mic stream
    const source = audioContext.createMediaStreamSource( stream );
    const scriptNode = audioContext.createScriptProcessor(4096, 1, 1);
    source.connect(scriptNode);
    scriptNode.connect(audioContext.destination);
    // output to speaker
    // source.connect(audioContext.destination);

    // on process event
    scriptNode.onaudioprocess = (e) => {
      // get mica data
      console.log(e.inputBuffer.getChannelData(0))
    };

}, console.log);
```
Із мікрофоном також можна використовувати `audioContext.createAnalyser()` для отрмання спектральних характеристик сигналу. Їх часто використовують для візуалізації звукових сигналів. А те як візуалізувати програвання файлу наведено у наступному розділі. Виж таким же чином можете зроьити це ідля сигналу отриманого із мікрофона.
## Sound vizualiztion
Отже ми вже навчилися створювати власний програвач аудіо файлів. Тепре же в даному розділі я б хотів показати як дещо покращити наш програвач додавши візуалізацію звукової хвилі(Sinewave) та спектральний характеристик(frequency) або  ж equalizer (назвем їх audio bars).  Так як показано нижче.Приклад можна переглянути [тут](https://sound-in-js.herokuapp.com/example4) а повиий код реалізації [тут](https://github.com/VolodymyrTymets/sound-in-js/tree/master/client/src/components/Example4)

![](https://github.com/VolodymyrTymets/articles/blob/master/sound-in-js/img/fig5.png?raw=true)

Отже з чого почакти? Для знадобляться два canvases. Пропишіть їх у html та отримай те доступ до них у js.

```
// in hmml
 <div className="bars-wrapper">
      <canvas className="frequency-bars" width="1024" height="100"></canvas>
      <canvas className="sinewave" width="1024" height="100"></canvas>
</div>
....
// in  js
 const frequencyC = document.querySelector('.frequency-bars');
 const sinewaveC = document.querySelector('.sinewave');
```
До них вернемся пізніше. Із попередіх розділві ми дізналися як використовувати `AudioContext` щоб декодувати файл і програти його. Для того щоб отрмати детальнішу інформацію про нього використовується [AudioAnalyser](https://developer.mozilla.org/en-US/docs/Web/API/BaseAudioContext/createAnalyser). Отже змінимо наш метод `getAudioContext` трошки.

```
const getAudioContext = () => {
  AudioContext = window.AudioContext || window.webkitAudioContext;
  const audioContext = new AudioContext();
  const analyser = audioContext.createAnalyser();

  return { audioContext, analyser };
};
```
Тпер теж сами зробимо із нашим методом `loadFile`:
```
const loadFile = (url, { frequencyC, sinewaveC }) => new Promise(async (resolve, reject) => {
   const response = await axios.get(url, {  responseType: 'arraybuffer' });
   const { audioContext, analyser } = getAudioContext();
   const audioBuffer = await audioContext.decodeAudioData(response.data);
   ...
   let source = audioContext.createBufferSource();
   source.buffer = audioBuffer;
   source.start();
```
Як бачити даний метод приймаж наші canvases як параметри. Тепер нам слід законектити `analyser` до `source` щоб мати сожливість використовувати його для нашого аудіо файла. Також слід  викликати два методи `drawFrequency` and `drawSinewave` для побдови audio bars:
```
source.connect(analyser);
drawFrequency();
drawSinewave();
```
Для побудови Sinewave нам потрібно знати дві речі як взяти дані та спосіб їх відображення. А які ж дані ми будемо відображати? У першому розділі я наводив невеличкий опис про природу звуку і те як він зберігається на пристроях. Там я описав що звук це пеане значення в момент часу digital audio. Отже настав момент розкодувати і відобразити наші точки. Для цього використовується метод [getByteTimeDomainData](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode/getByteTimeDomainData). Проте оскільки даний метод приймає масив ми створимо наш маси спешу.

```
...
 const audioBuffer = await audioContext.decodeAudioData(response.data);
 analyser.fftSize = 1024;
 let sinewaveDataArray = new Uint8Array(analyser.fftSize);
    // draw Sinewave
   const drawSinewave = function() {
     // get sinewave data
     analyser.getByteTimeDomainData(sinewaveDataArray);
     requestAnimationFrame(drawSinewave);

     // canvas config
     sinewaveСanvasCtx.fillStyle = styles.fillStyle;
     sinewaveСanvasCtx.fillRect(0, 0, sinewaveC.width, sinewaveC.height);
     sinewaveСanvasCtx.lineWidth = styles.lineWidth;
     sinewaveСanvasCtx.strokeStyle = styles.strokeStyle;
     sinewaveСanvasCtx.beginPath();

     // draw wave
     const sliceWidth = sinewaveC.width * 1.0 / analyser.fftSize;
     let x = 0;

     for(let i = 0; i < analyser.fftSize; i++) {
       const v = sinewaveDataArray[i] / 128.0; // byte / 2 || 256 / 2
       const y = v * sinewaveC.height / 2;

       if(i === 0) {
         sinewaveСanvasCtx.moveTo(x, y);
       } else {
         sinewaveСanvasCtx.lineTo(x, y);
       }
       x += sliceWidth;
     }

     sinewaveСanvasCtx.lineTo(sinewaveC.width, sinewaveC.height / 2);
     sinewaveСanvasCtx.stroke();
   };
```
Із важливого тут слід пояснити два моменти. Перше це [analyser.fftSize](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode/fftSize) параметр. Він вказує з якою точністю декодувати ауді одані тобто якої довжини буде масив `sinewaveDataArray`. Попробуйте змінити дане значення і побачите наскільки міняється вид хвилі. А друге це [`requestAnimationFrame(drawSinewave)`](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) означає що наша функція виконувитеметь перед кожним оновденням кадрів. У се решта це досить простий код роботи із канвасом.
Для побудови equalizer нпишемо функцію `drawFrequency` Її реалізація нічим не відрізняються від попередньої за винятком виклику метода `analyser.getByteFrequencyData(frequencyDataArray)` та коду робои із канвасом (тепер ми будуємо прямокутники а не лінію).

```
analyser.fftSize = styles.fftSize;
let frequencyDataArray = new Uint8Array(analyser.frequencyBinCount);

   const drawFrequency = function() {
    // get equalizer data
     analyser.getByteFrequencyData(frequencyDataArray);
     requestAnimationFrame(drawFrequency);

     // canvas config
     frequencyСanvasCtx.fillStyle = styles.fillStyle;
     frequencyСanvasCtx.fillRect(0, 0, frequencyC.width, frequencyC.height);
     frequencyСanvasCtx.beginPath();

     // draw frequency - bar
     const barWidth = (frequencyC.width / analyser.frequencyBinCount) * 2.5;
     let barHeight;
     let x = 0;

     for(let i = 0; i < analyser.frequencyBinCount; i++) {
       barHeight = frequencyDataArray[i];

       frequencyСanvasCtx.fillStyle = styles.strokeStyle;
       frequencyСanvasCtx.fillRect(x, frequencyC.height - barHeight / 2, barWidth, barHeight / 2);

       x += barWidth + 1;
     }
   };
```
ну щож тепер ви знаєте як будувати нескладну візуалізацію звуку. Якщо ви раніше працбвали із канвасом думаю додати якісь додаткові ефекти не складе для вас проблему.

## Sound streaming
Раніше ми розглянули як працювати із аудіо файлом. Проте перед початко роботи із ним ми завжди чекали поки він загрузиться. А якщо це файл дісно великий і вадить досить багато то й очікувати користувачу прийшлося б досить довго. А якщо це взагалі не файл а якесь stream source. Про це поговоримо далі.    
### Stream audio filе
Булоб чудово програвати файл і грузити його одночасно. На жаль [Fetch_API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) не підтримують streaming responce. Проте похожий функціонал можна реалізувати за допомогю сокетів.
Для того щоб розбити наш файл на chanks і відправляти на клієнт частинами скористаємося бібліотекою [socket.io-stream](https://www.npmjs.com/package/socket.io-stream). Реалізація виглядатиме  наступним чином. Повний код реалізації доступний [тут](https://github.com/VolodymyrTymets/sound-in-js/blob/master/server/index.js).
```
const ss = require('socket.io-stream');
const server = http.createServer(app);
const io = require('socket.io').listen(server);

io.on('connection', client => {
  const stream = ss.createStream();
  client.on('track', () => {
    const filePath = path.resolve(__dirname, './private', './track.wav');
    // get file info
    const stat = fileSystem.statSync(filePath);
    const readStream = fileSystem.createReadStream(filePath);
    // pipe stream with response stream
    readStream.pipe(stream);
    ss(client).emit('track-stream', stream, { stat });
  });
});
```
Даний код чекає на підключення користувача і події від нього `track` після чого емітить подію `track-stream`. тепер ми отримаємо на слієнті обєкт `stream` з методом `on('data'`.
Тепер на кслієнті нам слід змінти нам метод `loadFile` який ми використовували у попередніх прикладах додавши socket.
```
import ss from 'socket.io-stream';
const socket = socketClient(url);
const loadFile = ({ frequencyC, sinewaveC }, styles, onLoadProcess) => new Promise(async (resolve, reject) => {
   socket.emit('track', () => {});
   ss(socket).on('track-stream', (stream, { stat }) => {
     stream.on('data', (data) => {
       // calculate loading process rate 
       const loadRate = (data.length * 100 ) / stat.size;
       onLoadProcess(loadRate);
       // next step here
     })
   });
```
Даний код конектиться до наого сервера та "make call" track event. після чого отримуємо stream `ss(socket).on('track-stream'` а вже із  stream отримуємо chunk `stream.on('data'`. А маючи інформацію про розмір файлу і довжину chunk можна обрахувати процес завантаження.

Тепер одна з найскладніших части над якою прийшлося добре подумати це як програти файл цілком якщо ми мамо тільки окремі його частини. Тут пиникає одразу кілька проблем.
### Header
Перша проблема в тому що ми не можемо просто взяти і написати ось так.
```
audioContext.decodeAudioData(data); // will throw exeption here
```
А все тому що socket.io-stream відсилає дані в чистому форматі `raw` і decodeAudioData не знає що з ними роби. А іструкції що робити і даними знаходять в header. У першому розділі я нафодив приклад будови файлу і писував там header. ось тепер ця інформація на і пригодилися. щоб вирішити цю проблему можна піти двома шляхами обо виділити header на сервері і відправити клієнту або згенерувати його на клієнті що на мою думку простіши. Для цього можна скористатися функціює `withWaveHeader` (повний код доступний [тут](https://github.com/VolodymyrTymets/sound-in-js/blob/master/client/src/components/Example5/wave-heared.js)). Тепер ша код виглядатиме наступним чином:
```
 const audioBufferChunk = await audioContext.decodeAudioData(withWaveHeader(data, 2, 44100));
```
> відповідні функціх ви можете згенерувати для різних типів файлів або ж на йти в інтернеті. Також інформацію про кількість каналі краще брати із файла але для простоти я упустив цей момент.

### Відтворення файлу в цілому
Тепер коли ми розкодували наші дані ми може їх програти. Але проблема в тому що це леше кусочок даних (навіть на пів секунди) і якщо ми попробуємо програти дані по мірі їх поступлення получиться хаос. Попробуйте виконати наступний код.
 ```
 
 stream.on('data', async (data) => {
       const audioBufferChunk = await audioContext.decodeAudioData(withWaveHeader(data, 2, 44100));
       source = audioContext.createBufferSource();
       source.buffer = audioBufferChunk;
       source.connect(audioContext.destination);
       source.start();
 ```
Ви почуєте набір непонятних вам звуків але не пісню. В все тому що дані приходитимуть набагато швидше ніж кожен кусочок сід програвати. Отже кожен кусочок даних слід програвати із певною затримкою. Але з якою? Напевно ця затримка має бути рівна тривалості попереднього кусочка даних. Попробуйе змінти лише одну строчку коду   
```
  source.start(source.buffer.duration);
```
Даний код означатиме програй мені це аудіо кусок чере `source.buffer.duration` час.




###   stream server mic


