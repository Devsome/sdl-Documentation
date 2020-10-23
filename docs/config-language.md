title: Language
template: extrahead.html

<a href="https://github.com/Devsome/silkroad-laravel/blob/master/config/language.example.php" target="_blank">config/language.example.php</a>

You need to copy the `language.example.php` to `language.php` or run this command

```bash
cp config/language.example.php config/language.php
```

```php
/*
 * Used for the application language
 */
return [
    'en' => [
        'name' => 'English',
        'icon' => '/image/flags/United_States_of_America.png'
    ],
    'es' => [
        'name' => 'Español',
        'icon' => '/image/flags/Spain.png'
    ],
    'hi' => [
        'name' => 'हिन्दी',
        'icon' => '/image/flags/India.png'
    ],
    'pt' => [
        'name' => 'Portugues',
        'icon' => '/image/flags/Portugal.png'
    ],
    'ar' => [
        'name' => 'العربية',
        'icon' => '/image/flags/Saudi_Arabia.png'
    ]
];
```
If you want to enable, disable or add a new language, just edit the `language.php` file. Remember, that in the `resources/lang/x` a folder with all the language key must exist, else the locale_fallback variable from the app.php is used.

<figure>
  <img src="/images/project/language.png" width="50%" />
  <figcaption>Language array</figcaption>
</figure>
