[![Build Status](https://travis-ci.org/hollodotme/fast-cgi-client.svg?branch=master)](https://travis-ci.org/hollodotme/fast-cgi-client)
[![Coverage Status](https://coveralls.io/repos/github/hollodotme/fast-cgi-client/badge.svg?branch=master)](https://coveralls.io/github/hollodotme/fast-cgi-client?branch=master)

# Fast CGI Client

A PHP fast CGI client to send requests (a)synchronously to PHP-FPM using the [FastCGI Protocol](http://www.mit.edu/~yandros/doc/specs/fcgi-spec.html).

This library is based on the work of [Pierrick Charron](https://github.com/adoy)'s [PHP-FastCGI-Client](https://github.com/adoy/PHP-FastCGI-Client/) 
and was ported and modernized to PHP 7.0/PHP 7.1 and extended with unit tests.

---

## Installation

### Use version 1.x for compatibility with PHP 7.0.x

```bash
composer require hollodotme/fast-cgi-client:^1.0
```

### Use version 2.x for compatibility with PHP 7.1.x

```bash
composer require hollodotme/fast-cgi-client:^1.0
```

---

## Usage

### Init client with a unix domain socket connection

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

require( 'vendor/autoload.php' );

use hollodotme\FastCGI\Client;
use hollodotme\FastCGI\SocketConnections\UnixDomainSocket;

$connection = new UnixDomainSocket(
	'unix:///var/run/php/php7.0-fpm.sock',  # Socket path to php-fpm
	5000,                                   # Connect timeout in milliseconds (default: 5000)
	5000,                                   # Read/write timeout in milliseconds (default: 5000)
	false,                                  # Make socket connection persistent (default: false)
	false                                   # Keep socket connection alive (default: false) 
);

$client = new Client( $connection );
```

### Init client with a network socket connection

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

require( 'vendor/autoload.php' );

use hollodotme\FastCGI\Client;
use hollodotme\FastCGI\SocketConnections\NetworkSocket;

$connection = new NetworkSocket(
	'127.0.0.1',    # Hostname
	9000,           # Port
	5000,           # Connect timeout in milliseconds (default: 5000)
	5000,           # Read/write timeout in milliseconds (default: 5000)
	false,          # Make socket connection persistent (default: false)
	false           # Keep socket connection alive (default: false) 
);

$client = new Client( $connection );
```

### Send request synchronously

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

require( 'vendor/autoload.php' );

use hollodotme\FastCGI\Client;
use hollodotme\FastCGI\SocketConnections\UnixDomainSocket;

$client  = new Client( new UnixDomainSocket( 'unix:///var/run/php/php7.0-fpm.sock' ) );
$content = http_build_query(['key' => 'value']);

$response = $client->sendRequest(
	[
		'GATEWAY_INTERFACE' => 'FastCGI/1.0',
		'REQUEST_METHOD'    => 'POST',
		'SCRIPT_FILENAME'   => '/path/to/target/script.php',
		'SERVER_SOFTWARE'   => 'hollodotme/fast-cgi-client',
		'REMOTE_ADDR'       => '127.0.0.1',
		'REMOTE_PORT'       => '9985',
		'SERVER_ADDR'       => '127.0.0.1',
		'SERVER_PORT'       => '80',
		'SERVER_NAME'       => 'your-server',
		'SERVER_PROTOCOL'   => 'HTTP/1.1',
		'CONTENT_TYPE'      => 'application/x-www-form-urlencoded',
		'CONTENT_LENGTH'    => mb_strlen($content)	
	],
	$content
);

print_r( $response );
```

### Send request asynchronously

```php
<?php declare(strict_types=1);

namespace YourVendor\YourProject;

require( 'vendor/autoload.php' );

use hollodotme\FastCGI\Client;
use hollodotme\FastCGI\SocketConnections\NetworkSocket;

$client  = new Client( new NetworkSocket( '127.0.0.1', 9000 ) );
$content = http_build_query(['key' => 'value']);

$requestId = $client->sendAsyncRequest(
	[
		'GATEWAY_INTERFACE' => 'FastCGI/1.0',
		'REQUEST_METHOD'    => 'POST',
		'SCRIPT_FILENAME'   => '/path/to/target/script.php',
		'SERVER_SOFTWARE'   => 'hollodotme/fast-cgi-client',
		'REMOTE_ADDR'       => '127.0.0.1',
		'REMOTE_PORT'       => '9985',
		'SERVER_ADDR'       => '127.0.0.1',
		'SERVER_PORT'       => '80',
		'SERVER_NAME'       => 'your-server',
		'SERVER_PROTOCOL'   => 'HTTP/1.1',
		'CONTENT_TYPE'      => 'application/x-www-form-urlencoded',
		'CONTENT_LENGTH'    => mb_strlen($content)	
	],
	$content
);

echo "Request sent, got ID: {$requestId}";
```

---

## Command line tool (for debugging only)

Run a call through a network socket:

    bin/fcgiget localhost:9000/status

Run a call through a Unix Domain Socket

    bin/fcgiget unix:/var/run/php/php7.0-fpm.sock/status
