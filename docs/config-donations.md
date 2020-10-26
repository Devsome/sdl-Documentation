title: Donations
template: extrahead.html

Currently are two Payment methods enabled.

1. Paypal
2. Stripe

Hopefully you got the newest files, else you can get it with `git pull`. Once you updated your files you need to install the new dependencies with `composer install`. After that you can insert the new Payment method **Stripe** with that seeder command.

```bash
php artisan db:seed --class="DonationMethodsSeeder"
```

Paypal & Stripe are going to update itself, you need to activate them in the Backend if you want to use it.

If you are facing any Problems try to run `composer dump-autoload` or for the safety, just run it.

## How to configure them?

Check out the `.env.example`

```env
# Paypal Information
PAYPAL_MODE="sandbox" # live
PAYPAL_SANDBOX_API_CLIENT_ID=""
PAYPAL_SANDBOX_API_SECRET=""
PAYPAL_LIVE_API_CLIENT_ID=""
PAYPAL_LIVE_API_SECRET=""

# Stripe Information
STRIPE_PUBLIC_KEY="pk_test_51HR3q4IUbMZhevnR3hrXn..."
STRIPE_SECRET_KEY="sk_test_51HR3q4IUbMZhevnRtuCRp..."
```

After you put that information into your `.env` do not forget to clear the config with `php artisan config:cache`.
In the backend you can configure the packages the user can buy and you can see a logging.
