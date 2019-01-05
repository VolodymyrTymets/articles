# How to work with sound in javascript
Не так давно я мав можливість попрацювати із звуком в межах одного проекту. Для цього прийшолся дослідити дане питання дещо глибше. Завдяки цьому я дізнався багато нового та цікавого про те які насправді процеси відбуваються коли ми просто слухаємо музику. Тому я вирішив узагальнити нові знання і поділитися ними із іншими людьми. Можливо комусь також пригодяться дані знання. А почнем ми із невеличкої теоріх і плавно перейдем до прикладів того що можна робити у реальних проектах із звуком.
## Sound nature in computing
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

## How to load from the server
## What we can do with sound
### play sound
### visualize sound
### a litle bit of math
### listen of mic
## Sound streaming
   stream audio file
   stream server mic