# Laravel Bernard Service Provider

Brings Bernard to Laravel. Laravel already has a great queue.. That's right, but it only works with Laravel. If your project/company utilizes multiple frameworks, Bernard provides leverage.

## Install & Setup

Extend `composer.json` file:

    {
        "require": {
            "bernard/laravel": "@dev"
        }
    }

Register the service provider in `app/config/app.php`:

    // ...
    'aliases' => array(
        // ..

        'Bernard\Laravel\BernardServiceProvider'

        // ..
    )

## Choose Driver

Now you need to choose the Driver you want to use. Initialize the default config file with `artisan`:

    php artisan config:publish bernard/laravel

This creates the file `app/config/packages/bernard/laravel/config.php`.


### Redis

Config in `app/config/packages/bernard/laravel/config.php`

    <?php

    return array(
        'driver' => 'predis',
    );

Setup `predis` in IoC:

    <?php

    App::singleton('predis', function () {
        return new \Predis\Client(null, array(
            'prefix' => 'bernard:'
        ));
    });

### SQS

Config in `app/config/packages/bernard/laravel/config.php`

    <?php

    return array(
        'driver' => 'sqs',

        // optional: use prefetching for efficiency
        //'prefetch' => 10,

        // optional: pre-set queue name -> url mappings
        //'queue_urls' => array('some-queue' => 'https://sqs.eu-west-1.amazonaws.com/123123/some-queue', ...)
    );

Setup `sqs` in IoC:

    <?php

    use Aws\Sqs\SqsClient;

    // ...

    App::singleton('sqs', function () {
        return SqsClient::factory(array(
           'key'    => 'Your AWS Access Key',
           'secret' => 'Your AWS Secret Key',
           'region' => 'Your AWS Region'
       ));
    });

### Iron MQ

Config in `app/config/packages/bernard/laravel/config.php`

    <?php

    return array(
        'driver' => 'iron_mq',

        // use prefetching for efficiency
        //'prefetch' => 10
    );


Setup `iron_mq` in IoC:

    <?php

    App::singleton('iron_mq', function () {
        return new \IronMq(array(
            'token'      => 'Your IronMQ Token',
            'project_id' => 'Your IronMQ Project ID',
        ));
    });

## Usage

### In Laravel

In your Laravel app, add a new message to the queue:

    $this->app['bernard:producer']->produce(new \Bernard\Message\DefaultMessage('MyService', array(
        'my' => 'args',
    )));

### From command line

    # create a new message
    php artisan bernard:produce MyService '{"json":"data"}'

    # consume messages
    php artisan bernard:consume my-service
