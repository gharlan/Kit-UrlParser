# RFC 3986 URL Parser #

*UrlParser* is PHP library that provides a [RFC 3986](https://tools.ietf.org/html/rfc3986)
compliant URL parser and a [PSR-7](http://www.php-fig.org/psr/psr-7/) compatible
URI component. The purpose of this library is to provide a parser that
accurately implements the RFC specification unlike the built in function
`parse_url()`, which differs from the specification in some subtle ways.

This library has two main purposes. The first to provide information from the
parsed URLs. To achive this, the library implements the standard URI handling
interface from the PSR-7 and also provides additional methods that make it
easier to retrieve commonly used information from the URLs. The second purpose
is to also permit the modification of said URLs using the interface from the
PSR-7 standard in addition to few extra methods that make some tasks more
straightforward.

While this library is mainly intended for parsing URLs, the parsing is simply
based on the generic URI syntax. Thus, it is possible to use this library to
validate and parse any other types of URIs against the generic syntax. The
library does not perform any scheme specific validation for the URLs.

In addition to the default RFC 3986 compliant mode, the library also offers
options that allow parsing of URLs that contain UTF-8 characters in different
components of the URL while converting them to the appropriate percent encoded
and IDN ascii formats.

The API documentation, which can be generated using Apigen, can be read online
at: http://kit.riimu.net/api/urlparser/

[![Build Status](https://img.shields.io/travis/Riimu/Kit-UrlParser.svg?style=flat)](https://travis-ci.org/Riimu/Kit-UrlParser)
[![Code Coverage](https://img.shields.io/scrutinizer/coverage/g/Riimu/Kit-UrlParser.svg?style=flat)](https://scrutinizer-ci.com/g/Riimu/Kit-UrlParser/)
[![Scrutinizer Code Quality](https://img.shields.io/scrutinizer/g/Riimu/Kit-UrlParser.svg?style=flat)](https://scrutinizer-ci.com/g/Riimu/Kit-UrlParser/)
[![HHVM Status](https://img.shields.io/hhvm/riimu/Kit-UrlParser.svg)](http://hhvm.h4cc.de/package/riimu/Kit-UrlParser)
[![PHP7 Status](https://img.shields.io/badge/PHP7-tested-brightgreen.svg)]()

## Requirements ##

In order to use this library, the following requirements must be met:

  * PHP version 5.4
  * [PSR Http Message](https://github.com/php-fig/http-message) library is required
  * In order to parse IDNs, the php extension `intl` must be enabled

## Installation ##

This library can be installed by using [Composer](http://getcomposer.org/). In
order to do this, you must download the latest Composer version and run the
`require` command to add this library as a dependency to your project. The
easiest way to complete these two tasks is to run the following two commands
in your terminal:

```
php -r "readfile('https://getcomposer.org/installer');" | php
php composer.phar require "riimu/kit-urlparser:2.*"
```

If you already have Composer installed on your system and you know how to use
it, you can also install this library by adding it as a dependency to your
`composer.json` file and running the `composer install` command. Here is an
example of what your `composer.json` file could look like:

```json
{
    "require": {
        "riimu/kit-urlparser": "2.*"
    }
}
```

After installing this library via Composer, you can load the library by
including the `vendor/autoload.php` file that was generated by Composer during
the installation.

### Manual installation ###

You can also install this library manually without using Composer. In order to
do this, you must download the [latest release](https://github.com/Riimu/Kit-UrlParser/releases/latest)
and extract the `src` folder from the archive to your project folder. To load
the library, you can simply include the `src/autoload.php` file that was
provided in the archive.

Note that if you install this library manually, you must also install the
dependencies by yourself. Installing the library via Composer also installs the
dependencies for you.

## Usage ##

Using this library is relatively straightforward. The library provides a URL
parsing class `UriParser` and an immutable value object class `Uri` that
represents the URL. To parse an URL, you could simply provide the URL as a
string to the `parse()` method in `UriParser` which returns an instance of `Uri`
that has been generated from the parsed URL.

For example:

```php
<?php

require 'vendor/autoload.php';

$parser = new \Riimu\Kit\UrlParser\UriParser();
$uri = $parser->parse('http://www.example.com');

echo $uri->getHost(); // Outputs 'www.example.com'
```

Alternatively, you can just skip using the `UriParser` completely and simply
provide the URL as a constructor parameter to the `Uri`:

```php
<?php

require 'vendor/autoload.php';
$uri = new \Riimu\Kit\UrlParser\Uri('http://www.example.com');
echo $uri->getHost(); // Outputs 'www.example.com'
```

The main difference between using the `parse()` method and the constructor is
that the `parse()` method will return a `null` if the provided URL is not a
valid url, while the constructor will throw an `InvalidArgumentException`.

To retrieve different types of information from the URL, the `Uri` class
provides various different methods to help you. Here is a simple example as an
overview of the different available methods:

```php
<?php

require 'vendor/autoload.php';

$parser = new \Riimu\Kit\UrlParser\UriParser();
$uri = $parser->parse('http://jane:pass123@www.example.com:8080/site/index.php?action=login&prev=index#form');

echo $uri->getScheme() . PHP_EOL;         // outputs: http
echo $uri->getUsername() . PHP_EOL;       // outputs: jane
echo $uri->getPassword() . PHP_EOL;       // outputs: pass123
echo $uri->getHost() . PHP_EOL;           // outputs: www.example.com
echo $uri->getTopLevelDomain() . PHP_EOL; // outputs: com
echo $uri->getPort() . PHP_EOL;           // outputs: 8080
echo $uri->getStandardPort() . PHP_EOL;   // outputs: 80
echo $uri->getPath() . PHP_EOL;           // outputs: /site/index.php
echo $uri->getPathExtension() . PHP_EOL;  // outputs: php
echo $uri->getQuery() . PHP_EOL;          // outputs: action=login&prev=index
echo $uri->getFragment() . PHP_EOL;       // outputs: form

print_r($uri->getPathSegments());    // [0 => 'site', 1 => 'index.php']
print_r($uri->getQueryParameters()); // ['action' => 'login', 'prev' => 'index']
```

The `Uri` component also provides various methods for modifying the URL, which
allows you to construct new URLs from separate components or modify existing
ones. Note that the `Uri` component is an immutable value object, which means
that each of the modifying methods return a new `Uri` instance instead of
modifying the existing one. Here is a simple example of constructing an URL
from it's components:

```php
<?php

require 'vendor/autoload.php';

$uri = (new \Riimu\Kit\UrlParser\Uri())
    ->withScheme('http')
    ->withUserInfo('jane', 'pass123')
    ->withHost('www.example.com')
    ->withPort(8080)
    ->withPath('/site/index.php')
    ->withQueryParameters(['action' => 'login', 'prev' => 'index'])
    ->withFragment('form');

// Outputs: http://jane:pass123@www.example.com:8080/site/index.php?action=login&prev=index#form
echo $uri;
```

As can be seen from the previous example, the `Uri` component also provides a
`__toString()` method that provides the URL as a string.

### Retrieving Information ###

Here is the list of methods that the `Uri` component provides for retrieving
information from the URL:

  * `getScheme()` returns the scheme from the URL or an empty string if the URL
    has no scheme.

  * `getAuthority()` returns the component from the URL that consists of the
    username, password, hostname and port in the format `user-info@hostname:port`

  * `getUserInfo()` returns the component from the URL that contains the
    username and password separated by a colon.

  * `getUsername()` returns the *decoded* username from the URL or an empty
    string if there is no username present in the URL.

  * `getPassword()` returns the *decoded* password from the URL or an empty
    string if there is no password present in the URL.

  * `getHost()` return the hostname from the URL or an empty string if the URL
    has no host.

  * `getIpAddress()` returns the IP address from the host, if the host is an
    IP address. Otherwise this method will return `null`. If an IPv6 address
    was provided, the address is returned without the surrounding braces.

  * `getTopLevelDomain()` returns the top level domain from the host. If there
    is no host or the host is an IP address, an empty string will be returned
    instead.

  * `getPort()` returns the port from the URL or a `null` if there is no port
    present in the url. This method will also return a `null` if the port is the
    standard port for the current scheme (e.g. 80 for http).

  * `getStandardPort()` returns the standard port for the current scheme. If
    there is no scheme or the standard port for the scheme is not known, a
    `null` will be returned instead.

  * `getPath()` returns the path from the URL or an empty string if the URL has
    no path.

  * `getPathSegments()` returns an array of *decoded* path segments (i.e. the
    path split by each forward slash). Empty path segments are discarded and not
    included in the returned array.

  * `getPathExtension()` returns the file extension from the path or an empty
    string if the URL has no path.

  * `getQuery()` returns the query string from the URL or an empty string if the
    URL has no query string.

  * `getQueryParameters()` parses the query string from the URL using the
    `parse_str()` function and returns the array of parsed values.

  * `getFragment()` returns the fragment from the URL or an empty string if the
    URL has no fragment.

  * `__toString()` returns the URL as a string.

### Modifying the URL ###

The `Uri` component provides various methods that can be used to modify URLs
and construct new ones. Note that since the `Uri` class is an immutable value
object, each method returns a new instance of `Uri` rather than modifying the
existing one.

  * `withScheme($scheme)` returns a new instance with the given scheme. An empty
    scheme can be used to remove the scheme from the URL. Note that any provided
    scheme is normalized to lowercase.

  * `withUserInfo($user, $password = null)` returns a new instance with the
    given username and password. Note that the password is ignored unless an
    username is provided. Empty username can be used to remove the username and
    password from the URL. Any character that cannot be inserted in the URL by
    itself will be percent encoded.

  * `withHost($host)` returns a new instance with the given host. An empty host
    can be used to remove the host from the URL. Note that this method does not
    accept international domain names. Note that this method will also normalize
    the host to lowercase.

  * `withPort($port)` returns a new instance with the given port. A `null` can
    be used to remove the port from the URL.

  * `withPath($path)` returns a new instance with the given path. An empty path
    can be used to remove the path from the URL. Note that any character that is
    not a valid path character will be percent encoded in the URL. Existing
    percent encoded characters will not be double encoded, however.

  * `withPathSegments(array $segments)` returns a new instance with the path
    constructed from the array of path segments. All invalid path characters in
    the segments will be percent encoded, including the forward slash and
    existing percent encoded characters.

  * `withQuery($query)` returns a new instance with the given query string. An
    empty query string can be used to remove the path from the URL. Note that
    any character that is not a valid query string character will be percent
    encoded in the URL. Existing percent encoded characters will not be double
    encoded, however.

  * `withQueryParameters(array $parameters)` returns a new instance with the
    query string constructed from the provided parameters using the
    `http_build_query()` function. All invalid query string characters in the
    parameters will be percent encoded, including the ampersand, equal sign and
    existing percent encoded characters.

  * `withFragment($fragment)` returns a new instance with the given fragment. An
    empty string can be used to remove the fragment from the URL. Note that any
    character that is not a valid fragment character will be percent encoded in
    the URL. Existing percent encoded characters will not be double encoded,
    however.

### UTF-8 and International Domains Names ###

By default, this library provides a parser that is RFC 3986 compliant. The RFC
specification does not permit the use of UTF-8 characters in the domain name or
any other parts of the URL. The correct representation for these in the URL is
to use the an IDN standard for domain names and percent encoding the UTF-8
characters in other parts.

However, to help you deal with UTF-8 encoded characters, many of the methods in
the `Uri` component will automatically percent encode any characters that cannot
be inserted in the URL on their own, including UTF-8 characters. Due to
complexities involved, however, the `withHost()` method does not allow UTF-8
encoded characters.

By default, the parser also does not parse any URLs that include UTF-8 encoded
characters because that would be against the RFC specification. However, the
parser does provide two additional parsing modes that allows these characters
whenever possible.

If you wish to parse URLs that may contain UTF-8 characters in the user
information (i.e. the username or password), path, query or fragment components
of the URL, you can simply use the UTF-8 parsing mode. For example:

```php
<?php

require 'vendor/autoload.php';

$parser = new \Riimu\Kit\UrlParser\UriParser();
$parser->setMode(\Riimu\Kit\UrlParser\UriParser::MODE_UTF8);

$uri = $parser->parse('http://www.example.com/föö/bär.html');
echo $uri->getPath(); // Outputs: /f%C3%B6%C3%B6/b%C3%A4r.html
```

UTF-8 characters in the domain name, however, are a bit more complex issue. The
parser, however, does provide a rudimentary support for parsing these domain
names using the IDNA2003 mode. For example:

```php
<?php

require 'vendor/autoload.php';

$parser = new \Riimu\Kit\UrlParser\UriParser();
$parser->setMode(\Riimu\Kit\UrlParser\UriParser::MODE_IDNA2003);

$uri = $parser->parse('http://www.fööbär.com');
echo $uri->getHost(); // Outputs: www.xn--fbr-rla2ga.com
```

Note that using this parsing mode requires the PHP extension `intl` to be
enabled. The appropriate parsing mode can also be provided to the constructor of
the `Uri` component using the second constructor parameter.

While support for parsing these UTF-8 characters is available, this library does
not provide any methods for the reverse operations since the purpose of this
library is to deal with RFC 3986 compliant URIs.

### URL Normalization ###

Due to the fact that the RFC 3986 specification defines some URLs as equivalent
despite having some slight differences, this library does some minimal
normalization to the provided values. You may encounter these instances when,
for example, parsing URLs provided by users. The most notable normalizations you
may encounter are as follows:

  * The `scheme` and `host` components are considered case insensitive. Thus,
    these components will always be normalized to lower case.
  * The port number will not be included in the strings returned by
    `getAuthority()` and `__toString()` if the port is the standard port for the
    current scheme.
  * Percent encodings are treated in case insensitive manner. Thus, this library
    will normalize the hexadecimal characters to upper case.
  * The number of forward slashes in the beginning of the path in the string
    returned by `__toString()` may change depending on whether the URL has an
    `authority` component or not.
  * Percent encoded characters in parsed and generated URIs in the `userinfo`
    component may differ due to the fact that the `UriParser` works with the
    PSR-7 specification which does not provide a way to provide encoded username
    or password.

## Credits ##

This library is copyright 2013 - 2015 to Riikka Kalliomäki.

See LICENSE for license and copying information.
