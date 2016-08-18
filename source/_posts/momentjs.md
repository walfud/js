---
title: momentjs
---

```java
import moment from 'moment';

//   YYYY/YY: year
//     MM/M : month, [1, 12]
//     DD/D : day,   [1, 31]
//     HH/H : hour,  [0, 23]
//     mm/m : minute
//     ss/s : second
//      SSS : millisecond
//     ZZ/Z : timezone, +08:00 / +0800
//
//      a/A : am,pm / AM,PM
//  DDDD/DDD: day of year,     [1, 365]
//        e : day of week,     [0, 6]
//        Q : quarter of year, [1, 4]
// MMMM/MMM : literal month,   [Jan, Dec] / [January, December]
// dddd/ddd : literal week,    [Sun, Sat] / [Sunday, Saturday]
//
// http://momentjs.com/docs/#/displaying/format/

// local      2016-08-18 20:38:53
moment().format('YYYY-MM-DD HH:mm:ss');
// utc        2016-08-18 12:38:53
moment.utc().format('YYYY-MM-DD HH:mm:ss');
// specified  2016-08-18 20:38:53
moment().utcOffset(8).format('YYYY-MM-DD HH:mm:ss');

// utc 2016-04-05 00:00:00   ->   local 2016-04-05 08:00:00
moment.utc([2016, 3, 5]).format('YYYY-MM-DD HH:mm:ss');
// local 2016-04-05 00:00:00   ->   utc 2016-04-04 16:00:00
moment([2016, 3, 5]).utc().format('YYYY-MM-DD HH:mm:ss');
```
