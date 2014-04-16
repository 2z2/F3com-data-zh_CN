# Base

基本类体现了框架的核心。它包含了运行一个应用所需的大多功能,基本类文件 `base.php` 包含了基本且必要的 [Cache](cache), [Prefab](prefab-registry), [View](view), [ISO](iso) and [Registry](prefab-registry#registry) 类减少不必要的磁盘I/O开销从而达到最佳表现状态.

如果你需要的仅仅是这个包内的基础的特色功能,你可以随心所欲的移除`lib/`目录下的其他任何文件。

命名空间: `\` <br/>
文件位置: `lib/base.php`

---

## The Hive

`hive`是以`键/值`的形式存储框架中变量的内存数组,存储在`hive`的值确保了变量可以在应用中的所有类和方法是用。

### set

**将值绑定在 `hive` 的一个键上,形成键/值对**

```php
mixed set ( string $key, mixed $val [, int $ttl = 0 ] )
```
*示例：设置框架变量:*

```php
$f3->set('a',123); // a=123, integer
$f3->set('b','c'); // b='c', string
$f3->set('c','whatever'); // c='whatever', string
$f3->set('d',TRUE); // d=TRUE, boolean
```
*示例:设置数组*

```php
$f3->set('hash',array( 'x'=>1,'y'=>2,'z'=>3 ) );
// dot notation is also possible:
$f3->set('hash.x',1);
$f3->set('hash.y',2);
$f3->set('hash.z',3);
```
*示例:设置对象*

```php
$f3->set('a',new \stdClass);
$f3->set('a->hello','world');
echo $f3->get('a')->hello; // world
```

#### Caching properties

如果 `$ttl` 参数是大于0的( > 0 ), 并且[framework cache engine](cache)开启, 指定的变量将会被缓存 `$ttl` 秒,只有获得新的到期`$ttl`时已经缓存的变量才会被更新。
如果该`key`在之前已经被缓存了并且`$ttl`设置的是0, 这个键值在`cache`也会被更新, 方式是恢复旧的过期时间。

你可以缓存字符串,数组或者其他类型,甚至是已经完成的对象,`get()`方法会从`cache`中自动装载他们。

*示例:缓存框架变量*

```php
// cache string
$f3->set('simplevar','foo bar 1337', 3600); // cache for 1 hour
//cache big computed arrays
$f3->set('fruits',array(
    'apple',
    'banana',
    'peach',
), 3600);
// cache objects
$f3->set('myClass1', new myClass('arg1'), 3600);
// change expire time for a single cookie var
$f3->set('COOKIE.foo', 123, 3600);  // 1 hour
$f3->set('COOKIE.bar', 456, 86400); // 1 day
```

#### System variables

框架已经拥有一些自己的系统变量[system variables] (quick-reference#system-variables) ,你可以修改他们在改变框架的行为表现, 例如:

```php
$f3->set('CACHE', TRUE);
$f3->set('HALT', FALSE);
$f3->set('CASELESS', FALSE);
```

用F3框架是可以设置PHP的全局变量的[COOKIE, GET, POST, REQUEST, SESSION, FILES, SERVER, ENV system variables](quick-reference#cookie,-get,-post,-request,-session,-files,-server,-env),这8个变量会与相关的PHP全局变量同步。

<div class="alert alert-info"><strong>提示:</strong> 如果你设置了或者访问了SESSION的键, 那么这个session会自动开始而不需要你手动去开始他.</div>

**谨记**: `Hive`的键是非常敏感的,<br/>所以,根`hive`键会对如下允许的字符:[&nbsp;a-z&nbsp;A-Z&nbsp;0-9&nbsp;_&nbsp;]进行有效性检查。

### get

**通过`hive`键取回对应的值**

```php
mixed get ( string $key [, string|array $args = NULL ] )
```

预获取之前已经存储的框架变量,接着看:

```php
$f3->set('myVar','hello world');
echo $f3->get('myVar'); // 输出字符串 'hello world'
$local_var = $f3->get('myVar'); // 变量$local_var 已经拿到了 'hello world'
```

<div class="alert alert-info"><strong>提示:</strong> 当缓存开启并且变量并未在运行时被定义,使用get()方法时,F3会尝试从Cache中装载数据.</div>

访问数组尤为方便. 你也可以使用JS.记号法形如`'myarray.bar'`,这样更易于阅读和写.
<!-- special alignment ez 2 read 4 beginners -->

```php
$f3->set('myarray',
           array(
                    0 => 'value_0',
                    1 => 'value_1',
                'bar' => 123,
                'foo' => 'we like candy',
                'baz' => 4.56,
                )
         );

echo $f3->get('myarray[0]'); // value_0
echo $f3->get('myarray.1'); // value_1
echo $f3->get('myarray.bar'); // 123
echo $f3->get('myarray["foo"]'); // aha,我们爱语法糖
echo $f3->get('myarray[baz]'); // 4.56, notice alternate use of single, double and no quotes
```

### sync

**同步相应的`hive`键值到PHP全局变量**

```php
array sync ( string $key )
```

Usage:

```php
$f3->sync('SESSION'); // 确保PHP全局变量 `SESSION` 和F3变量的 `SESSION`相同
```

<div class="alert alert-info">
    F3框架会自动同步如下PHP全局变量:
    <b>GET</b>, <b>POST</b>, <b>COOKIE</b>, <b>REQUEST</b>, <b>SESSION</b>, <b>FILES</b>, <b>SERVER</b>, <b>ENV</b>
</div>



### ref

**通过`hive`键得到它的参照以及内容**

```php
mixed &ref ( string $key [, bool $add = true ] )
```

Usage:

```php
$f3->set('name','John');
$b = &$f3->ref('name'); // $b是框架变量`name`的参照变量,而不是拷贝
$b = 'Chuck'; // modifiying the reference updates the framework variable 'name'
echo $f3->get('name'); // Chuck
```

你也添加当前不存在的`hive`键,如数组元素,对象属性。第二个参数默认是`TRUE`.

```php

$new = &$f3->ref('newVar'); // creates new framework hive var 'newVar' and returns reference to it
$new = 'hello world'; // set value of php variable, also updates reference
echo $f3->get('newVar'); // hello world

$new = &$f3->ref('newObj->name');
$new = 'Sheldon';
echo $f3->get('newObj')->name; // Sheldon
echo $f3->get('newObj->name'); // Sheldon
echo $f3->get('newObj.name'); // Sheldon

$a = &$f3->ref('hero.name');
$a = 'SpongeBob';
// or
$b = &$f3->ref('hero'); // variable
$b['name'] = 'SpongeBob';  // becomes array with key 'name'
$my_array = $f3->get('hero');
echo $my_array['name']; // 'SpongeBob'
```

如果第二个参数 `$add` 是 `false`, 它只会返回给你`只读`的hive key的内容,这种行为仅仅类似调用来get()方法,如果hive key不存在,返回NULL.

### exists

**如果key并未设置，返回`TRUE`反之返回已经已缓存key的时间戳和TTL**

```php
bool exists ( string $key [, mixed &$val=NULL] )
```

`exists` 方法在并未在`hive`中找到该key的情况下进行Cache的后端存储检查，如果缓存中找到了key则会直接返回`array ( $timestamp, $ttl )`.

<div class="alert alert-info"><strong>提示:</strong> `exists`方法是用来PHP中的`isset()`方法来判断`hive`key是否设置以及其是否为空.</div>

Usage:

```php
$f3->set('foo','value');

$f3->exists('foo'); // true
$f3->exists('bar'); // false, was not set above
```

`exists` 配合 [PHP global variables automatically synched by F3](base#sync)使用特别方便:

```php
// 已经与`hive`同步过的PHP全局变量
$f3->exists('COOKIE.userid');
$f3->exists('SESSION.login');
$f3->exists('POST.submit');
```

<div class="alert alert-warning"><strong>提示:</strong> 如果你检查SESSION 键是否存在时，该session会自动启动.</div>

### devoid

**如果`hive`的key为空且没有缓存，则返回TRUE**

```php
bool devoid ( string $key )
```

如果`hive`中没有找到该key,`devoid`方法会检查后台缓存存储.

<div class="alert alert-info"><strong>提示:</strong> 'devoid'使用了PHP中的empty()方法来判断一个hive key是否为空以及是否缓存.</div>

Usage:

``` php
$f3->set('foo','');
$f3->set('bar',array());
$f3->set('baz',array(),10);

$f3->devoid('foo'); // true
$f3->devoid('bar'); // true
$f3->devoid('baz'); // false
```

### clear

**注销`hive`键值，该键不再存在**

```php
void clear ( string $key )
```

完全从内存中移除一个hive key以及他的值:

```php
$f3->clear('foobar');
$f3->clear('myArray.param1'); // removes key `param1` from array `myArray`
```

如果给出的参数key之前已经被缓存过，该方法也会从缓存中一并移除.

Some more special usages:

```php
$f3->clear('SESSION'); // destroys the user SESSION
$f3->clear('COOKIE.foobar'); // removes a cookie
$f3->clear('CACHE'); // clears all cache contents
```

<div class="alert alert-info"><strong>提示:</strong> 后端XCache缓存不支持一次性清除所有缓存内容</div>

### mset

**使用关联数组设置多个变量**

```php
void mset ( array $vars [, string $prefix = '' [, integer $ttl = 0 ]] )
```

Usage:

```php
$f3->mset(
    array(
        'var1'=>'value1',
        'var2'=>'value2',
        'var3'=>'value3',
    )
);

echo $f3->get('var1'); // value1
echo $f3->get('var2'); // value2
echo $f3->get('var3'); // value3
```

你可以设置第二个参数`$prefix`来统一增加所有变量的前缀.

```php
$f3->mset(
    array(
        'var1'=>'value1',
        'var2'=>'value2',
        'var3'=>'value3',
    ),
    'pre_'
);

echo $f3->get('pre_var1'); // value1
echo $f3->get('pre_var2'); // value2
echo $f3->get('pre_var3'); // value3
```

如果你想缓存这些变量，直接设定第三个变量`$ttl`即可，约定该参数为正整数，单位是秒.

### hive

**以数组形式返回所有`hive`内容**

```php
array hive ()
```

Usage:

```php
printf ("A Busy Hive: <pre>%s</pre>", var_export( $f3->hive(), true ) );
```

### copy

**拷贝`hive`一变量的内容到另外一个变量上**

```php
mixed copy ( string $src, string $dst )
```

返回一个可写的引用给`$dst`hive键,如果`$dst`已经存在于hive中，该方法将会毫不犹豫的覆盖它.

Usage:

```php
$f3->set('foo','123');
$f3->set('bar','barbar');
$bar = $f3->copy('foo','bar'); // bar = '123'
$bar = 456;
$f3->set('foo','789');
echo $f3->get('bar'); // '456'
```

### concat

**连接字符串并存为`hive`字符串变量**

```php
string concat ( string $key, string $val )
```

返回拼接的结果. **提示**: 如果`$key`在`hive`并不存在，该方法将会自动在`hive`中自动创建.

Usage:

```php
$f3->set('count', 99);
$f3->set('item', 'beer');
$text = $f3->concat('count', ' bottles of '.$f3->get('item'));
$text .= ' on the wall';
$f3->concat('wall', $f3->get('count')); // new 'wall' hive key is created
echo $f3->get('wall'); // 99 bottles of beer on the wall
```

### flip

**交换`hive`中数组变量的键和值**

```php
array flip ( string $key )
```

Usage:

```php
$f3->set('data', array(
    'foo1' => 'bar1',
    'foo2 '=> 'bar2',
    'foo3' => 'bar3',
));
$f3->flip('data');

print_r($f3->get('data'));
/* output:
Array
(
    [bar1] => foo1
    [bar2] => foo2
    [bar3] => foo3
)
*/
```

### push

**在`hive`数组变量的尾部(最后面)添加一个元素**

```php
mixed push ( string $key, mixed $val )
```

Usage:

```php
$f3->set('fruits',array(
    'apple',
    'banana',
    'peach',
));
$f3->push('fruits','cherry');

print_r($f3->get('fruits'));
/* output:
Array
(
    [0] => apple
    [1] => banana
    [2] => peach
    [3] => cherry
)
*/
```

### pop

**从`hive`数组变量中移除最后一个元素**

```php
mixed pop ( string $key )
```

Usage:

```php
$f3->set('fruits',array(
	'apple',
	'banana',
	'peach'
));
$f3->pop('fruits'); // returns "peach"

print_r($f3->get('fruits'));
/* output:
Array
(
    [1] => apple
    [2] => banana
)
*/
```

### unshift

**在`hive`数组变量的头部(最开始)添加一个元素**

```php
mixed unshift ( string $key, string $val )
```

Usage:

```php
$f3->set('fruits',array(
	'apple',
	'banana',
	'peach'
));
$f3->unshift('fruits','cherry');

print_r($f3->get('fruits'));
/* output:
Array
(
    [0] => cherry
    [1] => apple
    [2] => banana
    [3] => peach
)
*/
```

### shift

**从`hive`数组变量中移除第一个元素**

```php
array|NULL shift ( string $key )
```

返回左移的`hive`数组变量或者当数组变量已经为空或不是一个数组时返回`NULL`. 

<div class="alert alert-warning">
<b>提示</b>: <code>shift</code> use the PHP function <code>array_shift()</code>. It means that all numerical array keys of the hive array variable will be modified to start counting from zero while literal keys won't be touched
</div>

Example:

```php
$f3->set('fruits', array(
    'crunchy'=>'apples',
    '11'=>'bananas',
    '6'=>'kiwis',
    'juicy'=>'peaches'
));
$f3->shift('fruits'); // returns "apples"
print_r($f3->get('fruits'));
/* output:
Array
(
    [0] => bananas
    [1] => kiwis
    [juicy] => peaches
)
*/
```

### merge

**Merge array with hive array variable**

```php
array merge ( string $key, array $src )
```

Return the resulting array of the merge. (Does not touch the value of the hive key)

Example:

``` php
$f3->set('foo', array('blue','green'));
$bar = $f3->merge('foo', array('red', 'yellow'));

/* $bar now is:
array (size=4)
  [0] => string 'blue' (length=4)
  [1] => string 'green' (length=5)
  [2] => string 'red' (length=3)
  [3] => string 'yellow' (length=6)
*/
```

---

## Encoding & Conversion

### fixslashes

**Convert backslashes to slashes**

```php
string fixslashes ( string $str )
```

Usage:

```php
$filepath = __FILE__; // \www\mysite\myfile.txt
$filepath = $f3->fixslashes($filepath); // /www/mysite/myfile.txt
```

### split

**Split comma-, semi-colon, or pipe-separated string**

```php
array split ( string $str )
```

Usage:

```php
$data = 'value1,value2;value3|value4';
print_r($f3->split($data));
/* output:
Array
(
    [0] => value1
    [1] => value2
    [2] => value3
    [3] => value4
)
*/
```

### stringify

**Convert PHP expression/value to compressed exportable string**

```php
string stringify ( mixed $arg [, array $stack = NULL ] )
```

This function allows you to convert any PHP expression, value, array or any object to a compressed and exportable string.

The `$detail` parameter controls whether to walk recursively into nested objects or not.

Example with a simple 2D array:

```php
$elements = array('water','earth','wind','fire');
$fireworks = array($elements, shuffle($elements), array_flip($elements));
echo $f3->csv($fireworks);
// Outputs e.g.:
'array('water','earth','wind','fire'),true,array('earth'=>0,'fire'=>1,'water'=>2,'wind'=>3)'
```

```php
$car = new stdClass();
$car->color = 'green';
$car->location = array('35.360496','138.727798');
echo $f3->stringify($car);
// Outputs e.g.:
'stdClass::__set_state('color'=>'green','location'=>array('35.360496','138.727798'))'
```

### csv

**Flatten array values and return as CSV string**

```php
string csv ( array $args )
```

Usage:

```php
$elements = array('water','earth','wind','fire');
echo $f3->csv($elements); // displays 'water','earth','wind','fire' // including single quotes
```

### camelcase

**Convert snake_case string to camelCase**

```php
string camelcase ( string $str )
```

Usage:

```php
$str_s_c = 'user_name';
$f3->camelcase($str_s_c); // returns 'userName'
```
### snakecase

**Convert camelCase string to snake_case**

```php
string snakecase ( string $str )
```

Usage:

```php
$str_CC = 'userName';
$f3->snakecase($str_CC); // returns 'user_name'
```

### sign

**Return -1 if specified number is negative, 0 if zero,	or 1 if the number is positive**

```php
int sign ( mixed $num )
```

### hash

**Generate 64bit/base36 hash**

```php
string hash ( string $str )
```

Generates a 11-characters length hash for a given string

Example:

```php
$f3->hash('foobar'); // returns '0i43fmgps1r' (length=11)
```

### base64

**Return Base64-encoded equivalent**

```php
string base64 ( string $data, string $mime )
```

Example:

```php
echo $f3->base64('<h1>foobar</h1>','text/html');
// data:text/html;base64,PGgxPmZvb2JhcjwvaDE+
```

### encode

**Convert special characters to HTML entities**

```php
string encode ( string $str )
```

Encodes symbols like `& " ' < >` and other chars, based on your applications ENCODING setting. (default: UTF-8)

Example:

```php
echo $f3->encode("we <b>want</b> 'sugar & candy'");
// we &amp;lt;b&amp;gt;want&amp;lt;/b&amp;gt; 'sugar &amp;amp; candy'

echo $f3->encode("§9: convert symbols & umlauts like ä ü ö");
// &amp;sect;9: convert symbols &amp;amp; umlauts like &amp;auml; &amp;uuml; &amp;ouml;
```

### decode

**Convert HTML entities back to characters**

```php
string decode ( string $str )
```

Example:

```php
echo $f3->decode("we &amp;lt;b&amp;gt;want&amp;lt;/b&amp;gt; 'sugar &amp;amp; candy'");
// we <b>want</b> 'sugar &amp; candy'

echo $f3->decode("&amp;sect;9: convert symbols &amp;amp; umlauts like &amp;auml; &amp;uuml; &amp;ouml;");
// §9: convert symbols &amp; umlauts like ä ü ö welcome!
```

### clean

**Remove HTML tags (except those enumerated) and non-printable characters to mitigate XSS/code injection attacks**

```php
string clean ( mixed $var [, string $tags = NULL ] )
```
`$var` can be either a `string` or an `array`. In the latter case, it will be recursively traversed to clean each and every element.

`$tags` defines a list [<small>(as per the split syntax)</small>](base#split) of _allowed_ html tags that will **not** be removed.

<div class="alert alert-success"><strong>Advice</strong>: It is recommended to use this function to sanitize submitted form input.</div>

Examples:

```php
$foo = "99 bottles of <b>beer</b> on the wall. <script>alert('scripty!')</script>";
echo $f3->clean($foo); // "99 bottles of beer on the wall. alert('scripty!')"
```

```php
$foo = "<h1><b>nice</b> <span>news</span> article <em>headline</em></h1>";
$h1 = $f3->clean($foo,'h1,span'); // <h1>nice <span>news</span> article headline</h1>
```

### scrub

**Similar to clean(), except that variable is passed by reference**

```php
string scrub ( mixed &$var [, string $tags = NULL ] )
```

Example:

```php
$foo = "99 bottles of <b>beer</b> on the wall. <script>alert('scripty!')</script>";
$bar = $f3->scrub($foo);
echo $foo; // 99 bottles of beer on the wall. alert('scripty!')
```

### serialize

**Return string representation of PHP value**

```php
string serialize ( mixed $arg )
```

Usage:

```php
// example using json_encode
$myArray = array('a' => 1, 'b' => 2, 'c' => 3, 'd' => 4, 'e' => 5);
echo $f3->serialize($myArray);  // outputs {"a":1,"b":2,"c":3,"d":4,"e":5}
```

Depending on the [SERIALIZER](quick-reference#serializer "Definition and usage of the SERIALIZER system variable") system variable, this method converts anything into a portable string expression. Possible values are **igbinary**, **json** and **php**.
F3 checks on startup if igbinary is available and prioritize it, as the igbinary extension works much faster and uses less disc space for serializing. Check [igbinary on github](https://github.com/igbinary/igbinary "Igbinary is a drop in replacement for the standard php serializer").

### unserialize

**Return PHP value derived from string**

```php
string unserialize ( mixed $arg )
```

See [serialize](base#serialize) for further description.

---

## Localisation

<div class="alert alert-info"><strong>Note:</strong> F3 follows <a href="http://userguide.icu-project.org/formatparse/" target="_blank">ICU formatting rules</a> <b>without</b> requiring the PHP's <tt>intl</tt> module.</div>

### format

**Return locale-aware formatted string**

```php
string format( string $format [, mixed $arg0 [, mixed $arg1...]] )
```

The `$format` string contains one or more placeholders identified by a position index enclosed in curly braces, the starting index being 0.
The placeholders are replaced by the values of the provided arguments.

```php
echo $f3->format('Name: {0} - Age: {1}','John',23); //outputs the string 'Name: John - Age: 23'
```

The formatting can get more precise if the expected type is provided within placeholders.

Current supported types are:

* date
* date,long
* date,custom
* time
* time,custom
* number,integer
* number,currency
* number,percent
* number,decimal
* plural

```php
echo $f3->format('Current date: {0,date} - Current time: {0,time}',time());
//outputs the string 'Current date: 04/12/2013 - Current time: 11:49:57'
echo $f3->format('Created on: {0,date,custom,%A, week: %V });
//outputs the string 'Created on: Monday, week 45'
echo $f3->format('{0} is displayed as a decimal number while {0,number,integer} is rounded',12.54);
//outputs the string '12.54 is displayed as a decimal number while 13 is rounded'
echo $f3->format('Price: {0,number,currency}',29.90);
//outputs the string 'Price: $29.90'
echo $f3->format('Percentage: {0,number,percent}',0.175);
//outputs the string 'Percentage: 18%' //Note that the percentage is rendered as an integer
echo $f3->format('Decimal Number: {0,number,decimal,2}', 0.171231235);
//outputs the string 'Decimal Number: 0,17'
```

The **plural** type syntax is a little bit less straightforward since it allows you to associate a different output depending on the input quantity. 

The plural type syntax must start with `0, plural,` followed by a list of plural keywords associated with the desired output. The accepted keywords are '*zero*', '*one*', '*two*' and '*other*'. 

Furthermore, you can insert the matching numeral in your output strings thanks to the `#` sign that will be automatically replaced by the matching number, as illustrated in the example below: 

```php
$cart_dialogs= '{0, plural,'.
	'zero	{Your cart is empty.},'.
	'one	{One (#) item in your cart.},'.
	'two	{A pair of items in your cart.},'.
	'other	{There are # items in your cart.}
}';

echo $f3->format($cart_dialogs,0); // displays 'Your cart is empty.'
echo $f3->format($cart_dialogs,1); // displays 'One (1) item in your cart.'
echo $f3->format($cart_dialogs,2); // displays 'A pair of items in your cart.'
echo $f3->format($cart_dialogs,3); // displays 'There are 3 items in your cart.'
```
Each plural keyword is optional and you can for example omit the plural keyword 'two' if the 'other' one fits that case. Of course, if you omit'em all, only the numerals will be displayed. As a general rule, keep at least the 'other' plural keyword as a fallback.

**Automatic Pluralization of Hive variables**

<div class="alert alert-info"><strong>Nice to Remember:</strong> F3, for your convenience, and to tremendously ease the use of pluralization in your templates, automatically formats variables from the Hive as long as they have a valid plural formatted string attached to them when they have been `set`.</div>

Example:

```php
$f3->set('a_books_story', 
	'{0, plural,'. 
		'zero	{There is n#thing on the table.},'. 
		'one	{A book is on the table.},'. 
		'two	{Two (#) books are on this table.},'. 
		'other	{There are # books on the table.}'. 
	'}' 
); 
echo $f3->get('a_books_story',0); // displays 'There is n0thing on the table.'
echo $f3->get('a_books_story',1); // displays 'A book is on the table.'
echo $f3->get('a_books_story',2); // displays 'Two (2) books are on this table.'
echo $f3->get('a_books_story',7); // displays 'There are 7 books on the table.'
```

### language

**Assign/auto-detect language**

```php
string language ( string $code )
```

This function is used while booting the framework to auto-detect the possible user language, by looking at the HTTP request headers, specifically the Accept-Language header sent by the browser.

Use the [LANGUAGE](quick-reference#language "Definition and usage of the LANGUAGE system variable") system variable to get and set languages, since it also handles dependencies like setting the locales using [php setlocale(LC_ALL,...)](http://php.net/manual/en/function.setlocale.php) and changing dictionary files.
The  [FALLBACK](quick-reference#fallback "Definition and usage of the FALLBACK system variable") system variable defines a default language, that will be used, if none of the detected languages are available as a dictionary file.

Example:

```php
$f3->get('LANGUAGE'); // 'de-DE,de,en-US,en'
$f3->set('LANGUAGE', 'es-BR,es');
$f3->get('LANGUAGE'); // 'es-BR,es,en' the fallback language is added at the end of the list
```

### lexicon

**Transfer lexicon entries to hive**

```php
array lexicon ( string $path )
```

This function is used while booting the framework to auto-load the dictionary files, located within the defined path in `LOCALES` var.

```php
$f3->set('LOCALES','dict/');
```

A dictionary file can be a php file returning a key-value paired associative array, or an .ini-style formatted config file.
[Read the guide about language files here](views-and-templates#multilingual-support).

---

## Routing

### build

**Replace tokenized URL with current route's token values**

``` php
string build ( string $url )
```

Example:

``` php
// for instance the route is '/subscribe/@channel' with PARAMS.channel = 'fatfree'

echo $f3->build('@channel'); // displays 'fatfree'
echo $f3->build('/get-it/now/@channel'); // displays '/get-it/now/fatfree'
echo $f3->build('/subscribe/@channel');  // displays '/subscribe/fatfree'
```

### mock

**Mock an HTTP request**

```php
NULL mock ( string $pattern [, array $args = NULL [, array $headers = NULL [, string $body = NULL ]]] )
```

This emulates a HTTP request.

Basic usage example:

```php
$f3->mock('GET /page/view');
```

### parse

**Parse a string containing key-value pairs and use them as routing tokens**

``` php
NULL parse ( string $str )
```

Example:

```php
$f3->parse('framework=f3 , speed=fast , features=full');
echo $f3->get('PARAMS.framework');  // 'f3'
echo $f3->get('PARAMS.speed');      // 'fast'
```

### route

**Bind a route pattern to a given handler**

```php
null route ( string|array $pattern, callback $handler [, int $ttl = 0 [, int $kbps = 0 ]] )
```

Basic usage example:

```php
$f3->route('POST /login','AuthController->login');
```

#### Route Pattern

The `$pattern` var describes a route pattern, that consists of the request type(s) and a request URI, both separated by a space char.

##### Verbs

Possible request type (Verb) definitions, that F3 will process, are: **GET**, **POST**, **PUT**, **DELETE**, **HEAD**, **PATCH**, **CONNECT**.

You can combine multiple verbs, to use the same route handler for all of them. Simply separate them by a pipe char, like `GET|POST`.

##### Tokens

The request URI may contain one or more **token(s)**, that a meant for defining dynamic routes. Tokens are indicated by a `@` char prior their name. See this example:

```php
$f3->route('GET|HEAD /@page','PageController->display'); // ex: /about
$f3->route('POST /@category/@thread','ForumThread->saveAnswer'); // /games/battlefield3
$f3->route('GET /image/@width-@height/@file','ImageCompressor->render'); // /image/300-200/mario.jpg
```

After processing the incoming request URI (initiated by [run](base#run)), you'll find the value of each of those tokens in the `PARAMS` system variable as named key, like `$f3->get('PARAMS.file')`. // 'mario.jpg'

<div class="alert alert-info">
    <b>Notice:</b> Routes and their according verbs are grouped by their URL pattern. Static routes _precede_ routes with dynamic tokens or wildcards.
    This means that having a static route, which overloads a matching dynamic route, requires you to define separately all required VERB patterns to this specific route. 
</div>

##### Wildcards

You can also define wildcards (`/*`) in your routing URI. Furthermore, you can use them in combination with `@` tokens.

```php
$f3->route('GET /path/*/@page', function($f3,$params) {
    // called URI: "/path/cat/page1"
    // $params is the same as $f3->get('PARAMS');
    $params[0]; // contains the full route path. "/path/cat/page"
    $params[1]; // and further numeric keys in PARAMS hold the catched wildcard paths and tokens. in this case "cat"
    $params['page']; // is your last segment, in this case "page1"
});
```

<div class="alert alert-info">
    <b>Notice:</b> The `PARAMS` var contains all tokens as named key, and additionally all tokens and wildcards as numeric key, depending on their order of appearance.
</div>

The route above also works with sub categories. Just call `/path/cat/subcat/page1`

```php
$params[1]; // now contains "/cat/subcat"
```

It even works with some more sub-levels. You just need to explode this value with a `/`-delimiter to handle your sub-categories.
Something like `/path/*/@pagetitle/@pagenum` is also quite easy.

It becomes complicated when you try to use more than one wildcard, because only the first `/*`-wildcard can hold unlimited path-segments.
Any further wildcards can only contain exactly one part between the slashes (`/`). So try to keep it simple.

##### Groups

It's possible to assign multiple routes to the same route handler, using an array of routes in `$pattern`. It would look like this:

```php
$f3->route(
  array(
    'GET /archive',
    'GET /archive/@year',
    'GET /archive/@year/@month',
    'GET /archive/@year/@month/@day'
  ),
  function($f3,$params){
	$params+=array('year'=>2013,'month'=>1,'day'=>1); //default values
    //etc..
  }
);
```

##### Named Routes

Since `F3 v3.2.0` you may also assign a name to your routes. Therefore follow this pattern:

```php
$f3->route('GET @beer_list: /beer', 'Beer->list');
```

Names are inserted after the route VERB and preceded by an @ symbol. All named route can be accessed by the [ALIASES](quick-reference#aliases) for further processing in templates or for rerouting.
Check out the User Guide about [creating named routes](routing-engine#named-routes) for additional information.

#### Route Handler

Can be a callable class method like 'Foo->bar' or 'Foo::bar', a function name, or an anonymous function.

F3 automatically passes the framework object to methods of route handler controller classes, i.e.

```php
$f3->set('hello','world');
$f3->route('GET /foo/@file','Bar->baz');

class Bar {
    function baz($f3,$args) {
        echo $f3->get('hello');
        echo $args['file'];
    }
}
```

#### Caching

The 3rd argument `$ttl` defines the caching time in seconds. Setting this argument to a positive value will call the [expire](base#expire) function to set cache metadata in the HTTP response header. It also caches the route response, but only GET and HEAD requests are cacheable.

If `CACHE` is turned off, `$ttl` will only control the browser cache using [expire](base#expire) header metadata.
If `CACHE` is turned on, and there is a positive `$ttl` value set for the current request URI handler, F3 additionally will cache the output for this page, and refresh it when `$ttl` expires.
Read more about it [here](https://groups.google.com/d/msg/f3-framework/lwaqZjtwCvU/PC-gK8Ki9PMJ) and [here](https://groups.google.com/d/msg/f3-framework/lwaqZjtwCvU/LDUlPhQfc84J).

#### Bandwidth Throttling

Set the 4th argument `$kbps` to your desired speed limit, to enable throttling. [Read more](optimization#bandwidth-throttling) about it in the user guide.

### reroute

**Reroute to specified URI**

```php
null reroute ( string $uri [, bool $permanent = FALSE] )
```

Examples of usage:

```php
// an old page is moved permanently
$f3->route('GET|HEAD /obsoletepage', function($f3) {
    $f3->reroute('/newpage', true);
});

// whereas a Post/Redirect/Get pattern would just redirect temporarily
$f3->route('GET|HEAD /login', 'AuthController->viewLoginForm');
$f3->route('POST /login', function($f3) {
    if( AuthController->checkLogin == true )
        $f3->reroute('/members', false);
    else
        $f3->reroute('/login', false);
});

// we can also reroute to external URLs
$f3->route('GET /partners', function($f3) {
    $f3->reroute('http://externaldomain.com');
});

// it's also possible to reroute to named routes
$f3->route('GET @beer_list: /beer', 'Beer->list');
$f3->route('GET /old-beer-page', function($f3) {
    $f3->reroute('@beer_list');
});

// even with dynamic parameter in your named route
$f3->route('GET @beer_producers: /beer/@country/@village', 'Beer->byproducer');
$f3->route('GET /old-beer-page', function($f3) {
    $f3->reroute('@beer_producers(@country=Germany,@village=Rhine)');
});
```

### map

**Provide ReST interface by mapping HTTP verb to class method**

```php
null map ( string $url, string $class [, int $ttl = 0 [, int $kbps = 0 ]] )
```

Its syntax works slightly similar to the **route** function, but you need not to define a HTTP request method in the 1st argument,
because they are mapped as functions in the Class you prodive in the `$class` argument.
[Read more about the REST support in the User Guide](routing-engine#rest-representational-state-transfer).

Example of usage:

```php
$f3->map('/news/@item','News');

class News {
    function get() {}
    function post() {}
    function put() {}
    function delete() {}
}
```

### run

**Match routes against incoming URI and call their route handler**

```php
null run ()
```

Example of usage:

```php
$f3 = require __DIR__.'/lib/base.php';
$f3->route('GET /',function(){
    echo "Hello World";
});
$f3->run();
```

After processing the incoming request URI, the routing pattern that matches that URI is saved in the `PATTERN` var, the current HTTP request URI in the `URI` var and the request method in the `VERB` var.
The `PARAMS` var will contains all tokens as named keys, and additionally all tokens and wildcards as numeric keys, depending on their order of appearance.

Generate a 404 error when a tokenized class doesn't exist.

<div class="alert alert-info">
	<p><b>Notice:</b> If a static and dynamic route pattern both match the current URI, then the <em>static</em> route pattern has priority.</p>
</div>


### call

**Execute callback/hooks (supports 'class->method' format)**

```php
mixed|false call ( callback $func [, mixed $args = NULL [, string $hooks = '' ]] )
```

This method provides that facility to invoke callbacks and their arguments. F3 recognizes these as valid callbacks:

* Anonymous/lambda functions (aka closures)
* `array('class','method')`
* `class::method` static method
* `class->method` equivalent to `array(new class,method)`

`$args` if specified provides a means of executing the callback with parameters.

`$hooks` is used by the `route()` method to specify pre- and post-execution functions, i.e. `beforeroute()` and `afterroute()`. (refer to the section [Event Handlers](routing-engine#event-handlers) for more explanations about `beforeroute()` and `afterroute()`)

### chain

**Execute specified callbacks in succession; Apply same arguments to all callbacks**

```php
array chain ( array|string $funcs [, mixed $args = NULL ] )
```

This method invokes several callbacks in succession:

```php
echo $f3->chain('a; b; c', 0);

function a($n) {
	return 'a: '.($n+1); // a: 1
}

function b($n) {
	return 'b: '.($n+1); // b: 1
}

function c($n) {
	return 'c: '.($n+1); // c: 1
}
```

### relay

**Execute specified callbacks in succession; Relay result of previous callback as argument to the next callback**

```php
array relay ( array|string $funcs [, mixed $args = NULL ] )
```

This method invokes callback in succession like [chain](#chain) but applies the result of the first function as argument of the succeeding function, i.e.:

```php
echo $f3->relay('a; b; c', 0);

function a($n) {
	return 'a: '.($n+1); // a: 1
}

function b($n) {
	return 'b: '.($n+1); // b: 2
}

function c($n) {
	return 'c: '.($n+1); // c: 3
}
```

---
## File System


### mutex

** Create mutex, invoke callback then drop ownership when done**

```php
mixed mutex ( string $id, callback $func [, mixed $args = NULL ] )
```

A [Mutual Exclusion (mutex)](http://en.wikipedia.org/wiki/Mutual_exclusion "Wikipedia :: Mutual Exclusion") is a cross-platform mechanism for synchronizing access to resources, in such a way that a process that has acquired the mutex gains exclusive access to the resource. Other processes trying to acquire the same mutex will be in a suspended state until the mutex is released.

```php
$f3->mutex('test',function() {
	// Critical section
	session_start();
	$contents=file_get_contents('mutex');
	sleep(5);
	file_put_contents('mutex',$contents.date('r').' '.session_id()."\n");
});
```

### read

**Read file (with option to apply Unix LF as standard line ending)**

```php
string|FALSE read ( string $file [, bool $lf = FALSE ] )
```

Uses the [`file_get_contents()`](http://www.php.net/file_get_contents "PHP Manual :: file_get_contents") <small>PHP function</small> to read the entire `$file` and return it as a string. Returns `FALSE` on failure.

IF `$lf` is `TRUE`, an EOL conversion to UNIX LF format is performed on all the lines ending.

### write

**Exclusive file write**

```php
int|FALSE write ( string $file, mixed $data [, bool $append = FALSE ] )
```

Uses the [`file_put_contents()`](http://www.php.net/file_put_contents "PHP Manual :: file_put_contents") <small>PHP function</small> with the `LOCK_EX` <small>PHP</small> flag to acquire an exclusive lock on the file while proceeding to the writing.

If `$append` is `TRUE` and the `$file` already exits, the `$data` is appended to the file content instead of overwriting it.

### rel

**Return path relative to the base directory**

``` php
string rel ( string $url )
```

Example:

```php
$f3->set('BASE','http://fatfreeframework.com/');
echo $f3->rel( 'http://fatfreeframework.com/gui/img/supported_dbs.jpg' ); // 'gui/img/supported_dbs.jpg'
```

---

## Misc

### instance

**Return class instance**

```php
$f3 = \Base::instance();
```

This is used to grab the framework instance at any point of your code.


### blacklisted

**Lookup visitor's IP against common DNS blacklist services**

```php
bool blacklisted ( string $ip )
```

This function get called while bootstrapping the application and will lookup the visitors IP against common DNS blacklist services defined by the [DNSBL system variable](quick-reference#dnsbl). This is very useful to protect your application against Spam bots or DOS attacks.

**Return `TRUE` if the IPv4 `$ip` address is present in [DNSBL](quick-reference#dnsbl)**

### config

**Configure framework according to .ini-style file settings**

```php
null config ( string $file )
```

This will parse a configuration file, provided by `$file` and setup the framework with variables and routes.

See the user guide section about [configuration files](framework-variables#configuration-files) to get a full description about how to setup your ini file.


### dump

**Dump (_echo_) expression with syntax highlighting**

```php
null dump ( mixed $expr )
```

<i class="icon-thumbs-up"></i> _NOTICE: The syntax highlighting depends on the [DEBUG level](quick-reference#debug "The DEBUG system variable")_.

### highlight

**Apply syntax highlighting**

```php
string highlight ( string $text )
```

Applies syntax highlighting to a given string and returns the highlighted string.

Example:

```php
$highlighted_code = $f3->highlight( '$fatfree->rocks(\'FAST\' AND $light)' );
```
Returns:

```html
<code><span class="variable">$fatfree</span><span class="object_operator">-&gt;</span><span class="string">rocks</span><span>(</span><span class="constant_encapsed_string">'FAST'</span><span class="whitespace"> </span><span class="logical_and">AND</span><span class="whitespace"> </span><span class="variable">$light</span><span>)</span></code>
```
<div class="alert alert-warning">Keep in mind you need the `code.css` stylesheet to correctly see the syntax highlighting in your browser pages. You can include it in your pages with &lt;link href="code.css" rel="stylesheet" /&gt; (code.css is bundled into the framework 'lib/' folder)</div>

### compile

**Convert JS-style token to PHP expression**

```php
string compile ( string $str )
```
This method is mainly used by the [Preview class](preview), the lightweight template engine, to convert tokens to variables.

Example:

```php
$f3->compile('@RAINBOW.cyan'); // returns: $RAINBOW['cyan']
```

### status

**Send HTTP/1.1 status header; Return text equivalent of status code**

```php
string status ( int $code )
```

Use this method for sending various [HTTP status messages](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html "w3.org :: Status Code Definitions") to the client, e.g.

```php
$f3->status(404); // Sends a '404 Not Found' client error
$f3->status(407); // Sends a '407 Proxy Authentication Required' client error
$f3->status(503); // Sends a '503 Service Unavailable' server error
```

### error

**Execute error handler**

```php
null error ( int $code [, string $text = '' [, array $trace = NULL ]] )
```

Calling this function logs an error and executes the [ONERROR](quick-reference#onerror) handler if defined.
Otherwise it will display a default error page in HTML for synchronous requests, or gives a JSON string for AJAX requests.

### expire

**Send cache metadata to HTTP client**

```php
void expire ( [ int $secs = 0 ] )
```

There is little need to call this method directly because it is automatically invoked at runtime by the framework, depending on whether the page should be cached  or otherwise. The framework sends the necessary HTTP cache control headers to the browser so you don't need to send it yourself.

```php
$f3->expire(0); // sends 'Cache-Control: no-cache, no-store, must-revalidate'
```

### unload

**Execute framework/application shutdown sequence**

```php
null unload ( $cwd )
```
First, changes PHP's current directory to directory `$cwd`, writes session data and ends the session by calling the `session_commit()` <small>PHP function</small>.

Then it shutdowns the application and calls the shutdown handler defined in [UNLOAD](quick-reference#unload).

As a final fallback, an HTTP error `500` is raised if one of the following `E_ERROR, E_PARSE, E_CORE_ERROR, E_COMPILE_ERROR` is detected and not handled by the shutdown handler.
