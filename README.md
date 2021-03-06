# QuiteSimpleXMLElement

[![Build Status](http://img.shields.io/travis/danmichaelo/quitesimplexmlelement.svg?style=flat-square)](https://travis-ci.org/danmichaelo/quitesimplexmlelement)
[![Coverage Status](http://img.shields.io/coveralls/danmichaelo/quitesimplexmlelement.svg?style=flat-square)](https://coveralls.io/r/danmichaelo/quitesimplexmlelement?branch=master)
[![Code Quality](http://img.shields.io/scrutinizer/g/danmichaelo/quitesimplexmlelement/master.svg?style=flat-square)](https://scrutinizer-ci.com/g/danmichaelo/quitesimplexmlelement/?branch=master)
[![SensioLabsInsight](https://insight.sensiolabs.com/projects/2654313d-e4ea-48dd-90c0-c13863d10c3a/mini.png)](https://insight.sensiolabs.com/projects/2654313d-e4ea-48dd-90c0-c13863d10c3a)
[![Latest Stable Version](http://img.shields.io/packagist/v/danmichaelo/quitesimplexmlelement.svg?style=flat-square)](https://packagist.org/packages/danmichaelo/quitesimplexmlelement)
[![Total Downloads](http://img.shields.io/packagist/dt/danmichaelo/quitesimplexmlelement.svg?style=flat-square)](https://packagist.org/packages/danmichaelo/quitesimplexmlelement)

The `QuiteSimpleXMLElement` class is a small wrapper built around the `SimpleXMLElement` class that adds some convenience methods and makes it easier to work with namespaces. The main reason for developing the class was to let objects returned by the `xpath()` method inherit namespaces from the original object. The package was formerly known as `CustomXMLElement`. 

QuiteSimpleXMLElement supports PHP 5.6 and 7.x. If you need PHP 5.3 support, use the `0.4.*` version range as PHP 5.3 support was removed in version 0.5.

The library is actively maintained and pull requests are welcome.

## Why this library was developed

Taking an example document,

```php
$xml = '<root xmlns:dc="http://purl.org/dc/elements/1.1/">
    <dc:a>
      <dc:b >
        1.9
      </dc:a>
    </dc:b>
  </root>';
```

Using SimpleXMLElement I found myself having to register namespaces over and over again:

```php
$root = new SimpleXMLElement($xml);
$root->registerXPathNamespace('d', 'http://purl.org/dc/elements/1.1/');
$a = $root->xpath('d:a');
$a[0]->registerXPathNamespace('d', 'http://purl.org/dc/elements/1.1/');
$b = $a[0]->xpath('d:b');
```

When using QuiteSimpleXMLElement, it should only be necessary to register the namespaces once and for all.

```php
$node = new QuiteSimpleXMLElement($xml);
$node->registerXPathNamespace('d', 'http://purl.org/dc/elements/1.1/');
$a = $node->xpath('d:a');
$b = $a->xpath('d:b');
```

The namespaces can also be defined using the alternative constructor `QuiteSimpleXMLElement::make`:

```php
$node = QuiteSimpleXMLElement::make($xml, ['d' => 'http://purl.org/dc/elements/1.1/']);
$a = $node->xpath('d:a');
$b = $a->xpath('d:b');
```

A note on the design: I would have preferred to extend the original SimpleXMLElement class, but the constructor is static, which is why I wrote a wrapper instead.

## Helper methods

The library defines some new methods to support less typing and cleaner code.

### attr($name)

Returns the value of an attribute as a string. Namespace prefixes are supported.

```php
echo $node->attr('id');
```

### text($xpath)

Returns the text content of the node

```php
echo $node->text('d:a/d:b');
```

### first($xpath)

Returns the first node that matches the given path, or null if none.

```php
$node = $node->first('d:a/d:b');
```

#### all($xpath)

Returns all nodes that matches the given path, or an empty array if none.

```php
$node = $node->all('d:a/d:b');
```

### has($xpath)

Returns true if the node exists, false if not

```php
if ($node->has('d:a/d:b') {
	…
}
```

### setValue($value)

Sets the value of a node

```php
$node->setValue('Hello world');
```

### replace($newNode)

Replaces the current node with a new one. Example:

```php
$book = new QuiteSimpleXMLElement('
<book>
	<chapter>
		<title>Chapter one</title>
	</chapter>
	<chapter>
		<title>Chapter two</title>
	</chapter>
</book>
');

$introduction = new QuiteSimpleXMLElement('
	<introduction>
		<title>Introduction</title>
	</introduction>
');

$firstChapter = $book->first('chapter');
$firstChapter->replace($introduction);
```

gives

```xml
<?xml version="1.0"?>
<book>
    <introduction>
        <title>Introduction</title>
    </introduction>
    <chapter>
        <title>Chapter two</title>
    </chapter>
</book>
```

Works with namespaces as well, but any namespaces used in the replacement node
must be specified in that document as well. See `QuiteSimpleXMLElementTest.php`
for an example.
