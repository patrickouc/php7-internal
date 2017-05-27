## 7.7 zval的操作
扩展中经常会用到各种类型的zval，PHP提供了很多宏用于不同类型zval的操作，尽管我们也可以自己操作zval，但这并不是一个好习惯，因为zval有很多其它用途的标识，如果自己去管理这些值将是非常繁琐的一件事，所以我们应该使用PHP提供的这些宏来操作用到的zval。

本节提到的zval泛指zval及各种zend_value。

### 7.7.1 新生成各类型zval
PHP7将变量的引用计数转移到了具体的value上，所以zval更多的是作为统一的传输格式，很多情况下只是临时性使用，比如函数调用时的传参，最终需要的数据是zval携带的zend_value，函数从zval取得zend_value后就不再关心zval了，这种就可以直接在栈上分配zval。分配完zval后需要将其设置为我们需要的类型以及设置其zend_value，PHP中定义的`ZVAL_XXX()`系列宏就是用来干这个的，这些宏第一个参数z均为要设置的zval的指针，后面为要设置的zend_value。

* __ZVAL_UNDEF(z):__ 表示zval被销毁
* __ZVAL_NULL(z):__ 设置为NULL
* __ZVAL_FALSE(z):__ 设置为false
* __ZVAL_TRUE(z):__ 设置为true
* __ZVAL_BOOL(z, b):__ 设置为布尔型，b为IS_TRUE、IS_FALSE，与上面两个等价
* __ZVAL_LONG(z, l):__ 设置为整形，l类型为zend_long，如：`zval z; ZVAL_LONG(&z, 88);`
* __ZVAL_DOUBLE(z, d):__ 设置为浮点型，d类型为double
* __ZVAL_STR(z, s):__ 设置字符串，将z的value设置为s，s类型为zend_string*，不会增加s的refcount，支持interned strings
* __ZVAL_NEW_STR(z, s):__ 同ZVAL_STR(z, s)，s为普通字符串，不支持interned strings
* __ZVAL_STR_COPY(z, s):__ 将s拷贝到z的value，s类型为zend_string*，同ZVAL_STR(z, s)，这里会增加s的refcount
* __ZVAL_ARR(z, a):__ 设置为数组，a类型为zend_array*
* __ZVAL_NEW_ARR(z):__ 新分配一个数组，主动分配一个zend_array
* __ZVAL_NEW_PERSISTENT_ARR(z):__ 创建持久化数组，通过malloc分配，需要手动释放
* __ZVAL_OBJ(z, o):__ 设置为对象，o类型为zend_object*
* __ZVAL_RES(z, r):__ 设置为资源，r类型为zend_resource*
* __ZVAL_NEW_RES(z, h, p, t):__ 新创建一个资源，h为资源handle，t为type，p为资源ptr指向结构
* __ZVAL_REF(z, r):__ 设置为引用，r类型为zend_reference*
* __ZVAL_NEW_EMPTY_REF(z):__ 新创建一个空引用，没有设置具体引用的value
* __ZVAL_NEW_REF(z, r):__ 新创建一个引用，r为引用的值，类型为zval*
* ...

### 7.7.2 获取zval的值及类型
zval的类型通过`Z_TYPE(zval)`、`Z_TYPE_P(zval*)`两个宏获取，这个值取的就是`zval.u1.v.type`，但是设置时不要只修改这个type，而是要设置typeinfo，因为zval还有其它的标识需要设置，比如是否使用引用计数、是否可被垃圾回收、是否可被复制等等。

内核提供了`Z_XXX(zval)`、`Z_XXX_P(zval*)`系列的宏用于获取不同类型zval的value。

