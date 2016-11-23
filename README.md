Block - PHP Native Template System
===========================================

Block is PHP native template system inspiring by Laravel Blade.
Block is not template language, block doesn't need to be compiled and cached like blade, twig, smarty, etc.

## Getting Started

#### Preparation

Block using static methods, so instead initialize an object, 
you just need to set class alias and set views directory.

```php
<?php

class_alias('Emsifa\Block', 'Block'); // this is optional, you can aliasing as whatever you want
Block::setDirectory('path/to/views');

```

#### Your first template

Create file `path/to/views/hello.php`.

```php
<h1><?= $title ?></h1>
<p>
	<?= $message ?>
</p>
```

#### Render it

Then somewhere in your code, you can render it with `render` method like this:

```php

echo Block::render('hello', [
	'title' => 'Hello World'
	'message' => 'Lorem ipsum dolor sit amet'
]);

```

> Note: you must ignore extension `.php` in `render`, `insert` and `extend` method.

## Extending and Blocking

There is two main view type in most template engines or template systems. 
Master view, and Page view.

Master view is a view that contain base and required html tags like `<doctype>`, `<html>`, `<head>`, `<body>`, etc.
Page view is a view that `extend` master view and contain some blocks that defined in master view.

> Note: Master view **is not** for rendered by `render` method. Master view is just for extended by some view pages.

If you familiar with laravel blade syntax, here is the differences.

| Blade                 | Block                               |
|-----------------------|-------------------------------------|
| @extends('view-name') | <?php Block::extend('view-name') ?> |
| @section('content')   | <?php Block::start('content') ?>    |
| @stop                 | <?php Block::stop() ?>              |
| @show                 | <?php Block::show() ?>              |
| @yield('content')     | <?php Block::get('content') ?>      |

Here is a simple real world case using extending and blocking

#### Create Master View

```php
<!-- Stored in path/to/views/master.php -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title><?= $title ?></title>
  <?= Block::start('stylesheets') ?>
  <link rel="stylesheet" href="bootstrap.css">
  <?= Block::show() ?>
</head>
<body>
  <header>
    <h1>App Name</h1>
  </header>
  <div id="content">
    <?= Block::get('content') ?>
  </div>
  <footer>
    @copy; 2016 - my app
  </footer>
  <?= Block::start('scripts') ?>
  <script src="jquery.js"></script>
  <script src="bootstrap.js"></script>
  <?= Block::show() ?>
</body>
</html>
```

> In example above we use `<?=` for `Block::start`, `Block::show` methods. 
  It is ok because those methods doesn't return a value.
  But for `Block::get` you need to `echo` them, so you must use `<?=` instead just `<?php`. 

#### Create Page View

In master view above, there are blocks `stylesheets`, `content`, and `scripts`.
So we need to define them in our page view. 

```php
<!-- Stored in path/to/views/pages/lorem-ipsum.php -->
<?= Block::extend('master') ?>

<?= Block::start('content') ?>
<p>
	Lorem ipsum dolor sit amet, consectetur adipisicing elit. 
	Officiis, mollitia ad commodi. 
	Eligendi saepe unde iusto quis, praesentium deleniti eos incidunt quas vero, 
	voluptatem, reiciendis inventore, aliquam expedita et rerum.
</p>
<?= Block::stop() ?>

<?= Block::start('stylesheets') ?>
	<?= Block::parent() ?>
	<link rel="stylesheet" href="lorem.css">
<?= Block::stop() ?>

<?= Block::start('scripts') ?>
	<?= Block::parent() ?>
	<script src="lorem.js"></script>
	<script>
		initPage();
	</script>
<?= Block::stop() ?>
```

> All blocks above actually is optional. But we need to show you how real world like.

#### Render It!

Now you can render a page view with render method.

```php
echo Block::render('pages/lorem-ipsum', [
	'Lorem Ipsum'
]); 
```

And the result should look like this

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Lorem Ipsum</title>
  <link rel="stylesheet" href="bootstrap.css">
  <link rel="stylesheet" href="lorem.css">
</head>
<body>
  <header>
    <h1>App Name</h1>
  </header>
  <div id="content">
    <p>
      Lorem ipsum dolor sit amet, consectetur adipisicing elit. 
      Officiis, mollitia ad commodi. 
      Eligendi saepe unde iusto quis, praesentium deleniti eos incidunt quas vero, 
      voluptatem, reiciendis inventore, aliquam expedita et rerum.
    </p>
  </div>
  <footer>
    @copy; 2016 - my app
  </footer>
  <script src="jquery.js"></script>
  <script src="bootstrap.js"></script>
  <script src="lorem.js"></script>
  <script>
    initPage();
  </script>
