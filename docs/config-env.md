title: .env File
template: extrahead.html

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
