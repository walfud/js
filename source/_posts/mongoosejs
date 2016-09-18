---
title: mongoosejs 看这一篇就够了
tag: js, 看这一篇就够了
---

[Schema](http://mongoosejs.com/docs/schematypes.html)
```javascript
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

})
```
特别注意: 对于 `Date/Mixed` 类型, mongoose 无法追踪值得变更, 因此需要手动标记:
```javascript
const Assignment = mongoose.model('Assignment', { dueDate: Date });
Assignment.findOne(function (err, doc) {
    doc.dueDate.setMonth(3);
    doc.save(callback); // THIS DOES NOT SAVE YOUR CHANGE

    doc.markModified('dueDate');
    doc.save(callback); // works
})
```
