title: Vouchers
template: extrahead.html

<a href="hhttps://github.com/Devsome/silkroad-laravel/blob/master/config/vouchers.php" target="_blank">config/vouchers.php</a>

```php
/*
* List of characters that will be used for voucher code generation.
*/
'characters' => '23456789ABCDEFGHJKLMNPQRSTUVWXYZ',
```

```php
/*
* Voucher code prefix.
*
* Example: foo
* Generated Code: foo-AGXF-1NH8
*/
'prefix' => null,
```


```php
/*
 * Voucher code suffix.
 *
 * Example: foo
 * Generated Code: AGXF-1NH8-foo
 */
'suffix' => null,
```

```php
/*
 * Code mask.
 * All asterisks will be removed by random characters.
 */
'mask' => '****-****-****-**',
```

```php
/*
 * Separator to be used between prefix, code and suffix.
 */
'separator' => '-',
```
