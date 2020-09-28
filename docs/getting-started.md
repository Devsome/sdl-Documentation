title: Installation
template: extrahead.html

This guide is for Debian/Ubuntu (vServer), if you want to install it on a Windows, use <a href="/windows-guide">this</a> guide.

### Requirements
- PHP v7.4 The minimum required for project
- git
- Webserver
- Silkroad Databases

All the other stuff we need, we are going to install it.

### Cloning that Project

Downloading that project with git.
```
git clone https://github.com/Devsome/silkroad-laravel.git
```

Installing all the packages which are required.
```
cd silkroad-laravel && composer install
```

### SRO Database customizations

#### Procedure
For the ranking and other stuff you need to edit your currenct Silkroad procedures.
After all the `set` variables you need to paste this stuff
`SRO_VT_LOG`.`_AddLogChar`

```sql
IF(@EventID = 4 or @EventID = 6)
BEGIN
  SET NOCOUNT ON;
    INSERT INTO loginhistory ([CharID], [status])
    VALUES (@CharID, @EventId)
  IF EXISTS (SELECT 1 FROM onlineofflinelog WHERE CharID = @CharID)
    UPDATE onlineofflinelog
    SET    status = @EventID
    WHERE	CharID = @CharID
  ELSE
    INSERT INTO onlineofflinelog ([CharID],  [status])
    VALUES      (@CharID, @EventID)
END
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

That inserts a record into the `SRO_VT_LOG`.`onlineofflinelog` & `SRO_VT_LOG`.`loginhistory`.<br>
If you think, that you are missing that table. Do the next step and the Table exist. <br>
Also the EventId = 6 part is for the ranking of your players.

#### Adding ItemPoints

Open your Silkroad Database query and run this commands.
```SQL
ALTER TABLE dbo._Char ADD ItemPoints int NOT NULL DEFAULT 0;
ALTER TABLE dbo._Guild ADD ItemPoints int NOT NULL DEFAULT 0;
```

That stuff allows the Database to fill the Itempoints to the ranking.

### Editing the .ENV

You need to edit your .env.example file from that project and fill it with your credentials.
```
cp .env.example .env
```
or just rename it manually to `.env`

Information:
```md
# Your website url
APP_URL="https://silkroad-laravel/"

# Simple description of your page
APP_DESCRIPTION="Website description"

# Timezone for the whole webpage / timers
APP_TIMEZONE="Europe/Berlin"

# You can fill this out
DISCORD_WEBHOOK_AUCTION="https://discordapp.com/api/webhooks/ID"
```

Paypal Information:
```md
PAYPAL_MODE="live"
PAYPAL_SANDBOX_API_CLIENT_ID=""
PAYPAL_SANDBOX_API_SECRET=""
PAYPAL_LIVE_API_CLIENT_ID=""
PAYPAL_LIVE_API_SECRET=""
```

Web project Database:
```md
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=DB_SILKROAD
DB_USERNAME=root
DB_PASSWORD=root
```

Silkroad Database stuff:
```md
DB_SQL_HOST=YourIp
DB_SQL_PORT=YourPort
DB_SQL_DATABASE_ACCOUNT=SRO_VT_ACCOUNT
DB_SQL_DATABASE_LOG=SRO_VT_LOG
DB_SQL_DATABASE_SHARD=SRO_VT_SHARD
DB_SQL_DATABASE_CUSTOM=SRO_VT_CUSTOM
DB_SQL_USERNAME=YourUsername
DB_SQL_PASSWORD=YourPassword
```

E-Mail stuff:
```md
MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```


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
