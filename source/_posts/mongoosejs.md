---
title: mongoosejs 看这一篇就够了
tag:
  - js
  - 看这一篇就够了
date: 2016-09-10 00:00:00
---

### 快速入门
`npm install mongoose`
```js
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/test');

const Cat = mongoose.model('Cat', { name: String });

// Model#save([options], [options.safe], [options.validateBeforeSave], [fn])
const kitty = new Cat(
    { name: 'walfud' }
);
kitty.save(function (err) {
    ...
});

// Query#find([criteria], [callback])
Cat.find(
    {},
    (err, doc) => {
    ...
});
// Query#findOne([callback])   --- 返回第一条 doc
// Query#findOne([criteria], [projection], [callback])
Cat.findOne((err, doc) => {
    ...
});

// Document#update(doc, options, callback)
kitty.update(
    { name: 'duflaw' },
    { w: 1 },
    (err, doc) => {
    ...
});
// Query#update([criteria], [doc], [options], [callback])
Cat.update(
    { name: 'walfud' },
    { name: 'duflaw' },
    (err, doc) => {
    ...
});

// Model#remove([fn])
kitty.remove((err, doc) => {
    ...
});
// Query#remove([criteria], [callback])
Cat.remove(
    { name: 'walfud' },
    (err, doc) => {
    ...
});
```

### [Schema](http://mongoosejs.com/docs/schematypes.html)

```js
const MySchema = new Schema({

    // 支持的类型
    str:    String,
    num:    Number,
    date:   Date,
    buffer: Buffer,
    bool:   Boolean,
    anything: Schema.Types.Mixed,
    id:       Schema.Types.ObjectId,

    // 数组和嵌套
    strings: [String],
    nested: {
        stuff: { type: String, lowercase: true, trim: true }
    },
    nested2: OtherSchema,


    // 限定
    // 通用
    general: {
        type: Schema.Types.Mixed,
        required: true,
        default: "default value",
        select: true, // 设置 `find` 的时候是否是默认的 projection
        validate: { // 自定义 validator
            validator: function(v, cb) {
                setTimeout(function() {
                    cb(/\d{3}-\d{3}-\d{4}/.test(v));
                }, 5);
            },
            message: '{VALUE} is not a valid phone number!'
        },
        required: [true, 'User phone number required']
    }
    // 针对 Number
    range: {
        type: Number,
        min: 18,
        max: 65
    },
    // 针对 String
    lower: {
        type: String,
        lowercase: true,
        uppercase: true,
        trim: true,
        match: /^...$/,
        enum: ["foo", "bar"]
    },


    // Getter/Setter
    integer: {
        type: Number,
        get: v => Math.round(v),
        set: v => Math.round(v),
    },


    // MISC.
    primary: {
        type: String,
        index: true,
        unique: true, // 如果指定了 `unique: true`, 则默认 `index: true`, 因此可以不指定 `index`.
    }

});
```

特别注意: 对于 `Date/Mixed` 类型, mongoose 无法追踪值得变更, 因此需要手动标记:

```js
const Assignment = mongoose.model('Assignment', { dueDate: Date });
Assignment.findOne((err, doc) => {
    doc.dueDate.setMonth(3);
    doc.save(callback); // THIS DOES NOT SAVE YOUR CHANGE

    doc.markModified('dueDate');
    doc.save(callback); // works
});
```

# [Model](http://mongoosejs.com/docs/models.html)

```js
const Tank = mongoose.model('Tank', yourSchema);

const small = new Tank({ size: 'small' });
small.save(function (err) {
    if (err) return handleError(err);
    // saved!
});

// or

Tank.create({ size: 'small' }, function (err, small) {
    if (err) return handleError(err);
    // saved!
});
```

注意事项: `model` 方法会复制 `schema` 对象, 因此, 一定要在调用 `.model()` 之前设置好 `schema` 对象!

# [CRUD](http://mongoosejs.com/docs/queries.html)

```js
const Person = mongoose.model('Person', yourSchema);

// find each person with a last name matching 'Ghost', selecting the `name` and `occupation` fields
Person.findOne({
        'name.last': 'Ghost'  // criteria, 查询条件, 与 mongo shell 一致
    },
    'name occupation',        // projection, 选择返回的列
    function(err, person) {   // callback
        if (err) return handleError(err);
        console.log('%s %s is a %s.', person.name.first, person.name.last, person.occupation) // Space Ghost is a talk show host.
    }
);

// or

// find each person with a last name matching 'Ghost'
const query = Person.findOne({
    'name.last': 'Ghost'
});

// selecting the `name` and `occupation` fields
query.select('name occupation');

// execute the query at a later time
query.exec(function(err, person) {
    if (err) return handleError(err);
    console.log('%s %s is a %s.', person.name.first, person.name.last, person.occupation) // Space Ghost is a talk show host.
});

// or 使用 cursor 进行 stream query

const cursor = Person.find({
    occupation: /host/
}).cursor();
cursor.on('data', function(doc) {
    // Called once for every document
});
cursor.on('close', function() {
    // Called when done
});
```

method  | return
------- | --------------------------
find    | [{}, {}, ...], document 列表
findOne | {}, 某个 document
update  | Number, 影响的行数
count   | Number, 行数

其中, 'criteria(查询条件)' 与 mongo shell 一致, 参考 [Operators](https://docs.mongodb.com/manual/reference/operator/).

### [Middleware(Hook)](http://mongoosejs.com/docs/middleware.html)
Middleware 分为两种:
* Document Middleware: `this` 引用是被更新的 document
  - init
  - validate
  - save
  - remove

* Query Middleware: `this` 引用是 query 对象
  - count
  - find
  - findOne
  - findOneAndRemove
  - findOneAndUpdate
  - insertMany
  - update

```js
// Pre
const schema = new Schema(..);
schema.pre('save', function(next) {
    // do stuff
    next();
});

schema.post('save', function(doc, next) {
    // do stuff
    next();
});
```

### [Plugin](http://mongoosejs.com/docs/plugins.html)
```js
// lastMod.js
module.exports = function lastModifiedPlugin(schema, options) {
    schema.add({
        lastMod: Date
    });

    schema.pre('save', function(next) {
        this.lastMod = new Date
        next()
    });
}

// game-schema.js
const lastMod = require('./lastMod');
const Game = new Schema({...
});
Game.plugin(lastMod);

// player-schema.js
const lastMod = require('./lastMod');
const Player = new Schema({...
});
Player.plugin(lastMod);

// 也可以一次性对所有 model 设置 plugin

const mongoose = require('mongoose');
mongoose.plugin(require('./lastMod'));

const gameSchema = new Schema({ ... });
const playerSchema = new Schema({ ... });
// `lastModifiedPlugin` gets attached to both schemas
const Game = mongoose.model('Game', gameSchema);
const Player = mongoose.model('Player', playerSchema);
```


### refs:
* [Mongodb Reference Cards](http://info-mongodb-com.s3.amazonaws.com/ReferenceCards15-PDF.pdf)
* [Node.js MongoDB Driver API](http://mongodb.github.io/node-mongodb-native/2.2/api/)
* [Mongoosejs](http://mongoosejs.com/index.html)
