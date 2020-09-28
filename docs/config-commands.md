title: Commands
template: extrahead.html

### Create and fill tables

Laravel is doing all the other stuff, just go to the silkroad-laravel folder you cloned that project and run:
```
php artisan storage:link && php artisan key:generate && php artisan migrate --seed
```

- This is for creating a storage link folder for images.
- Generating an secret key for the project.
- Creating all that web tables and filling it with default values.


### Cronjob

Setting up your cronjob
```
crontab -e
```

Then paste this and put it on the last line.
```
* * * * * php /var/www/artisan schedule:run
```

### Import Silkroad Accounts

If you want to add your current existing `TB_User` accounts to the web project, that the user can login and donate or update there password, you can run this command:

```
php artisan import:silkroad-accounts
```
