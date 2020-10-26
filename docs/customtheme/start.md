title: Start
template: extrahead.html

If you want to overwrite existing Views, you can create a own template.

The good thing about having your own template is that you can still get the latest updates with `git pull` without having your changes overwritten.

I have created a test template to illustrate this a little.

This is the resources folder with 2 themes.

<figure>
  <img src="/images/customtheme/resources.png" width="50%" />
  <figcaption>Resources folder</figcaption>
</figure>

The `test-theme` is now overwriting the

- app.blade.php
- sidebar.blade.php
- downloads.blade.php

So every change you do in the test-theme files, there will be overwritten.

To change some path or files example file `app.blade.php`
```html
<link rel="icon" href="{{ asset('themes/test-theme/favicon/sdl.png') }}">
<link href="{{ asset('themes/test-theme/css/main.css') }}" rel="stylesheet">
```


This is how the public folder will look like:

<figure>
  <img src="/images/customtheme/public.png" width="50%" />
  <figcaption>Resources folder</figcaption>
</figure>

You can see the `themes/test-theme/css/main.css`

Download the test-theme.zip <a href="/customtheme/test-theme.zip">here</a>.


### Hint
After you changed your theme in the `.env` you need to run these commands.

```
php artisan config:clear && php artisan view:clear && php artisan cache:clear
 ```
