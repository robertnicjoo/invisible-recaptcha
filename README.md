Invisible reCAPTCHA
==========
![php-badge](https://img.shields.io/badge/php-%3E%3D%205.6-8892BF.svg)
[![packagist-badge](https://img.shields.io/packagist/v/irando/invisible-recaptcha.svg)](https://packagist.org/packages/irando/invisible-recaptcha)
[![Total Downloads](https://poser.pugx.org/irando/invisible-recaptcha/downloads)](https://packagist.org/packages/irando/invisible-recaptcha)
[![travis-badge](https://api.travis-ci.org/irando/invisible-recaptcha.svg?branch=master)](https://travis-ci.org/irando/invisible-recaptcha)

![invisible_recaptcha_demo](http://i.imgur.com/1dZ9XKn.png)

## Why Invisible reCAPTCHA?

Invisible reCAPTCHA is an improved version of reCAPTCHA v2(no captcha).
In reCAPTCHA v2, users need to click the button: "I'm not a robot" to prove they are human. In invisible reCAPTCHA, there will be not embed a captcha box for users to click. It's totally invisible! Only the badge will show on the buttom of the page to hint users that your website is using this technology. (The badge could be hidden, but not suggested.)


|   Supported Versions	|
|-------------------	|
|       5.8.x        	|
|       5.7.x        	|
|       5.6.x        	|

Currently supports Laravel 5.8.x

## Installation

```
composer require irando/invisible-recaptcha
```

## Laravel 5

### Setup

Add ServiceProvider to the providers array in `app/config/app.php`.

```
irando\InvisibleReCaptcha\InvisibleReCaptchaServiceProvider::class,
```

> It also supports package discovery for Laravel 5.5.

### Configuration
Before you set your config, remember to choose `invisible reCAPTCHA` while applying for keys.
![invisible_recaptcha_setting](http://i.imgur.com/zIAlKbY.jpg)

Add `INVISIBLE_RECAPTCHA_SITEKEY`, `INVISIBLE_RECAPTCHA_SECRETKEY` to **.env** file.

```
// required
INVISIBLE_RECAPTCHA_SITEKEY={siteKey}
INVISIBLE_RECAPTCHA_SECRETKEY={secretKey}

// optional
INVISIBLE_RECAPTCHA_BADGEHIDE=false
INVISIBLE_RECAPTCHA_DATABADGE='bottomright'
INVISIBLE_RECAPTCHA_TIMEOUT=5
INVISIBLE_RECAPTCHA_DEBUG=false
```

> There are three different captcha styles you can set: `bottomright`, `bottomleft`, `inline`

> <strike>If you set `INVISIBLE_RECAPTCHA_BADGEHIDE` to true, you can hide the badge logo.</strike>
>
> **Note:** due to style issue for temporary (I might fix this later) you have no way to hide recaptcha badge unless you add your own css style manually.
>
> How?
> ```css
> .grecaptcha-badge {display: none !important;}
> ```

> You can see the binding status of those catcha elements on browser console by setting `INVISIBLE_RECAPTCHA_DEBUG` as true.

### Usage

Before you render the captcha, please keep those notices in mind:

* `render()` or `renderHTML()` function needs to be called within a form element.
* You have to ensure the `type` attribute of your submit button has to be `submit`.
* There can only be one submit button in your form.

##### Display reCAPTCHA in Your View

```php
// recommended
{!! app('captcha')->render() !!}

// or you can use this in blade
@captcha
```

With custom language support:

```php
// recommended
{!! app('captcha')->render('en') !!}

// or you can use this in blade
@captcha('en')
```

##### Usage with Javascript frameworks like VueJS:

The `render()` process includes three distinct sections that can be rendered separately incase you're using the package with a framework like VueJS which throws console errors when `<script>` tags are included in templates.

You can render the polyfill (do this somewhere like the head of your HTML:)

```php
{!! app('captcha')->renderPolyfill() !!}
// Or with blade directive:
@captchaPolyfill
```

You can render the HTML using this following, this needs to be INSIDE your `<form>` tag:

```php
{!! app('captcha')->renderCaptchaHTML() !!}
// Or with blade directive:
@captchaHTML
```

And you can render the neccessary `<script>` tags including the optional language support by using:

```php
// The argument is optional.
{!! app('captcha')->renderFooterJS('en') !!}

// Or with blade directive:
@captchaScripts
// blade directive, with language support:
@captchaScripts('en')

```

##### Validation

Add `'g-recaptcha-response' => 'required|captcha'` to rules array.

```php
$validate = Validator::make(Input::all(), [
    'g-recaptcha-response' => 'required|captcha'
]);

```

## CodeIgniter 3.x

set in application/config/config.php :
```php
$config['composer_autoload'] = TRUE;
```

add lines in application/config/config.php :
```php
$config['recaptcha.sitekey'] = 'sitekey'; 
$config['recaptcha.secret'] = 'secretkey';
// optional
$config['recaptcha.options'] = [
    'hideBadge' => false,
    'dataBadge' => 'bottomright',
    'timeout' => 5,
    'debug' => false
];
```

In controller, use:
```php
$data['captcha'] = new \irando\InvisibleReCaptcha\InvisibleReCaptcha(
    $this->config->item('recaptcha.sitekey'),
    $this->config->item('recaptcha.secret'),
    $this->config->item('recaptcha.options'),
);
```

In view, in your form:
```php
<?php echo $captcha->render(); ?>
```

Then back in your controller you can verify it:
```php
$captcha->verifyResponse($_POST['g-recaptcha-response'], $_SERVER['REMOTE_ADDR']);
```

## Without Laravel or CodeIgniter

Checkout example below:

```php
<?php

require_once "vendor/autoload.php";

$siteKey = 'sitekey';
$secretKey = 'secretkey';
// optional
$options = [
    'hideBadge' => false,
    'dataBadge' => 'bottomright',
    'timeout' => 5,
    'debug' => false
];
$captcha = new \irando\InvisibleReCaptcha\InvisibleReCaptcha($siteKey, $secretKey, $options);

// you can override single option config like this
$captcha->setOption('debug', true);

if (!empty($_POST)) {
    var_dump($captcha->verifyResponse($_POST['g-recaptcha-response'], $_SERVER['REMOTE_ADDR']));
    exit();
}

?>

<form action="?" method="POST">
    <?php echo $captcha->render(); ?>
    <button type="submit">Submit</button>
</form>
```

## Take Control of Submit Function
Use this function only when you need to take all control after clicking submit button. Recaptcha validation will not be triggered if you return false in this function.

```javascript
_beforeSubmit = function(e) {
    console.log('submit button clicked.');
    // do other things before captcha validation
    // e represents reference to original form submit event
    // return true if you want to continue triggering captcha validation, otherwise return false
    return false;
}
```

## Customize Submit Function
If you want to customize your submit function, for example: doing something after click the submit button or changing your submit to ajax call, etc.

The only thing you need to do is to implement `_submitEvent` in javascript
```javascript
_submitEvent = function() {
    console.log('submit button clicked.');
    // write your logic here
    // submit your form
    _submitForm();
}
```
Here's an example to use an ajax submit (using jquery selector)
```javascript
_submitEvent = function() {
    $.ajax({
        type: "POST",
        url: "{{route('message.send')}}",
         data: {
            "name": $("#name").val(),
            "email": $("#email").val(),
            "content": $("#content").val(),
            // important! don't forget to send `g-recaptcha-response`
            "g-recaptcha-response": $("#g-recaptcha-response").val()
        },
        dataType: "json",
        success: function(data) {
            // success logic
        },
        error: function(data) {
            // error logic
        }
    });
};
```
## Repository
Repo: https://github.com/robertnicjoo/invisible-recaptcha

This repo demonstrates how to use this package with ajax way.

By: [CV. Irando](https://irando.co.id)

Forked and improved from [albertcht invisible recaptcha](https://github.com/albertcht/invisible-recaptcha)