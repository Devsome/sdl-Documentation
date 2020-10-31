title: Windows Guide
template: extrahead.html

# Windows Guide


## Silkroad-Laravel Manual Install

### Requirements

- Laragon Full (Git, MySQL, Apache, Ngnix, PHP, Composer, NodeJS, SSL emulator, virtual hosts, etc)
- PHP v7.4+ The minimum required for project
  - `Thead Safe` (TS) if you plan to use Apache
  - `Non Thread Safe` (NTS) if you plan to use Ngnix
- SQL Drivers for PHP (5.8)
- SQL Server Database from Silkroad already installed

#### Shortcut Links
- <a href="https://laragon.org/download/">laragon.org</a>
- <a href="https://www.php.net/downloads">php.net</a>
- <a href="https://docs.microsoft.com/en-us/sql/connect/php/download-drivers-php-sql-server?view=sql-server-ver15">php sql server drivers</a>

### Installation

#### Enviroment Setup

1. Install _Laragon Full_ and make sure to add the option about "Add console to Windows context"

2. Extract the PHP compressed file to the Laragon php folder, will be probably this:
```
C:\Laragon\bin\php
```

3. Install the _SQL Drivers for PHP_ on `ext` folder inside the last php folder extracted like:
```
C:\Laragon\bin\php\php7.4xxxxxxxxxxxx\ext
```

4. Execute _Laragon_, go to MENU and change the PHP version to use v7.4+

5. From same MENU, activate the _SQL Drivers for PHP_ extensions referencing your server system choosen and OS system (TS or NTS, x64 or x86):
- `php_pdo_sqlsrv_74_ts_x64.dll`
- `php_sqlsrv_74_ts_x64.dll`

6. Check _Laragon_ Preferences and activate virtual hosts

7. press DATABASE from _Laragon_ and create an empty **web** database (MySQL `utf8mb4_unicode_ci`)
named as `DB_SILKROAD` to match our .env.example file

8. Press TERMINAL button from _Laragon_ and make sure the php version is v7.4+
```bash
php -version
```
If is not correct, then try to add the PATH to your System Enviroment Variables from the php v7.4+ folder.

9. From same TERMINAL, Create a folder `silkroad-laravel` for your project
```bash
mkdir silkroad-laravel
```

10. Change directory from console to start all the project commands and stuffs from there
```bash
cd silkroad-laravel
```

#### Project Setup

1. Clonate/download the project to the current directory console
```bash
git clone https://github.com/Devsome/silkroad-laravel.git
```

2. Make a copy of `.env.example` file and name it as `.env`, fill it with both database credentials, also make sure APP_URL points to your virtual host (https://silkroad-laravel.test at this case)

3. Execute the next query on `SRO_VT_SHARD`
```sql
ALTER TABLE dbo._Char ADD ItemPoints int NOT NULL DEFAULT 0;
ALTER TABLE dbo._Guild ADD ItemPoints int NOT NULL DEFAULT 0;
```

4. You need to create a new Silkroad Database called `SRO_WEB_LARAVEL`

4. Modify the procedure on `SRO_VT_LOG`.`_AddLogChar` adding the following like so:
```sql
CREATE   procedure [dbo].[_AddLogChar]
    @CharID		int,
    @EventID		tinyint,
    @Data1		int,
    @Data2		int,
    @strPos		varchar(64),
    @Desc		varchar(128)
as
EXEC SRO_WEB_LARAVEL.dbo._WebProcess @CharID = @CharID,@EventID = @EventID,@DESC = @Desc,@strPos = @strPos
```

Important is to add this `EXEC SRO_WEB_LARAVEL.dbo._WebProcess @CharID = @CharID,@EventID = @EventID,@DESC = @Desc,@strPos = @strPos` right after the `as`.

Now modify the `_AddLogSiegeFortress` procedure and add the following at the end:

```sql
DECLARE @FWStatus VARCHAR(10) = (CASE WHEN @EventID = 1 THEN 'Running' WHEN @EventID = 2 THEN 'Stopped' ELSE 'Unknown' END)
IF (@EventID IN(1,2))
BEGIN
IF NOT EXISTS(SELECT * FROM SRO_WEB_LARAVEL.dbo._FortressStatus WHERE Status LIKE 'Running' OR Status LIKE 'Stopped')
BEGIN
TRUNCATE TABLE SRO_WEB_LARAVEL.dbo._FortressStatus
INSERT INTO SRO_WEB_LARAVEL.dbo._FortressStatus (Status,Date) VALUES (@FWStatus,GETDATE())
END
ELSE
UPDATE SRO_WEB_LARAVEL.dbo._FortressStatus SET Status = @FWStatus, Date = GETDATE()
END
```

5. Install php dependencies with composer
```bash
composer install
```

6. Laravel commands for generating the storage path, unique key and table migrations to databases.
```bash
php artisan storage:link && php artisan key:generate && php artisan migrate --seed
```

7. If you want to add your existing Silkroad accounts to the **web** database, execute:
```bash
php artisan import:silkroad-accounts
```

8. Add manually and backend role, insert on `model_has_roles` table from MySQL database:
```
role_id : 3
model_type : App\User
model_id : Your account ID on `users` table
```

### Notes

- Clearing the cache is a good idea for development stuffs.
```bash
php artisan cache:clear && php artisan config:cache
```

Thanks to [@JellyBitz](https://github.com/JellyBitz)
