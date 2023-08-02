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
Setting up broadcasting with Laravel, Vue, and WebSockets involves several steps. Broadcasting allows you to send real-time events to connected clients using WebSockets. In this guide, we'll use Laravel Echo as the JavaScript library to handle WebSocket connections and Laravel Broadcasting for the server-side event handling. 

We'll assume you have already set up Laravel and Vue in your project. If not, make sure to do that before proceeding..

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

### 4. Broadcast Events
Create the events you want to broadcast in your Laravel application. For example, you can create an event by running:

```javascript
php artisan make:event NewMessageEvent
```

### 5. Broadcast the Event
In your Laravel application code (controller or other logic), trigger the event to be broadcasted:

```php
use App\Events\NewMessageEvent;

// ... Your code ...

event(new NewMessageEvent($data));
```

### 6. Subscribe to the Event in Vue:
In your Vue components, you can now listen for the event and handle it accordingly. For example:

```javascript
  mounted() {

    Echo.channel(`message-channel`
      ).listen("NewMessageEvent", (e) => {

        // code here

      });

  }
```

### 7. Create channel route:
Open `channels.php` and add the following line or new channel route. ensure that the channel name is similar to the `Echo.channel` parameter from step 6.

```php
  Broadcast::channel('message-channel', function () {
      return true;
  });
```

### 8. Start the WebSocket Server:
Run the following command to start the Laravel WebSockets server:

```yaml
php artisan websockets:serve
```

he server should be running now, and you can access it at ws://your-domain.com:6001.

That's it! Now, when the NewMessageEvent is triggered in your Laravel application, it will be broadcasted through WebSockets, and your Vue components will receive the event data and handle it in real-time.

Please note that this guide assumes you're using Laravel, Vue, and Laravel Echo with the Pusher driver. If you choose to use a different broadcasting driver or WebSocket library, the steps might vary.