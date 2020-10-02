title: FAQ
template: extrahead.html

#### Frequently Asked Questions

<br>

1. <a href="#debug-on">Server Error - Debug on</a>
2. <a href="#how-can-i-see-the-backend">How can I see the backend?</a>
3. <a href="#missing-tables">Missing Tables</a>
4. <a href="#tables-are-empty">Tables are empty</a>
5. <a href="#images-are-not-shown">Images are not shown</a>
6. <a href="#multi-language">I don't need multi language</a>
7. <a href="#discord-widget">Discord widget not working</a>

<hr>

## Debug on

> You want to see the error message instead of "Server Error xxx"?
Then edit the `.env` and put the `APP_DEBUG=true` `DEBUGBAR_ENABLED=true`
After that you will see a error message instead of server error.
> When you done editing the `.env` file run `php artisan config:cache`

## How can I see the backend?

> Probably there is no account registered yet that has the rights. <br>
Table `DB_SILKROAD.model_has_roles` <br>
> Replace `X` with your id. You can find your id in that table `DB_SILKROAD.users` the `id`!

| role_id | model_type | model_id |
| -- | -- | -- |
| 3 | `App\User` | X |

<small>Hint: You can use the command `php artisan import:silkroad-accounts` to get all your Silkroad Accounts into that web database. The Accounts need a Name + E-Mail set.</small>


## Missing Tables

> You forgot to run that command `php artisan migrate`

## Tables are empty

> You forgot to run that command `php artisan db:seed`

## Images are not shown

> You forgot to run that command `php artisan storage:link`

## Multi language

> If you don't need multi language for your page, you can simple delete the other language.
Just open the `config/app.php` and search for `locale_enabled`, there you can delete the other language.
After that you need to run that command `php artisan config:cache` to recache your whole config and make the changes visible.

> But if you are saying you need one more language, check the `resources/lang` folder. For each language you need a separated folder. That folder name is the key in the `locale_enabled` array.

## Discord widget

> Your Discord server needs server-widgets activated. Then it can take some time to make it work.
