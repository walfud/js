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
moment().format('YYYY-MM-DD HH:mm:ss');               // local     2016-08-18 20:38:53
moment.utc().format('YYYY-MM-DD HH:mm:ss');           // utc       2016-08-18 12:38:53
moment().utcOffset(8).format('YYYY-MM-DD HH:mm:ss');  // specified 2016-08-18 20:38:53


moment.utc([2016, 3, 5]).format('YYYY-MM-DD HH:mm:ss');   // utc   2016-04-05 00:00:00 -> local 2016-04-05 08:00:00
moment([2016, 3, 5]).utc().format('YYYY-MM-DD HH:mm:ss'); // local 2016-04-05 00:00:00 -> utc   2016-04-04 16:00:00

// Set / Get
moment(1270451403123);
moment.unix(1270451403123.123);
moment([2010, 3, 5, 15, 10, 3, 123]);
moment({ years:'2010', months:'3', date:'5', hours:'15', minutes:'10', seconds:'3', milliseconds:'123'});  // from 2.10.7
moment().set({'year': 2013, 'month': 3});
moment().set('year', 2013);
moment().set('month', 3);  // April
moment().set('date', 1);
moment().set('hour', 13);
moment().set('minute', 20);
moment().set('second', 30);
moment().set('millisecond', 123);

moment().valueOf();
moment().unix();
moment().get('year');
moment().get('month');  // 0 to 11
moment().get('date');
moment().get('hour');
moment().get('minute');
moment().get('second');
moment().get('millisecond');

// Operation
moment().add(1, 'day');
moment().subtract(1, 'day');

// MISC.
moment().clone();

moment().isBefore(moment(1270451403123));
moment().isSameOrBefore(moment(1270451403123));
moment().isAfter(moment(1270451403123));
moment().isSameOrAfter(moment(1270451403123));
moment().isBetween(moment(1270451403123), moment(2270451403123));

moment().isLeapYear();
```
