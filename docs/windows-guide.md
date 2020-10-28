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

4. Modify the procedure on `SRO_VT_LOG`.`_AddLogChar` adding the following at the end.
```sql
/* Silkroad-Laravel */
-- Keep the table updated with char (login/logout) status --
IF(@EventID = 4 or @EventID = 6)
BEGIN
    SET NOCOUNT ON;
       INSERT INTO loginhistory ([CharID], [status])
       VALUES (@CharID, @EventId)
       IF EXISTS (SELECT 1 FROM onlineofflinelog WHERE CharID = @CharID)
            UPDATE onlineofflinelog
            SET    status = @EventID
            WHERE  CharID = @CharID
       ELSE
           INSERT INTO onlineofflinelog ([CharID],  [status])
           VALUES  (@CharID, @EventID)
END
-- Update item points on logout status --
IF (@EventID = 6)
BEGIN
    UPDATE [SRO_VT_SHARD].[dbo]._Char
        set ItemPoints = (  
            SELECT
            SUM(
               CASE
                   WHEN Common.CodeName128 LIKE '%_A_RARE' THEN ReqLevel1 + 5
                   WHEN Common.CodeName128 LIKE '%_B_RARE' THEN ReqLevel1 + 10
                   WHEN Common.CodeName128 LIKE '%_C_RARE' THEN ReqLevel1 + 15
               ELSE ReqLevel1
               END
       ) + SUM(ISNULL(Binding.nOptValue, 0)) + SUM(ISNULL(OptLevel, 0)) AS ItemPoints
        FROM [SRO_VT_SHARD].[dbo].[_Inventory] as inventory
        join [SRO_VT_SHARD].[dbo]._Items as Items on Items.ID64  = inventory.ItemID
        join [SRO_VT_SHARD].[dbo]._RefObjCommon as Common on Items.RefItemId  = Common.ID
        left join [SRO_VT_SHARD].[dbo]._BindingOptionWithItem as Binding on Binding.nItemDBID = Items.ID64
        where
            inventory.slot < 13 and
            inventory.slot != 8 and
            inventory.slot != 7 and
            inventory.CharID = _Char.CharID
    ) WHERE _Char.CharID = @CharID

    Declare @GuildID int;
    SELECT @GuildID = GuildID FROM [SRO_VT_SHARD].[dbo]._Char WHERE _Char.CharID = @CharID

    IF (@GuildID > 0)
    BEGIN
        UPDATE [SRO_VT_SHARD].[dbo]._Guild
          set ItemPoints = (
          SELECT
            SUM(Char.ItemPoints) as ItemPoints
            FROM [SRO_VT_SHARD].[dbo]._Char as Char
            where
                Char.GuildID = _Guild.ID
        ) WHERE _Guild.ID = @GuildID
    END
END
```

Also you need to add this to the `_AddLogChar` procedure for the Job and PvP stats

```sql
--Character kills table recorder (finish) , all credits goes to #HB

IF (@EventID = 20) -- PVP
BEGIN
    IF ( @desc LIKE '%Trader, Neutral, no freebattle team%'    -- Trader
        OR @desc LIKE '%Hunter, Neutral, no freebattle team%'    -- Hunter
        OR @desc LIKE '%Robber, Neutral, no freebattle team%'    -- Thief
        OR @desc like '%no job, Neutral, %no job, Neutral%'    -- Free PVP
    )
    BEGIN
        -- Get killer name
        DECLARE @killername VARCHAR(512) = @desc
        DECLARE @killeriD INT = 0
		SELECT  @killername = REPLACE(@killername, LEFT(@killername, CHARINDEX('(', @killername)),'')
		SELECT  @killername = REPLACE(@killername, RIGHT(@killername, CHARINDEX(')', REVERSE(@killername))),'')
        SELECT @killeriD = CharID FROM [SRO_VT_SHARD].[dbo].[_Char] WHERE CharName16 = @killername
        -- Get job type
        DECLARE @jobString VARCHAR(10) = LTRIM(RTRIM(SUBSTRING (@desc, 5, 7)))
        DECLARE @jobType INT = CASE
            WHEN @jobString IN ('Trader', 'Robber', 'Hunter') THEN (SELECT JobType FROM SRO_VT_SHARD.._CharTrijob WHERE CharID = @killeriD)
            ELSE 0 END
        -- Delete original log
        DELETE FROM _LogEventChar WHERE CharID = @CharID AND EventID = 20
            AND (strDesc LIKE '%Trader, Neutral, no freebattle team%'
            OR strDesc LIKE '%Hunter, Neutral, no freebattle team%'
            OR strDesc LIKE '%Robber, Neutral, no freebattle team%'
            OR @desc like '%no job, Neutral, %no job, Neutral%')
        -- Get additional info for notice message
        DECLARE @jobDesc VARCHAR(32) = CASE WHEN @jobType BETWEEN 1 AND 3 THEN 'Job Conflict' ELSE 'Free PVP' END
        DECLARE @strDesc VARCHAR(512)
        IF  (@jobString LIKE 'Trader' OR @jobString LIKE 'Robber' OR @jobString LIKE 'Hunter')
        BEGIN
            -- If it's a Job Kill, then write character nicknames
            DECLARE @killerNickName VARCHAR(64) = (SELECT NickName16 FROM [SRO_VT_SHARD].[dbo].[_Char] WHERE CharID = @killeriD)
            DECLARE @CharnickName VARCHAR(64) = (SELECT NickName16 FROM [SRO_VT_SHARD].[dbo].[_Char] WHERE CharID = @CharID)
            SET @strDesc = '[' + @killerNickName + '] has just killed [' + @CharnickName + '] in [' + @jobDesc + '] mode on [' + CONVERT(NVARCHAR(30), GETDATE(), 0) + ']'
        END

        ELSE BEGIN
            -- If it's normal PVP Kill, write real character names
            SET @strDesc = '[' + @killername + '] has just killed [' + @CharName + '] in [' + @jobDesc + '] mode on [' + CONVERT(NVARCHAR(30), GETDATE(), 0) + ']'
        END

		IF (@killeriD > 0)
		BEGIN
			-- Update the log
			INSERT INTO pvp_records VALUES (@killername, @killeriD, @CharName, @CharID, @jobType, GETDATE(), @strPos, @strDesc)
		END
    END
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