</body>
</html>
```

## Just it? No!

Here is some useful things and case

#### `$get($key, $default = NULL)`

There is `$get` variable that contain anonymous function in your view files.
This function allows you to get a value passed by `render` method. 
If the key exists, it will return that value, 
and if not it will return default value (NULL).

For example in master view above, if you don't pass `title` key in array, it will show an error undefined variable title.
So instead using `isset` like this

```php
<title><?= isset($title) ? $title : 'Default Title' ?></title>
```

You can use `$get` like this:

```php
<title><?= $get('title', 'Default Title') ?></title>
```

#### `Block::insert($view, array $data = array())

There is another view type called `partial view`. 
Partial view is a view file that contain a partial layout 
that you can use in some page or master view like widget, sidebar, navbar, main-menu, etc.
Partial view is like master view, it is not for rendered by `render` method. 
But you can render it by put it in master or page view via `insert` method.

For example, let's create a new page view that contain a widget slider 

First we need to create partial view for slider widget:

```php
<!-- Stored in path/to/views/partials/slider.php -->
<div class="widget widget-slider">
  <div class="slider-wrapper">
    <div class="slide-1">Slide 1</div>
    <div class="slide-2">Slide 2</div>
    <div class="slide-3">Slide 3</div>
  </div>
</div>

<?= Block::start('stylesheets') ?>
	<?= Block::parent() ?>
	<link rel="stylesheet" href="slider.css">
<?= Block::stop() ?>

<?= Block::start('scripts') ?>
	<?= Block::parent() ?>
	<script src="slider.js"></script>
<?= Block::stop() ?>

```

Then we put it in home page view.

```php
<!-- Stored in path/to/views/pages/home.php -->
<?= Block::extend('master') ?>

<?= Block::start('content') ?>
<div class="container">
  <?= Block::insert('partial/slider') ?>
  <p>
    Lorem ipsum dolor sit amet, consectetur adipisicing elit. 
    Officiis, mollitia ad commodi. 
    Eligendi saepe unde iusto quis, praesentium deleniti eos incidunt quas vero, 
    voluptatem, reiciendis inventore, aliquam expedita et rerum.
  </p>
</div>
<?= Block::stop() ?>

<?= Block::start('stylesheets') ?>
  <?= Block::parent() ?>
  <link rel="stylesheet" href="home.css">
<?= Block::stop() ?>

<?= Block::start('scripts') ?>
  <?= Block::parent() ?>
  <script src="home.js"></script>
  <script>
    initHomepage()
  </script>
<?= Block::stop() ?>
```

Now if you `echo Block::render('pages/home')`, the output should like:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Default Title</title>
  <link rel="stylesheet" href="bootstrap.css">
  <link rel="stylesheet" href="slider.css">
  <link rel="stylesheet" href="home.css">
</head>
<body>
  <header>
    <h1>App Name</h1>
  </header>
  <div id="content">
    <div class="container">
    <div class="widget widget-slider">
      <div class="slider-wrapper">
        <div class="slide-1">Slide 1</div>
        <div class="slide-2">Slide 2</div>
        <div class="slide-3">Slide 3</div>
      </div>
    </div>
    <p>
      Lorem ipsum dolor sit amet, consectetur adipisicing elit. 
      Officiis, mollitia ad commodi. 
      Eligendi saepe unde iusto quis, praesentium deleniti eos incidunt quas vero, 
      voluptatem, reiciendis inventore, aliquam expedita et rerum.
    </p>
    </div>
  </div>
  <footer>
    @copy; 2016 - my app
  </footer>
  <script src="jquery.js"></script>
  <script src="bootstrap.js"></script>
  <script src="slider.js"></script>
  <script src="home.js"></script>
  <script>
    initHomepage();
  </script>
</body>
</html>
```

Yup! `slider.css` and `slider.js` is placed in that order!

> Note: If you want to use page view data in partial view, you can pass `$__data` in `Block::insert`. 
  For example, in slider above will be `Block::insert('slider', $__data)`

#### Add Directory Namespace

You can put second argument in `setDirectory` method to set namespaced directory.

For example, you have module admin that have it's own views directory.

```php

Block::addDirectory('path/to/admin/views', 'admin');

// then you can load master/page/partial view in that directory like this
Block::render('admin::pages/dashboard');
Block::extend('admin::master');
Block::insert('admin::partials/some-chart');

```