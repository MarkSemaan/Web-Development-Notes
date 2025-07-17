## What are traits?
---
 Traits were introduced in PHP 5.4 as a mechanism for **code reuse**. They allow you to define a set of methods that can be "mixed in" or "copied" into multiple classes, effectively overcoming the single inheritance limitation for sharing behavior.

```php
php artisan make:trait
```

##### Example
---
##### Response Traits
```php
<?php
namespace App\Traits;
trait ResponseTrait{

	function responseJSON($status = sucess,  $payload, $status_code = '200'){
		return response()->json([
			"status" => $status,
			"payload" => $payload
		], $status_code);
	}
}
```