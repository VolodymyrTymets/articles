# Node js однопоточний? Як розпаралелити node js додаток.

Node js розроблявся як однопоточний асинхронний потік викликів. Це означає що ваш код виконуватиметься в одному потоці.
Така архітектура була експерементальною і була дещо відмінною від існуючих де на кожну операцію виділялвся окремий потік.
І вона оправдала себе в системах with streaming and low latency in mind. Ось чудове пояснення  [чому node js однопоточний](https://stackoverflow.com/questions/17959663/why-is-node-js-single-threaded)
З іншої сторони така архітектура не зовсім підходить для задач які вимагають серьозних ресурсих затрат і є накладними для процесора (складні математичні розрахунки, робота з великими файлами). На щастя node js не зовсім однопоточний і дозволяє вам оперувати власними потоками і писати багатопоточні рішення. From https://nodejs.org/en/about/
> Just because Node is designed without threads, doesn't mean you cannot take advantage of multiple cores in your environment. Child processes can be spawned by using our child_process.fork() API, and are designed to be easy to communicate with. Built upon that same interface is the cluster module, which allows you to share sockets between processes to enable load balancing over your cores.
##  Постановка задачі

Не так давно мені прийшлося реалізовувати невеличкий алгоритм. Під час його виконання всі інші частини програми просто зависали. Оптимізувати його було неможливо. Тому єдиним виходом був розпаралелення програми.

Після чого я подумав в яких типових задачах можна застосувати даний підхід. на думку прийшла робота із файлами. Тому я вирішив написати цю статтю взявши за основу невеличкий алгоритм, показавши його роботу в одному та декількох потоках.

Припустимо у вас є декілька файлів абож ціла куча. Це може бути .json .csv .exel не важливо. І вам слід провести якусь операцію над цими фалами розпарсити, зжати проффільтрувати і так далі. Якщо ваш алгоритм виконуватиметься в одному потоці він оброблятимк один файл за одним і це забере досить багато часу. 

Для прикладу я я взяв роботу .wav файлами. Суть в тому щоб взяти максимальні спектри .wav файлів і записати їх у .json. Спектри беруться за допомогою алгоритму [fft](https://en.wikipedia.org/wiki/Fast_Fourier_transform). Він є досить ресурсозатратний що є ідеальним для даної статті.

Ось який результат роботи алгоритму в одному потоці і декількох. В одному файли обробляюьтся по черзі, в декількох паралельно один одному і в два рази швидше.


![](https://github.com/VolodymyrTymets/articles/blob/master/js-threds/img/single-thread.gif?raw=true)
> Fig 1 Обробка файлі в одному потоці

![](https://github.com/VolodymyrTymets/articles/blob/master/js-threds/img/threads.gif?raw=true)
> Fig 1 Обробка файлі в  кількох потоках


## Реалізація в одному потоці 
Для реалізації даного алгоритму слід зробити три основні кроки.
1. Прочитати всі файли у нашій деректорії
2. Зробити над ними певну операції (декодування, пошук спектру)
3. Записати результат виконання (у файл, базу даних і так далі)

Нижче наведений код реалізації даного алгоритму в одному потоці. Повний код доступний у [тут](https://github.com/VolodymyrTymets/js-threads/blob/master/run-single-thread.js)
```
const decodeFiles = async (inFolder, outFolder) => {
	// 1 read all file inside of in folder
	const files = await getFilesInFolder(inFolder);
	for(let i=0; i < files.length; i++) {
		// 2 decode .wav to float array and get spectrum by fff
		decode(files[i].filePath).then(audioData => {
			const { spectrum } = fft(audioData.channelData[0]);
			const { splicedSpectrum } = spliceSpectrum(spectrum);

			// 3 write result ito out folder
			writeToFile(outFolder, files[i].fileName, splicedSpectrum);
		});
	}
};

decodeFiles(path.resolve(__dirname, './assets/in'), path.resolve(__dirname, './assets/out'));
```

При такому підході алгоритм оброблятиме файли послідовно. Кожен наступний файл чекатиме поки виконається попередній. Якщо обробка одного файлу займатиме багато часу то загалтний час виконання буде зростати із збільшенням файлів в дереикторії.

##  Реалізація в кількох потоках
Що реалізувати обробку файлу у окремому потоці ви можете зробити це трьома шляхами:
- запустити ваш код оброьки в окремому потоці за допомогою  [child_process](https://nodejs.org/api/child_process.html#child_process_child_process)
- скористатися пакетами для роботи із потоками із середовища [npm](https://www.npmjs.com/)
- скористатися класом [Worker Threads](https://nodejs.org/api/worker_threads.html) (__!!! experimental for now__)

### child_process
Використовуючи [child_process](https://nodejs.org/api/child_process.html) ми можете запускати додаткові скріпти чи інші  js фали як окремі потоки. Ваш основний потік може читати файли а додаткрвий скріпт виконуватиме усі інші операції над ними. Даний підхід мені не дуже сподобався адже це виглядає так ніби ви запускаєте кілька окремих програм. Тому я не став його реалізовувати проте ви можете почитати детальніше [тут](https://medium.freecodecamp.org/node-js-child-processes-everything-you-need-to-know-e69498fe970a)

### npm threads
Щоб запускати обробку одного фалу у окремому потоці я скористався пакетом [threads](https://www.npmjs.com/package/threads). Він дозволяє запускати js код в окремому пакеті та передавати дані у із основного потоку. Одже наш алгоритм міг би виглядати наступним чином:
```
const spawn = require('threads').spawn;
const { decode } = require('./src/utils/decoder');
const { fft, spliceSpectrum } = require('./src/utils/fft');

const decodeFiles = async (inFolder, outFolder) => {
	// 1 read all file inside of in folder
	const files = await getFilesInFolder(inFolder);
	
	for (let i = 0; i < files.length; i++) {
		const thread = spawn(function (input, done) {
			// code will be run in separate threads
			const { filePath } = input; // get data from main thread
			
			// 2 decode .wav to float array and get spectrum by fft
			decode(filePath).then(audioData => {
				const { spectrum } = fft(audioData.channelData[0]);
				const { splicedSpectrum } = spliceSpectrum(spectrum);

				// 3 write result ito out folder
				writeToFile(outFolder, files[i].fileName, splicedSpectrum);
				done({ success: true });
			});
		});

		thread
			.send({ filePath: files[i].filePath }) // send data to child thread
			.on('message', function (response) { // get response from child thread
				const { success } = response; // get data from child thread
				thread.kill();
			})
	}
};

```
Правда тут виникла одна проблема яку прийшлося виршувати. Ви не можете передати у child thread [функцію](https://github.com/andywer/threads.js/issues/77) із основного потока. Також ви не можете викликати  `require('../some-code');` в child thread.  Отже ваш виклик медоту `decode, fft, spliceSpectrum` просто викличуть `is not a function`.  Тобто усю реалізацію методів `decode, fft, spliceSpectrum` слід писати в середені `spawn(function(input, done) {...`. А це є не найкращим архітектурним рішення.
На щастя у дочірньому потоці можна викликати npm modules. Отдже можна написати локальний пакет який ечкспортуватиме вищезазначені методи для виконання у child thread і спокійно їх використовувати. Розмістимо це пакет в деректорії `./modules/fft-thread-woket`. Та ініціалізуєм пакет командою `npm init`. Після чого в основній репозиторії `npm i --save ./modules/fft-thread-woket`. Тепер даний пакет можна використовувати наступним чинов: 
```

const thread = spawn(function (input, done) {
    // get all code from localpackage
	const { calculateWavSpectrum } = require('fft-thread-worker');
	calculateWavSpectrum(input.filePath, done);
	...
})
...
```
Весь код локального пакету доступний [тут](https://github.com/VolodymyrTymets/js-threads/tree/master/modules/fft-thread-worker).

Тепер уже виглядає краще. Але використовувати усю цю громізтку структуру в основному коді теж не хотілося б. Краще написати Promise чи певний class helper який би брав на себе всю роботу із потоками на себе. Приблизно так це можна реалізувати: 

```
const spawn = require('threads').spawn;
class FFTThreadWorker {
	constructor(payload) {
		this.start = this.start.bind(this);
		this.log = this.log.bind(this);
		this._payload = payload;
	}

	log(message) {
		// console.log(`-> [FFTThreadWorker]: ${message.message || message}`);
	}

	start(filePath) {
		return new Promise((resolve, reject) => {
			const thread = spawn(function (input, done) {
				const { calculateWavSpectrum } = require('fft-thread-worker');
				calculateWavSpectrum(input.filePath, done);
			})
				.send({ filePath })
				.on('message', (response) => {
					thread.kill();
					resolve({ payload: this._payload, response });
				})
				.on('error', reject)
				.on('exit', () => this.log('stopped!'));
		})
	}
}

module.exports = { FFTThreadWorker };
```
Тепер даний код можна просто і лехко використовувати у нашому основному коді використовуючи прості `async/await` or `then/catch`. Тепер фінальний вигляд реалізації алгоритму обробки файлі за допомогою потоків виглядатиме наступним чином. Повний код можна найти [тут](https://github.com/VolodymyrTymets/js-threads/blob/master/run-threads.js)

```
const path = require('path')
const { getFilesInFolder } = require('./src/utils/file-reader');
const { FFTThreadWorker } = require('./src/FFT');
const { writeToFile } = require('./src/utils/file-writer');

const decodeFiles = async (inFolder, outFolder) => {
	// 1 read all file inside of in folder
	const files = await getFilesInFolder(inFolder);

	const results = await Promise.all(
		// 2 decode .wav to float array and get spectrum by fff
		files.map(({ filePath, fileName }) =>
			new FFTThreadWorker({ fileName, filePath }).start(filePath)));

	results.forEach(({
		 payload: { fileName, filePath  },
		 response: { splicedSpectrum }
	}) => {
		// 3 write result ito out folder
		writeToFile(outFolder, fileName, splicedSpectrum);
	});
};

decodeFiles(path.resolve(__dirname, './assets/in'), path.resolve(__dirname, './assets/out'));
```

### Worker Threads
Щоб реалізувати даний алгоритм за допомогою класу [Worker Threads](https://nodejs.org/api/worker_threads.html) слід зауважити що це експерементальний клас і використовувати його в реальних проектах не рекомендовано. Також преконайтеся що у вас версія ноди `v10.8.0` і ви запускаєте ваш код `node ----experimental-worker <you-js-file-path>`.
Реалізаія даного алгоритму схожа із попереднім але дещо простіша. Спершу ствворюємо `./src/utils/node.v10.8.0-fft-thread-worker.js`js фай з кодом яким запускатимемо у child thread. 

```
const {
	isMainThread, parentPort, workerData
} = require('worker_threads');

const { decode } = require('./decoder');
const { fft, spliceSpectrum } = require('./fft');
const { filePath, fileName } = workerData; // get data from main thread

decode(filePath).then(audioData => {
	const wave = audioData.channelData[0];
	const { spectrum } = fft(wave);
	const { splicedSpectrum } = spliceSpectrum(spectrum);
	console.log(`[${filePath}] processed`);
	// send data for main thread
	parentPort.postMessage({ fileName, filePath, splicedSpectrum });
});

```

Тут усе доволі просто імпортуємо `workerData` дані передані із головного потоку та  `parentPort` щоб відправити результат виконання основному потоку. Також існує додаткова константа `isMainThread` назва говорить сама за себе.

Тепер даний файл можна використати у класі  Worker Threads. Реалізація нашого алгоритму за допомогою Worker Threads виглядатиме наступним чином:
```
const {
	Worker, isMainThread, parentPort, workerData
} = require('worker_threads');
const path = require('path');
const { getFilesInFolder } = require('./src/utils/file-reader');
const { writeToFile } = require('./src/utils/file-writer');

const decodeFiles = async (inFolder, outFolder) => {
	console.time('executing time');
	// 1 read all file inside of in folder
	const files = await getFilesInFolder(inFolder);

	const results = await Promise.all(files.map(({ filePath, fileName }) =>
		new Promise((resolve, reject) => {
			// 2 decode .wav to float array and get spectrum by fff
			const worker = new Worker(path.resolve(__dirname, './src/utils/node.v10.8.0-fft-thread-worker.js'), {
				workerData: { filePath, fileName }
			});
			worker.on('message', resolve);
			worker.on('error', reject);
			worker.on('exit', (code) => {
				if (code !== 0)
					reject(new Error(`Worker stopped with exit code ${code}`));
			});
		})
	));

	results.forEach(({ fileName, filePath, splicedSpectrum }) => {
		// 3 write result ito out folder
		writeToFile(outFolder, fileName, splicedSpectrum);
	});

	console.timeEnd('executing time');
};

decodeFiles(path.resolve(__dirname, './assets/in'), path.resolve(__dirname, './assets/out'));
```
Можна також напистати додатковий клас як це вказувалося вище але я вже цього робити не став.

# Summary
У даній статті я взяв типову і поширину задачу та показав як можна її розпаралелити. Ви можете викоритовувати любий із способі і писати дійсно складні програми. Проте не забувайте про одну річ для створення окремого потоку процесору також потрібно затрачати час. Тому якщо ваша задача не вимагає значних затрат процесора то її розпаралеллення призведе тільки до уповільнення вашої програми. Потоки це теж всього лиш інстумент використовуйте їх розумно. 
