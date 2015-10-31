## Polycast
[![Packagist License](https://poser.pugx.org/leemason/polycast/license.png)](http://choosealicense.com/licenses/mit/)
[![Latest Stable Version](https://poser.pugx.org/leemason/polycast/version.png)](https://packagist.org/packages/leemason/polycast)
[![Total Downloads](https://poser.pugx.org/leemason/polycast/d/total.png)](https://packagist.org/packages/leemason/polycast)

Laravel Websocket broadcasting polyfill using ajax and mysql. Laravel 5.1 or Later

## Installation

Require this package with composer:

```
composer require leemason/polycast
```

After updating composer, add the ServiceProvider to the providers array in config/app.php

### Laravel 5.1:

```php
LeeMason\Polycast\PolycastServiceProvider::class,
```

Add the following in your broadcasting connections array located in config/broadcasting.php

```php
'polycast' => [
    'driver' => 'polycast',
    'delete_old' => 2, //this deletes old events after 2 minutes, this can be changed to leave them in the db longer if required.
]
```

Copy the package assets to your public folder with the publish command:

```php
php artisan vendor:publish --tag=public --force
```

Migrate the packages database migrations (creates the polycast_events table):

```php
php artisan migrate --path=vendor/leemason/polycast/migrations
```

## Usage

To Optionally set Polycast as your default broadcast events driver set ```polycast``` as the default in your config/broadcasting.php or ```BROADCAST_DRIVER=polycast``` in your .env file.

Once installed you create broadcastable events exactly the same as you do now (using the ShouldBroadcast trait), except you have a way to consume those events via browsers without the need for nodejs/redis or an external library to be installed/purchased.

This package doesnt aim to compete with libraries or solutions such as PRedis/SocketIO/Pusher.
But what it does do is provide a working solution for situations where you can't install nodejs and run a websocket server, or where the cost of services like Pusher aren't feasible.

The package utilizes vanilla javascript timeouts and ajax requests to "simulate" a realtime experience.
It does so by saving the broadcastable events in the database, via a setTimeout javascript ajax request, polls the packages receiver url and distrubutes the payloads via javascript callbacks.

I have tried to keep the javascript api similar to current socket solutions to reduce the learning curve.

Here's an example:

```javascript
<script src="<?php echo url('vendor/polycast/polycast.min.js');?>"></script>
<script>
    (function() {

        //create the connection
        var poly = new Polycast('http://localhost/polycast', {
            token: '<?php echo csrf_token();?>'
        });

        //subscribe to channel(s)
        var channel1 = poly.subscribe('channel1');
        var channel2 = poly.subscribe('channel2');

        //fire when event on channel 1 is received
        channel1.on('Event1WasFired', function(data){
            console.log(data);
        });

        //fire when event on channel 2 is received
        channel2.on('Event2WasFired', function(data){
            var body = document.getElementById('body');
            body.innerHTML = body.innerHTML + JSON.stringify(data);
        });

    }());
</script>
```


