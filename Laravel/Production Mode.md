# Putting a Laravel App into Production

Bringing a Laravel application to production involves a series of steps to ensure that your website is ready to be used by real users safely and efficiently.

## ENV File

When you bring a Laravel application to production, it is important to make some changes to the [.env](https://laravel.com/docs/10.x/configuration) file to properly configure the production environment.

+ APP_ENV

    > Change the value of APP_ENV to 'production' to indicate that you are in a production environment.

    ```
    APP_ENV=production
    ```

+ APP_KEY

    > Genera una nueva clave de aplicación única.

    ```powershell
    php artisan key:generate
    ```

+ APP_DEBUG

    > Change the value of APP_DEBUG to "false" to disable debugging in production.

    ```
    APP_DEBUG=false
    ```

+ APP_URL

    > Make sure your website URL is set correctly in APP_URL.

    ```
    APP_URL=https://yourwebsite.com
    ```

+ DATABASE

    > Adjust the database configuration to connect to the production database.

    ```
    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=database_name
    DB_USERNAME=username
    DB_PASSWORD=password
    ```
    
+ EMAIL

    > Configure email information so your app can send emails from the production environment.

    ```
    MAIL_MAILER=smtp
    MAIL_HOST=smtp.gmail.com
    MAIL_PORT=465
    MAIL_USERNAME=username
    MAIL_PASSWORD=password
    MAIL_ENCRYPTION=ssl
    MAIL_FROM_ADDRESS=email_address
    MAIL_FROM_NAME="${APP_NAME}"
    ```

## Autoloader optimization

Optimizing the [autoloader](https://getcomposer.org/doc/articles/autoloader-optimization.md) is particularly important when deploying your Laravel application to production, as it can significantly improve response times and overall performance.

> Run the following Composer command to optimize the autoloader

```
composer dump-autoload --optimize
```

> You can run the command along with installing dependencies

```
composer install --optimize-autoloader --no-dev
```

> If you are updating the dependencies you can also run the command

```
composer update --optimize-autoloader
```

## Asset Bundling

Asset bundling is the process of grouping multiple assets into a single file or a small number of files. This bundling process is often used to improve website and application performance by reducing the number of HTTP requests and optimizing the delivery of assets to the client's browser.

### Vite

> Build and version the [assets](https://laravel.com/docs/10.x/vite) for production

```powershell
npm run build
```

### Laravel Mix

> Vite has replaced [Laravel Mix](https://laravel-mix.com/) in new Laravel installations. If you don't want to migrate to Vite you can run the following command.

```powershell
npm run production
```

## Public Disk

The [public disk](https://laravel.com/docs/master/filesystem#the-public-disk) is one of the predefined file storage disks used for managing and serving publicly accessible files.

> To create the symbolic link, you may use the the next artisan command

```powershell
php artisan storage:link
```