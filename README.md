# Troubleshooting Bedrock 718
See https://github.com/roots/bedrock/issues/718.

## Setup
```
trellis init
trellis up
```
This should get your local provisioned with Redis and two sites running with [WP Redis](https://wordpress.org/plugins/wp-redis/) plugin installed & activated.

## Links:
- [site1.test Admin](https://site1.test/wp/wp-admin)
- [site2.test Admin](https://site2.test/wp/wp-admin)

## Steps to reproduce:
- Login (admin:admin), view Settings > General and confirm site details are correct.
- Go to `config/application.php` in both sites and uncomment lines 15 & 41, comment out line 40. This is the change introduced in 714.
- Check both sites. If one has changed to show details of the other, you are seeing the issue.
- Restart Redis on the server `sudo systemctl restart redis-server` and/or flush cache `wp cache flush` from the site folders. Confirm it does not solve the problem.
- Remove `web/app/object-cache.php` from the site that is showing incorrect information. Note this corrects the information.
- Enable Redis for that site again `wp redis enable` (you may need to reactivate the plugin from within WP-Admin). Note it's now incorrect.

## Outcome
In setting this repo up I ultimately found the real cause: using `getenv()`. This is actually noted in [Bedrock 714](https://github.com/roots/bedrock/pull/714).

To fix, in `config/application.php` for both sites, replace `getenv()` for `env()`. In this specific case the only usage is

```diff
// Unique cache key salt for memcached/redis
- Config::define('WP_CACHE_KEY_SALT', getenv('DB_NAME'));
+ Config::define('WP_CACHE_KEY_SALT', env('DB_NAME'));
```
