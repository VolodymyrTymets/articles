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

## Sound vizualiztion
## Sound streaming
   stream audio file
   stream server mic
