# serialize.js
Convert JS data types into string representations parsable by PHP's [unserialize()](http://php.net/manual/en/function.unserialize.php) function or parse serialized strings into JS data types. Multi-level arrays/objects
are fully supported.

**Methods:**
> _string_ serialize( _mixed_ data [, _bool_ assoc = false [, _bool_ nonEnum = false ]] )

> _mixed_ unserialize( _string_ str )

## Examples
Similar to `JSON.stringify()`, `serialize()` is invoked via the global `PHP` object:
```js
PHP.serialize('I can\'t believe it\'s not butter!'); // s:32:"I can't believe it's not butter!";
PHP.serialize({tobias:'queen mary'}); // O:8:"stdClass":1:{s:6:"tobias";s:10:"queen mary";}
PHP.serialize(['Fred','Daphne','Velma','Shaggy','Scooby-Doo']); // a:5:{i:0;s:4:"Fred";i:1;s:6:"Daphne";i:2;s:5:"Velma";i:3;s:6:"Shaggy";i:4;s:10:"Scooby-Doo";}
PHP.serialize(42); // i:42;
PHP.serialize(Math.PI); // d:3.141592653589793;
PHP.serialize(Infinity); // d:INF;
PHP.serialize(null); // N;
```
The second parameter, `assoc`, dictates whether objects will be conveyed as associative arrays to PHP.
By default, they will be represented as standard objects (`stdClass`).
```js
PHP.serialize({P:'pretty',Y:'young',T:'thing'}, false); // O:8:"stdClass":3:{s:1:"P";s:6:"pretty";s:1:"Y";s:5:"young";s:1:"T";s:5:"thing";}
PHP.serialize({P:'pretty',Y:'young',T:'thing'}, true); // a:3:{s:1:"P";s:6:"pretty";s:1:"Y";s:5:"young";s:1:"T";s:5:"thing";}
```
The resulting output of `var_dump(unserialize())` of those serialized strings is:
```
class stdClass#1 (3) {
  public $P =>
  string(6) "pretty"
  public $Y =>
  string(5) "young"
  public $T =>
  string(5) "thing"
}

array(3) {
  'P' =>
  string(6) "pretty"
  'Y' =>
  string(5) "young"
  'T' =>
  string(5) "thing"
}
```
The third parameter, `nonEnum`, dictates whether non-enumerable properties of an object will be serialized:
```js
PHP.serialize(Math,true,false); // a:0:{}
PHP.serialize(Math,true,true); // a:43:{s:1:"E";d:2.718281828459045;s:4:"LN10";d:2.302585092994046;s:3:"LN2";d:0.693147180559.... (truncated)
```
The `unserialize()` function can be invoked similarly:
```js
PHP.unserialize('s:29:"Public schools are so random.";'); // "Public schools are so random."
PHP.unserialize('a:6:{i:0;i:1;i:1;i:2;i:2;i:3;i:3;s:1:"a";i:4;s:1:"b";i:5;s:1:"c";}'); // [1, 2, 3, "a", "b", "c"]
PHP.unserialize('N;'); // null
PHP.unserialize('b:1;'); // true
PHP.unserialize('O:6:"Person":3:{s:9:"firstName";s:3:"Bob";s:8:"lastName";s:6:"Loblaw";s:10:"occupation";s:6:"Lawyer";}'); // {firstName: "Bob", lastName: "Loblaw", occupation: "Lawyer"}
```
Associative arrays will be unserialized as objects in JS, though bear in mind that there is no guarantee that
the order of keys will be preserved. Any PHP object type will be unserialized as an instance of `Object`.

Be prepared that `unserialize()` will throw an exception if it cannot parse the serialized string or if it was
passed a value other than a string. The exception message indicates the problem offset in the same way as
the notice issued by PHP's function.

## Supported Types
Every type of data in JS is technically supported, though the resulting serialized representation
may not always be completely meaningful. For instance, functions are treated as objects,
so they will likely be serialized as an empty object/array, unless properties were explicitly assigned.

Types that JavaScript reports as `"object"` via `typeof` that have an analogous PHP type
(i.e. arrays or `null`) are handled appropriately. All other types of objects are treated as
"plain" objects and only their own properties (as opposed to prototypical properties)
are serialized. The JavaScript value of `undefined` is conveyed as `NULL` since that is the closest equivalent in PHP.

Though not widely used<sup>1</sup>, JavaScript supports the concept of sparse arrays, wherein an array
has fewer defined values than its length property reports. For example:
```js
var arr = ['so',,,'far',,,'away'];
arr.length; // 7
arr.forEach(function(val){console.log(val);});
// so
// far
// away
```
(Notice that when iterating on the array, only the three defined values are handled.) PHP does not have a similar
concept of sparse arrays (e.g. `[ 0 => 'foo', 60 => 'bar' ]` does not implicitly set keys 1-59), so to
faithfully represent the complete picture of a sparse JS array, the undefined indices will be filled in with `null` values.

### Analogous Types
| JavaScript Type  | PHP Type |
| ------------ | ------------- |
| string | string |
| integer | integer |
| float | float / double |
| NaN | NAN |
| +/-Infinity | +/-INF |
| null | null |
| undefined | null |
| boolean | boolean |
| array | array |
| object | stdObj or assoc array |
| function | stdObj or assoc array |

## Why would I need this?
You probably won't. However, I was prompted to write this function when I had to interact with an API that expected
POST data containing a PHP-serialized object and delivered its responses as serialized data.
So perhaps you will run into a scenario like that. If not,
it's still interesting to take a look under the hood and see how PHP's
[serialize()](http://php.net/manual/en/function.serialize.php) function works!

---
<sup>1</sup> I have no citation, but it's probably true.
