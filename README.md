# winston

[![Join the chat at https://gitter.im/winstonjs/winston](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/winstonjs/winston?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

[![Version npm](https://img.shields.io/npm/v/winston.svg?style=flat-square)](https://www.npmjs.com/package/winston)[![npm Downloads](https://img.shields.io/npm/dm/winston.svg?style=flat-square)](https://www.npmjs.com/package/winston)[![Build Status](https://img.shields.io/travis/winstonjs/winston/master.svg?style=flat-square)](https://travis-ci.org/winstonjs/winston)[![Dependencies](https://img.shields.io/david/winstonjs/winston.svg?style=flat-square)](https://david-dm.org/winstonjs/winston)

[![NPM](https://nodei.co/npm/winston.png?downloads=true&downloadRank=true)](https://nodei.co/npm/winston/)

Chúng ta có thể đặt các thông báo lỗi vào một bản ghi . <span style="font-size:28px; font-weight:bold;">&quot;CHILL WINSTON! ... tôi muốn đặt nó trong các nơi log phù hợp.&quot;</span>

## Motivation

Winston được thiết kế để có được 1 đơn giản ghi thông báo logging thư viện hỗ trợ của của nhiều thông báo. 1 Thông báo là cơ bản của 1 
không gian lưu trữ bản ghi log của bạn. Nó giống như một thể hiện của 1 thông báo logger có trong config được định nghĩa bởi level. 

Của 1 ví dụ. 1 thông báo lỗi error logs để có  ta có thể muốn ghi lỗi được lưu trữ trong một vị trí như cơ sở dữ liệu . Bởi vì tất cả các log này có thể được lưu tại console hoặc 1 local file tập tin.


Thư viện này nhằm mục đích để tách những phần của quá trình này để làm cho nó linh hoạt và mở rộng hơn. Làm thế nào để nó được lưu trữ.


## Installation

Cài đặt :

```bashp
npm install winston
```

## Usage
Có hai cách khác nhau để sử dụng winston: trực tiếp thông qua các logger default. hoặc sử dụng bằng cách cài đặt của riêng bạn.

There are two different ways to use winston: directly via the default logger, or by instantiating your own Logger.  Vấn đề trước tiên chỉ đơn thuần được dự định là một logger chia sẻ thuận tiện để sử dụng trong suốt ứng dụng của bạn nếu bạn lựa chọn.

* [Logging](#logging)
  * [Sử dụng một mặc định logger](#using-the-default-logger)
  * [Sử dụng theo cách định nghĩa của riêng bạn](#instantiating-your-own-logger)
  * [logger với các metadata thiết lập](#logging-with-metadata)
  * [Tự động phân tích chuỗi logger](#string-interpolation)
* [Các cơ chế truyền tải lỗi](https://github.com/winstonjs/winston/blob/master/docs/transports.md)
  * [Các cơ chế truyền tải thông điệp cùng lúc](#multiple-transports-of-the-same-type)
* [Cơ chế đơn giản làm việc với bất kỳ logging](#profiling)
* [Log trực tiếp các bản ghi từ 1 nơi lựa chọn](#streaming-logs)
* [log theo từng trường hợp](#querying-logs)
* [Ngoại lệ](#exceptions)
  * [Xử lý ngoại lệ với winston](#handling-uncaught-exceptions-with-winston)
  * [Để Exit hoặc Not Exit](#to-exit-or-not-to-exit)
* [Các loại thông báo lỗi](#logging-levels)
  * [Sử dụng các loại logging levels](#using-logging-levels)
  * [Tùy chỉnhLogging Levels](#using-custom-logging-levels)
* [Đọc các tính năng](#further-reading)
  * [Events và Callbacks trong Winston](#events-and-callbacks-in-winston)
  * [Working với nhiều thông báo trong winston](#working-with-multiple-loggers-in-winston)
  * [sử dụng winston trong một CLI tool](#using-winston-in-a-cli-tool)
  * [Filters và  Rewriters](#filters-and-rewriters)
  * [Adding thêm một hình thức  Transports](#adding-custom-transports)
* [Cài đặt](#installation)
* [Chạy thử Tests](#run-tests)


## Logging

Thông báo level là một  `winston` phù hợp mức độ nghiêm trọng trong [RFC5424](https://tools.ietf.org/html/rfc5424): các mức độ quan trọng giả định được tăng dần từ mức nghiêm trọng đến ít quan trọng.

### Using the Default Logger
Các logger mặc định có thể truy cập thông qua các module winston trực tiếp. Bất kỳ phương pháp mà bạn có thể gọi về một thể hiện của một logger có sẵn trên logger default.

Ví dụ : 

``` js
  var winston = require('winston');

  winston.log('info', 'Hello distributed log files!');
  winston.info('Hello again distributed logs');

  winston.level = 'debug';
  winston.log('debug', 'Now my debug messages are written to console!');
```

Theo mặc định, chỉ có các Console hình thức là một thiết lập mặc định hiển thị thông báo logger.

Bạn có thể add hoặc xóa bỏ một hình thức truyền tải "transports" bỏi các phương pháp add() và remove() :

``` js
  winston.add(winston.transports.File, { filename: 'somefile.log' });
  winston.remove(winston.transports.Console);
```

Hoặc làm điều đó với configure() phương pháp và định nghĩa transports hình thức logger của bạn :

``` js
  winston.configure({
    transports: [
      new (winston.transports.File)({ filename: 'somefile.log' })
    ]
  });
```

Đối với tài liệu hướng dẫn thêm về làm việc với từng config transport được hỗ trợ bởi Winston sẽ thấy trong [Winston Transports](docs/transports.md) tài liệu.

### Instantiating your own Logger
Nếu bạn muốn quản lý các object lifetime thể hiện của object loggers. Bạn có thể làm nó nhanh chóng như sau :
``` js
  var logger = new (winston.Logger)({
    transports: [
      new (winston.transports.Console)(),
      new (winston.transports.File)({ filename: 'somefile.log' })
    ]
  });
```

Bạn có thể làm việc với logger này trong cùng một cách mà bạn làm việc với các logger mặc định:

``` js
  //
  // Logging
  //
  logger.log('info', 'Hello distributed log files!');
  logger.info('Hello again distributed logs');

  //
  // Adding / Removing Transports
  //   (Yes It's chainable)
  //
  logger
    .add(winston.transports.File)
    .remove(winston.transports.Console);
```

bạn cũng có thể cấu hình lại toàn bộ `winston.Logger` thí dụ sử dụng hàm `configure` method:

``` js
  var logger = new winston.Logger({
    level: 'info',
    transports: [
      new (winston.transports.Console)(),
      new (winston.transports.File)({ filename: 'somefile.log' })
    ]
  });

  // thay thế các hình thức trước đó bằng  một cấu hình mới
  // new configuration wholesale.
  //
  logger.configure({
    level: 'verbose',
    transports: [
      new (require('winston-daily-rotate-file'))(opts)
    ]
  });
```


### Logging with Metadata

ngoài logger một chuỗi tin nhắn string messages, winston cũng hỗ trợ thiết lập  kèm theo JSON metadata object và một metadata đơn giản í dụ như:

``` js
  winston.log('info', 'Test Log Message', { anything: 'This is metadata' });
```


Cách các đối tượng được lưu trữ khác nhau từ hình thức transports. để hỗ trợ tốt nhất các cơ chế lưu trữ được cung cấp 

ưới đây là một bản tóm tắt nhanh chóng như thế nào mỗi vận chuyển xử lý siêu dữ liệu metadata:

1. __Console:__ thông báo với  util.inspect(meta)
2. __File:__ thông báo với  util.inspect(meta)

## Multiple transports of the same type

Có thể sử dụng nhiều "multiple transports" giống như các loại level e.g. `winston.transports.File` 
bằng cách đi qua trong một tùy chỉnh `name` khi bạn xây dựng các hình thức transports.

``` js
var logger = new (winston.Logger)({
  transports: [
    new (winston.transports.File)({
      name: 'info-file',
      filename: 'filelog-info.log',
      level: 'info'
    }),
    new (winston.transports.File)({
      name: 'error-file',
      filename: 'filelog-error.log',
      level: 'error'
    })
  ]
});
```

Nếu sau này bạn muốn loại bỏ một trong cá hình thức truyền tải thông điệp, bạn có thể làm như vậy bằng cách sử dụng tên chuỗi name. e.g.:

``` js
logger.remove('info-file');
```

Bạn cũng có thể sử dụng bằng cách loại bỏ chính nó trong ví dụ này tương tự chuỗi trên 

``` js
// Notice it was first in the Array above
var infoFile = logger.transports[0];
logger.remove(infoFile);
```

## Profiling

Ngoài thông điệp logging và siêu dữ liệu, winston cũng có một cơ chế đơn giản hồ sơ thực hiện đối với bất kỳ logger name :

``` js
  // ví dụ với một hồ sơ profile của test name 
  // chạy  profile của config có name 'test'
  //  Ghi chú: Xem xét sử dụng Date.now () với các hoạt động bất đồng bộ async
  //
  winston.profile('test');

  setTimeout(function () {
    //
    // Stop profile of 'test'. Logging will now take place:
    //   "17 Jan 21:00:00 - info: test duration=1000ms"
    //
    winston.profile('test');
  }, 1000);
```


Tất cả các tin nhắn của info level thực hiện bởi default và các tin nhắn thiết lập cùng metadata thiết lập.Điều này không giống như một kế hoặc từ bản đồ trong config cấu hình nhưng tôi có thể mở ra một gợi ý suggestions / issues.

### String interpolation
Các chuỗi tin nhắn có thể được kết hợp từ method `log` giống như  [`util.format`][10].

Điều này cso thể áp dụng cho tất cả các tin nhắn log

``` js
logger.log('info', 'test message %s', 'my string');
// info: test message my string

logger.log('info', 'test message %d', 123);
// info: test message 123

logger.log('info', 'test message %j', {number: 123}, {});
// info: test message {"number":123}
// meta = {}

logger.log('info', 'test message %s, %s', 'first', 'second', {number: 123});
// info: test message first, second
// meta = {number: 123}

logger.log('info', 'test message', 'first', 'second', {number: 123});
// info: test message first second
// meta = {number: 123}

logger.log('info', 'test message %s, %s', 'first', 'second', {number: 123}, function(){});
// info: test message first, second
// meta = {number: 123}
// callback = function(){}

logger.log('info', 'test message', 'first', 'second', {number: 123}, function(){});
// info: test message first second
// meta = {number: 123}
// callback = function(){}
```





## Querying Logs

Winston hỗ trợ query của các log với giống như giống với.  [See Loggly Search API](https://www.loggly.com/docs/api-retrieving-data/).
 Cụ thể: : `File`, `Couchdb`, `Redis`, `Loggly`, `Nssocket`, and `Http`.

``` js
  var options = {
    from: new Date - 24 * 60 * 60 * 1000,
    until: new Date,
    limit: 10,
    start: 0,
    order: 'desc',
    fields: ['message']
  };

  //
  // Tìm kiếm các item lỗi từ hôm này đến hôm qua.
  //
  winston.query(options, function (err, results) {
    if (err) {
      throw err;
    }

    console.log(results);
  });
```

## Streaming Logs

Cho phép bạn hiển thị các logger từ trong hình thức lưu trữ logger của bạn. Như file hiển thị ra console.

``` js
  //
  // Start at the end.
  //
  winston.stream({ start: -1 }).on('log', function(log) {
    console.log(log);
  });
```

## Exceptions

### Handling Uncaught Exceptions with winston

Với `winston`, nó là có thể bắt catch và loggẻ ko xác định `uncaughtException` event từ tiến trình của bạn.

. Có hai cách riêng biệt của phép chức năng này, hoặc thông qua các mặc định winston logger hoặc dụ logger của cấu hình riêng bạn.


Nếu bạn muốn sử dụng tính năng này với các logger mặc định, Đơn giản gọi `.handleExceptions()` với một trường hợp cụ thể thì :

``` js
  //
  //Bạn có thể thêm một logger ngoại lệ riêng biệt bằng cách đi qua đối số `.handleExceptions`
  //
  winston.handleExceptions(new winston.transports.File({ filename: 'path/to/exceptions.log' }))

  //
  // Như một sự lựa chọn bạn có thể để true  `.handleExceptions` khi nạp thêm hình thức lưu trữ đến winston.
  //Bạn có thể sử dụng các  `.humanReadableUnhandledException`tùy chọn để có được ngoại lệ dễ đọc hơn.
  //
  winston.add(winston.transports.File, {
    filename: 'path/to/all-logs.log',
    handleExceptions: true,
    humanReadableUnhandledException: true
  });

  //
  // Trường hợp ngoại lệ cũng có thể được xử lý bởi nhiều hình thức .
  //
  winston.handleExceptions([ transport1, transport2, ... ]);
```

### To Exit or Not to Exit


Theo mặc định, winston sẽ thoát sau khi đăng một uncaughtException. Nếu đây không phải là hành vi mà bạn muốn, 

set `exitOnError = false`

``` js
  var logger = new (winston.Logger)({ exitOnError: false });

  //
  // or, like this:
  //
  logger.exitOnError = false;
```


Khi làm việc với các trường hợp tùy chỉnh logger bạn có thể vượt qua`exceptionHandlers` propertyhoặc set `.handleExceptions` trong tất cả các hình thức  transport.

Example 1

``` js
  var logger = new (winston.Logger)({
    transports: [
      new winston.transports.File({ filename: 'path/to/all-logs.log' })
    ],
    exceptionHandlers: [
      new winston.transports.File({ filename: 'path/to/exceptions.log' })
    ]
  });
```

Example 2

``` js
var logger = new winston.Logger({
  transports: [
    new winston.transports.Console({
      handleExceptions: true,
      json: true
    })
  ],
  exitOnError: false
});
```

The `exitOnError` tùy chọn cũng có thể là một chức năng để ngăn chặn lối ra chỉ một số errors:

``` js
  function ignoreEpipe(err) {
    return err.code !== 'EPIPE';
  }

  var logger = new (winston.Logger)({ exitOnError: ignoreEpipe });

  //
  // hoạc giống như điều này :
  //
  logger.exitOnError = ignoreEpipe;
```

## Logging Levels

Mỗi `Levels` đại diện một loại thông báo và thứ tự nó cao hơn tức quan trọng hơn.


``` js
{ error: 0, warn: 1, info: 2, verbose: 3, debug: 4, silly: 5 }
```

Tương tự như vậy, theo quy định chính xác trong RFC5424 `cái mức syslog` được ưu tiên 0-7 (cao nhất đến thấp nhất).


```js
{ emerg: 0, alert: 1, crit: 2, error: 3, warning: 4, notice: 5, info: 6, debug: 7 }
```


Nếu bạn không xác định một cách rõ ràng các cấp rằng `winston` nên sử dụng` mức npm` trên sẽ được sử dụng.

If you do not explicitly define the levels that `winston` should use the `npm` levels above will be used.

### Using Logging Levels
Thiết lập mức độ cho tin nhắn đăng nhập của bạn có thể được thực hiện theo một trong hai cách.Bạn có thể vượt qua như một chuỗi với  phương pháp log ("name") hoặc sử dụng các phương pháp xác định mức độ xác định trên mỗi winston Logger.

``` js
  //
  // Any logger instance
  //
  logger.log('silly', "127.0.0.1 - there's no place like home");
  logger.log('debug', "127.0.0.1 - there's no place like home");
  logger.log('verbose', "127.0.0.1 - there's no place like home");
  logger.log('info', "127.0.0.1 - there's no place like home");
  logger.log('warn', "127.0.0.1 - there's no place like home");
  logger.log('error', "127.0.0.1 - there's no place like home");
  logger.info("127.0.0.1 - there's no place like home");
  logger.warn("127.0.0.1 - there's no place like home");
  logger.error("127.0.0.1 - there's no place like home");

  //
  // Default logger
  //
  winston.log('info', "127.0.0.1 - there's no place like home");
  winston.info("127.0.0.1 - there's no place like home");
```

`winston` áp dụng một định nghĩa `level` thuộ tính trong mỗi hình thức với các  **maximum** mức độ của thông điệp mà một vận tải phải logger. For example, bằng cách sử dụng `npm` levels  bạn có thể chỉ cần log `error` nhắn cho giao diện điều khiển console và tất cả mọi thứ `info` and và dưới đây là một file cấu hình ví dụ (which includes `error` messages):

``` js
  var logger = new (winston.Logger)({
    transports: [
      new (winston.transports.Console)({ level: 'error' }),
      new (winston.transports.File)({
        filename: 'somefile.log',
        level: 'info'
      })
    ]
  });
```

Bạn cũng có thể tự động thay đổi level  ghi của một hình thức truyền tải sử dụng :

``` js
  var logger = new (winston.Logger)({
    transports: [
      new (winston.transports.Console)({ level: 'warn' }),
      new (winston.transports.File)({ filename: 'somefile.log', level: 'error' })
    ]
  });
  logger.debug("Sẽ không được hiển thị logger trong cả hai hình thức !");
  logger.transports.console.level = 'debug';
  logger.transports.file.level = 'verbose';
  logger.verbose("Will be logged in both transports!");
```
winston hỗ trợ mức đăng nhập tùy biến, mặc định để [NPM] [0] level để khai thác logger. Thay đổi level là dễ dàng:


``` js
  // Thay đổi level về default winston logger
  //
  winston.setLevels(winston.config.syslog.levels);

  // Thay đổi level của một thể hiện  của một thông báo
  //
  
  logger.setLevels(winston.config.syslog.levels);
```

Calling `.setLevels` on a logger will remove all of the previous helper methods for the old levels and define helper methods for the new levels. Thus, you should be careful about the logging statements you use when changing levels. For example, if you ran this code after changing to the syslog levels:

``` js
  //
  // Logger does not have 'silly' defined since that level is not in the syslog levels
  //
  logger.silly('some silly message');
```

### Using Custom Logging Levels

Bạn có thể định nghĩa Ngoài các npm` và 'mức được xác định trước' syslog` sẵn ở Winston, bạn cũng có thể chọn để định nghĩa của riêng bạn:


``` js
  var myCustomLevels = {
    levels: {
      foo: 0,
      bar: 1,
      baz: 2,
      foobar: 3
    },
    colors: {
      foo: 'blue',
      bar: 'green',
      baz: 'yellow',
      foobar: 'red'
    }
  };

  var customLevelLogger = new (winston.Logger)({ levels: myCustomLevels.levels });
  customLevelLogger.foobar('some foobar level-ed message');
```

Mặc dù có sự lặp lại nhỏ trong cấu trúc dữ liệu này, nó cho phép đóng gói đơn giản nếu bạn không muốn có màu sắc. Nếu bạn muốn có màu sắc, ngoài đi qua các cấp độ để Logger chính nó, bạn phải làm cho winston nhận thức của họ:


``` js
  //
  // Hãy winston nhận thức được những màu sắc
  //
  winston.addColors(myCustomLevels.colors);
```
Điều này cho phép transports với các tùy chọn 'colorize' thiết lập để phù hợp màu đầu ra của mức độ tùy chỉnh.

## Further Reading

### Events and Callbacks in Winston

Mỗi trường hợp của winston.Logger cũng là một thể hiện của một [EventEmitter] [1]. Một sự kiện đăng nhập sẽ được tăng lên mỗi lần vận chuyển đăng nhập thành công một tin nhắn:

``` js
  logger.on('logging', function (transport, level, msg, meta) {
   // [Msg] và [meta] hiện đã đăng tại [level] để [transport]
  });

  logger.info('CHILL WINSTON!', { seriously: true });
```
Nó cũng đáng nói đến là logger cũng phát ra một sự kiện 'error' mà bạn nên xử lý hoặc ngăn chặn nếu bạn không muốn trường hợp ngoại lệ unhandled:

``` js
  //
  // Handle lỗi
  //
  logger.on('error', function (err) { /* Làm việc gì đó */ });

  //
  // Hoặc chỉ cần ngăn chặn chúng.
  //
  logger.emitErrs = false;
```

Một phương pháp gọi lại chỉ khi các logger được định nghĩa thành công.

``` js
  logger.info('CHILL WINSTON!', { seriously: true }, function (err, level, msg, meta) {
    // [msg] và [meta] hiện nay đã được ghi lại ở [level] đế **tất cả** transport.
  });
```

### Working with multiple Loggers in winston


Thông thường trong lớn hơn, phức tạp hơn, các ứng dụng cần thiết phải có nhiều trường hợp logger với các thiết lập khác nhau.

``` js
  var winston = require('winston');

  //
  // Configure the logger for `category1`
  //
  winston.loggers.add('category1', {
    console: {
      level: 'silly',
      colorize: true,
      label: 'category one'
    },
    file: {
      filename: '/path/to/some/file'
    }
  });

  //
  // Configure the logger for `category2`
  //
  winston.loggers.add('category2', {
    couchdb: {
      host: '127.0.0.1',
      port: 5984
    }
  });
```


Bây giờ logger của bạn được thiết lập, bạn có thể yêu cầu winston _Print bất kỳ tập tin trong application_ của bạn và truy cập các logger được cấu hình sẵn:

``` js
  var winston = require('winston');

  //
  // Grab your preconfigured logger
  //
  var category1 = winston.loggers.get('category1');

  category1.info('logging from your IoC container-based logger');
```


Nếu bạn muốn quản lý `Container` chính mình, bạn chỉ đơn giản là có thể nhanh chóng một:

``` js
  var winston = require('winston'),
      container = new winston.Container();

  container.add('category1', {
    console: {
      level: 'silly',
      colorize: true
    },
    file: {
      filename: '/path/to/some/file'
    }
  });
```

### Sharing transports between Loggers in winston

Chia sẻ các hình thức giữa các logger trong winston

``` js
  var winston = require('winston');

  //
  // Setup transports to be shared across all loggers
  // in three ways:
  //
  // 1. By setting it on the default Container
  // 2. By passing `transports` into the constructor function of winston.Container
  // 3. By passing `transports` into the `.get()` or `.add()` methods
  //

  //
  // 1. By setting it on the default Container
  //
  winston.loggers.options.transports = [
    // Setup your shared transports here
  ];

  //
  // 2. By passing `transports` into the constructor function of winston.Container
  //
  var container = new winston.Container({
    transports: [
      // Setup your shared transports here
    ]
  });

  //
  // 3. By passing `transports` into the `.get()` or `.add()` methods
  //
  winston.loggers.add('some-category', {
    transports: [
      // Setup your shared transports here
    ]
  });

  container.add('some-category', {
    transports: [
      // Setup your shared transports here
    ]
  });
```

### Using winston in a CLI tool
A common use-case for logging is output to a CLI tool. Winston has a special helper method which will pretty print output from your CLI tool. Here's an example from the [require-analyzer][2] written by [Nodejitsu][3]:

```
  info:   require-analyzer starting in /Users/Charlie/Nodejitsu/require-analyzer
  info:   Found existing dependencies
  data:   {
  data:     colors: '0.x.x',
  data:     eyes: '0.1.x',
  data:     findit: '0.0.x',
  data:     npm: '1.0.x',
  data:     optimist: '0.2.x',
  data:     semver: '1.0.x',
  data:     winston: '0.2.x'
  data:   }
  info:   Analyzing dependencies...
  info:   Done analyzing raw dependencies
  info:   Retrieved packages from npm
  warn:   No additional dependencies found
```

Configuring output for this style is easy, just use the `.cli()` method on `winston` or an instance of `winston.Logger`:

``` js
  var winston = require('winston');

  //
  // Configure CLI output on the default logger
  //
  winston.cli();

  //
  // Configure CLI on an instance of winston.Logger
  //
  var logger = new winston.Logger({
    transports: [
      new (winston.transports.Console)()
    ]
  });

  logger.cli();
```

### Filters and Rewriters
Filters allow modifying the contents of **log messages**, and Rewriters allow modifying the contents of **log meta** e.g. to mask data that should not appear in logs.

Both filters and rewriters are simple Arrays of functions which can be provided when creating a `new winston.Logger(options)`. e.g.:

``` js
var logger = new winston.Logger({
  rewriters: [function (level, msg, meta) { /* etc etc */ }],
  filters:   [function (level, msg, meta) { /* etc etc */ }]
})
```

Like any Array, they can also be modified at runtime with no adverse side-effects to the `winston` internals.

``` js
logger.filters.push(function(level, msg, meta) {
  return meta.production
    ? maskCardNumbers(msg)
    : msg;
});

logger.info('transaction with card number 123456789012345 successful.');
```

This may result in this output:

```
info: transaction with card number 123456****2345 successful.
```

Where as for rewriters, if you wanted to sanitize the `creditCard` field of your `meta` you could:

``` js
logger.rewriters.push(function(level, msg, meta) {
  if (meta.creditCard) {
    meta.creditCard = maskCardNumbers(meta.creditCard)
  }

  return meta;
});

logger.info('transaction ok', { creditCard: 123456789012345 });
```

which may result in this output:

```
info: transaction ok creditCard=123456****2345
```

See [log-filter-test.js](./test/log-filter-test.js), where card number masking is implemented as an example along with [log-rewriter-test.js](./test/log-rewriter-test.js)

## Adding Custom Transports
Adding a custom transport is actually pretty easy. All you need to do is accept a couple of options, set a name, implement a log() method, and add it to the set of transports exposed by winston.

``` js
  var util = require('util'),
      winston = require('winston');

  var CustomLogger = winston.transports.CustomLogger = function (options) {
    //
    // Name this logger
    //
    this.name = 'customLogger';

    //
    // Set the level from your options
    //
    this.level = options.level || 'info';

    //
    // Configure your storage backing as you see fit
    //
  };

  //
  // Inherit from `winston.Transport` so you can take advantage
  // of the base functionality and `.handleExceptions()`.
  //
  util.inherits(CustomLogger, winston.Transport);

  CustomLogger.prototype.log = function (level, msg, meta, callback) {
    //
    // Store this message and metadata, maybe use some custom logic
    // then callback indicating success.
    //
    callback(null, true);
  };
```

### Custom Log Format
To specify a custom log format for a transport, you should set a formatter function. Currently supported transports are: Console, File, Memory.
An options object will be passed to the formatter function. It's general properties are: timestamp, level, message, meta. Depending on the transport type, there may be additional properties.

``` js
var logger = new (winston.Logger)({
  transports: [
    new (winston.transports.Console)({
      timestamp: function() {
        return Date.now();
      },
      formatter: function(options) {
        // Return string will be passed to logger.
        return options.timestamp() +' '+ options.level.toUpperCase() +' '+ (options.message ? options.message : '') +
          (options.meta && Object.keys(options.meta).length ? '\n\t'+ JSON.stringify(options.meta) : '' );
      }
    })
  ]
});
logger.info('Data to log.');
```

### Inspirations
1. [npm][0]
2. [log.js][4]
3. [socket.io][5]
4. [node-rlog][6]
5. [BigBrother][7]
6. [Loggly][8]

## Installation

### Installing npm (node package manager)
```
  curl http://npmjs.org/install.sh | sh
```

### Installing winston
```
  [sudo] npm install winston
```

## Run Tests
All of the winston tests are written in [vows][9], and designed to be run with npm.

``` bash
  $ npm test
```

#### Author: [Charlie Robbins](http://twitter.com/indexzero)
#### Contributors: [Matthew Bergman](http://github.com/fotoverite), [Marak Squires](http://github.com/marak)

[0]: https://github.com/npm/npmlog/blob/master/log.js
[1]: http://nodejs.org/docs/v0.3.5/api/events.html#events.EventEmitter
[2]: http://github.com/nodejitsu/require-analyzer
[3]: http://nodejitsu.com
[4]: https://github.com/visionmedia/log.js
[5]: http://socket.io
[6]: https://github.com/jbrisbin/node-rlog
[7]: https://github.com/feisty/BigBrother
[8]: http://loggly.com
[9]: http://vowsjs.org
[10]: http://nodejs.org/api/util.html#util_util_format_format
[14]: http://nodejs.org/api/stream.html#stream_class_stream_writable
[16]: https://github.com/indexzero/winston-mongodb
[17]: https://github.com/indexzero/winston-riak
[18]: https://github.com/appsattic/winston-simpledb
[19]: https://github.com/wavded/winston-mail
[21]: https://github.com/jesseditson/winston-sns
[22]: https://github.com/flite/winston-graylog2
[23]: https://github.com/kenperkins/winston-papertrail
[24]: https://github.com/jorgebay/winston-cassandra
[25]: https://github.com/jesseditson/winston-sns
[26]: https://github.com/inspiredjw/winston-dynamodb/