* __Z_LVAL(zval)、Z_LVAL_P(zval_p):__ 返回zend_long
* __Z_DVAL(zval)、Z_DVAL_P(zval_p):__ 返回double
* __Z_STR(zval)、Z_STR_P(zval_p):__ 返回zend_string*
* __Z_STRVAL(zval)、Z_STRVAL_P(zval_p):__ 返回char*，即：zend_string->val
* __Z_STRLEN(zval)、Z_STRLEN_P(zval_p):__ 获取字符串长度
* __Z_STRHASH(zval)、Z_STRHASH_P(zval_p):__ 获取字符串的哈希值
* __Z_ARR(zval)、Z_ARR_P(zval_p)、Z_ARRVAL(zval)、Z_ARRVAL_P(zval_p):__ 返回zend_array*
* __Z_OBJ(zval)、Z_OBJ_P(zval_p):__ 返回zend_object*
* __Z_OBJ_HT(zval)、Z_OBJ_HT_P(zval_p):__ 返回对象的zend_object_handlers，即zend_object->handlers
* __Z_OBJ_HANDLER(zval, hf)、Z_OBJ_HANDLER_P(zv_p, hf):__ 获取对象各操作的handler指针，hf为write_property、read_property等，注意：这个宏取到的为只读，不要试图修改这个值(如：Z_OBJ_HANDLER(obj, write_property) = xxx;)，因为对象的handlers成员前加了const修饰符
* __Z_OBJCE(zval)、Z_OBJCE_P(zval_p):__ 返回对象的zend_class_entry*
* __Z_OBJPROP(zval)、Z_OBJPROP_P(zval_p):__ 获取对象的成员数组
* __Z_RES(zval)、Z_RES_P(zval_p):__ 返回zend_resource*
* __Z_RES_HANDLE(zval)、Z_RES_HANDLE_P(zval_p):__ 返回资源handle
* __Z_RES_TYPE(zval)、Z_RES_TYPE_P(zval_p):__ 返回资源type
* __Z_RES_VAL(zval)、Z_RES_VAL_P(zval_p):__ 返回资源ptr
* __Z_REF(zval)、Z_REF_P(zval_p):__ 返回zend_reference*
* __Z_REFVAL(zval)、Z_REFVAL_P(zval_p):__ 返回引用的zval*

除了这些与PHP变量类型相关的宏之外，还有一些内核自己使用类型的宏：
```c
//获取indirect的zval，指向另一个zval
#define Z_INDIRECT(zval)            (zval).value.zv
#define Z_INDIRECT_P(zval_p)        Z_INDIRECT(*(zval_p))

#define Z_CE(zval)                  (zval).value.ce
#define Z_CE_P(zval_p)              Z_CE(*(zval_p))

#define Z_FUNC(zval)                (zval).value.func
#define Z_FUNC_P(zval_p)            Z_FUNC(*(zval_p))

#define Z_PTR(zval)                 (zval).value.ptr
#define Z_PTR_P(zval_p)             Z_PTR(*(zval_p))
```
zend_string常用的宏：
```c
//zstr类型为zend_string*
#define ZSTR_VAL(zstr)  (zstr)->val
#define ZSTR_LEN(zstr)  (zstr)->len
#define ZSTR_H(zstr)    (zstr)->h
#define ZSTR_HASH(zstr) zend_string_hash_val(zstr)
```
### 7.7.3 引用计数
```c
//获取引用数：pz类型为zval*
#define Z_REFCOUNT_P(pz)            zval_refcount_p(pz)
//设置引用数
#define Z_SET_REFCOUNT_P(pz, rc)    zval_set_refcount_p(pz, rc)
//增加引用
#define Z_ADDREF_P(pz)              zval_addref_p(pz)
//减少引用
#define Z_DELREF_P(pz)              zval_delref_p(pz)

#define Z_REFCOUNT(z)               Z_REFCOUNT_P(&(z))
#define Z_SET_REFCOUNT(z, rc)       Z_SET_REFCOUNT_P(&(z), rc)
#define Z_ADDREF(z)                 Z_ADDREF_P(&(z))
#define Z_DELREF(z)                 Z_DELREF_P(&(z))

//只对使用了引用计数的变量类型增加引用，建议使用这个
#define Z_TRY_ADDREF_P(pz) do {     \
    if (Z_REFCOUNTED_P((pz))) {     \
        Z_ADDREF_P((pz));           \
    }                               \
} while (0)

#define Z_TRY_DELREF_P(pz) do {     \
    if (Z_REFCOUNTED_P((pz))) {     \
        Z_DELREF_P((pz));           \
    }                               \
} while (0)

#define Z_TRY_ADDREF(z)             Z_TRY_ADDREF_P(&(z))
#define Z_TRY_DELREF(z)             Z_TRY_DELREF_P(&(z))
```

### 7.7.4 数组操作
