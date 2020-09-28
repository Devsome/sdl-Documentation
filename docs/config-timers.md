title: Timers
template: extrahead.html

<a href="https://github.com/Devsome/silkroad-laravel/blob/master/config/timers.php" target="_blank">config/timers.php</a>

For the icons, check this link out: <a href="https://fontawesome.com/icons?m=free" target="_blank">fontawesome.com</a>


```php
return [
    [
        'name' => 'Capture the flag',
        'hours' => [
            0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23
        ],
        'min' => 30,
    ],
    [
        'name' => 'Fortresswar',
        'days' => [
            'Sunday',
            'Monday',
        ],
        'hour' => 8,
        'min' => 14,
    ],
    [
        'name' => 'Register',
        'type' => 'static',
        'time' => 'Saturday 12:00 - 23:00',
        'icon' => 'fas fa-clock',
    ],
    [
        'name' => 'Medusa',
        'hours' => [
            1,22,23
        ],
        'min' => 14,
        'icon' => 'fas fa-crosshairs',
    ],
];
```

For example when you add a new timer:

```php
[
    'name' => 'Documentation',
    'hours' => [
        10, 15, 20
    ],
    'min' => 0,
    'icon' => 'fas fa-star'
]
```

<figure>
  <img src="/images/project/timers.png" width="50%" />
  <figcaption>Documentation timer</figcaption>
</figure>
