# Generated Hydrator

GeneratedHydrator is a library about high performance transition of data from arrays to objects and from objects
to arrays.

| Tests | Releases | Downloads | Dependencies |
| ----- | -------- | ------- | ------------- | --------- | ------------ |
|[![Build Status](https://travis-ci.org/Ocramius/GeneratedHydrator.png?branch=master)](https://travis-ci.org/Ocramius/GeneratedHydrator) [![Coverage Status](https://coveralls.io/repos/Ocramius/GeneratedHydrator/badge.png?branch=master)](https://coveralls.io/r/Ocramius/GeneratedHydrator)|[![Latest Stable Version](https://poser.pugx.org/ocramius/generated-hydrator/v/stable.png)](https://packagist.org/packages/ocramius/generated-hydrator) [![Latest Unstable Version](https://poser.pugx.org/ocramius/generated-hydrator/v/unstable.png)](https://packagist.org/packages/ocramius/generated-hydrator)|[![Total Downloads](https://poser.pugx.org/ocramius/generated-hydrator/downloads.png)](https://packagist.org/packages/ocramius/generated-hydrator)|[![Dependency Status](https://www.versioneye.com/package/php--ocramius--generated-hydrator/badge.png)](https://www.versioneye.com/package/php--ocramius--generated-hydrator)|

## What does this thing do?

A [hydrator](http://framework.zend.com/manual/2.1/en/modules/zend.stdlib.hydrator.html) is an object capable of
extracting data from other objects, or filling them with data.

A hydrator performs following operations:

 * Convert `Object` to `array`
 * Put data from an `array` into an `Object`

GeneratedHydrator uses proxying to instantiate very fast hydrators, since this will allow access to protected properties
of the object to be handled by the hydrator.

Also, a hydrator of GeneratedHydrator implements `Zend\Stdlib\Hydrator\HydratorInterface`.

## Usage

Here's an example of how you can create and use a hydrator created by GeneratedHydrator:

```php
<?php

use GeneratedHydrator\Configuration;
use GeneratedHydrator\Factory\HydratorFactory as Factory;

require_once __DIR__ . '/vendor/autoload.php';

class Example
{
    public    $foo = 1;
    protected $bar = 2;
    protected $baz = 3;
}

$config  = new Configuration();
$factory = new Factory($config);

$hydrator = $factory->createProxy('Example');

$object = new Example();

var_dump($hydrator->extract($object)); // array('foo' => 1, 'bar' => 2, 'baz' => 3)
$hydrator->hydrate(
    array('foo' => 4, 'bar' => 5, 'baz' => 6),
    $object
);
var_dump($hydrator->extract($object)); // array('foo' => 4, 'bar' => 5, 'baz' => 6)
```

## Performance comparison

A hydrator generated by GeneratedHydrator is very, very, very fast.
Here's the performance of the various hydrators of `Zend\Stdlib\Hydrator` compared to a hydrator built
by GeneratedHydrator:

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

$iterations = 10000;

class Example
{
    public $foo;
    public $bar;
    public $baz;
    public function setFoo($foo) { $this->foo = $foo; }
    public function setBar($bar) { $this->bar = $bar; }
    public function setBaz($baz) { $this->baz = $baz; }
    public function getFoo() { return $this->foo; }
    public function getBar() { return $this->bar; }
    public function getBaz() { return $this->baz; }
    public function exchangeArray($data) {
        $this->foo = $data['foo']; $this->bar = $data['bar']; $this->baz = $data['baz'];
    }
    public function getArrayCopy() {
        return array('foo' => $this->foo, 'bar' => $this->bar, 'baz' => $this->baz);
    }
}

$object    = new Example();
$data      = array('foo' => 1, 'bar' => 2, 'baz' => 3);
$config    = new GeneratedHydrator\Configuration();
$factory   = new GeneratedHydrator\Factory\HydratorFactory($config);
$hydrators = array(
    $factory->createProxy('Example'),
    new Zend\Stdlib\Hydrator\ClassMethods(),
    new Zend\Stdlib\Hydrator\Reflection(),
    new Zend\Stdlib\Hydrator\ArraySerializable(),
);

foreach ($hydrators as $hydrator) {
    $start = microtime(true);

    for ($i = 0; $i < $iterations; $i += 1) {
        $hydrator->hydrate($data, $object);
        $hydrator->extract($object);
    }

    var_dump(microtime(true) - $start);
}
```

This will produce something like following:

```php
0.028156042098999s
2.606673002243s
0.56710886955261s
0.60278487205505s
```

As you can see, the generated hydrator is 20 times faster than `Zend\Stdlib\Hydrator\Reflection`
and `Zend\Stdlib\Hydrator\ArraySerializable`, and more than 90 times faster than
`Zend\Stdlib\Hydrator\ClassMethods`.

## Limitations

As of current implementation, GeneratedHydrator will not distinguish between properties from following
example:

```php
class Foo
{
    private $bar;
}

class Bar extends Foo
{
    private $bar;
}

class Baz extends Foo
{
    private $bar;
}
```

This will be solved in milestone [1.1.0](https://github.com/Ocramius/GeneratedHydrator/issues?milestone=3)

## Contributing

Please read the [CONTRIBUTING.md](https://github.com/Ocramius/GeneratedHydrator/blob/master/CONTRIBUTING.md) contents
if you wish to help out!
