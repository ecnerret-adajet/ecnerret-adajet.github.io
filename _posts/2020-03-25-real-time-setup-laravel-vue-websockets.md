---
layout: single
title:  "Real-Time Broadcast for Laravel 7 and Vue and websockets "
categories:
  - Web
tags:
  - real-time
  - broadcast
  - events
  - websockets
  - vue
  - laravel
---
Long before ago, Setting up real-time web based applications is complex and not easy to maintain.
thankfully pusher and websockets has arrived and helped many developers to run realtime sockets into their laravel apps.

Here's the a simple guide to include realtime feature into your laravel apps. yung laravel-websocket pacakge.

### 1. Install Laravel Websockets
You may refer the official documentation of the page and follow the instructions. or you may go along with this guide.

from your laravel project path from your command line interface. run the following script below.

```yaml
composer require beyondcode/laravel-websockets
```

```yaml
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="migrations"
```

```yaml
php artisan migrate
```

### 2. Pusher Prerequisites
Install pusher channels via composer
```yaml
composer require pusher/pusher-php-server "~4.0"
```
Open the `config/broadcasting.php` and this configuration for `option`
```javascript
    'options' => [
          'cluster' => env('PUSHER_APP_CLUSTER'),
          'encrypted' => true,
          'host' => '127.0.0.1', // make sure the host ip is same with your served ip
          'port' => 6001,
          'scheme' => 'http'
      ],
```
**Note:** Important note: make sure the `host` IP is your served IP. when running `php artisan serve`
{: .notice--info}

lastly, go to `.env` file and change the Broadcast_driver to pusher

```yaml
BROADCAST_DRIVER=pusher
```

### 3. Laravel Echo
Install laravel echo in order to subscribe to pusher channel
```yaml
npm install --save laravel-echo pusher-js
```

Open `resources/js/bootstrap.js` and enable the echo configuration required to listen channels for events.

```javascript
import Echo from 'laravel-echo';

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: process.env.MIX_PUSHER_APP_KEY,
    wsHost: window.location.hostname,
    wsPort: 6001,
    disableStats: true,
});
```

### 4. Setup Channel route





