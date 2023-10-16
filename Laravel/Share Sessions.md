# Share Sessions Between Laravel Applications

It's convenient to have a single user account that you can use for authentication across all of your applications. Therefore, a user will be logged in everywhere after authenticating with any of these apps.

To do this we must first perform some steps and configure each of the applications. What we need to do is:

+ Use a common database
+ Share cookies, sessions and users
+ Use the same encryption key
+ Use the same domain

## Create a Common Database

We will just keep common information like users, sessions, roles, and permissions in this database. If necessary, you can store more data, but bear in mind that all programs will have access to it.

In this case, managing the ACL will be facilitated using a Laravel application integrated with Jetstream. We have the following migrations in this project:

+ 2014_10_12_000000_create_users_table.php
+ 2014_10_12_100000_create_password_reset_tokens_table.php
+ 2014_10_12_200000_add_two_factor_columns_to_users_table.php
+ 2019_08_19_000000_create_failed_jobs_table.php
+ 2019_12_14_000001_create_personal_access_tokens_table.php
+ 2023_03_13_134518_create_sessions_table.php

If your project does not have a migration for the `sessions` table, you can create it with the artisan command:

```powershell
php artisan session:table
```

## Configure ENV File

In the first step we must create the variables that we will use to configure the connection and those that will help us create the cookies.

#### Application Key

The value of the `APP_KEY` variable must be the same in all your applications, you can generate a new one with the artisan command:

```powershell
php artisan key:generate
```
#### Database keys

We need to create new variables that we will use to configure the new connection that we will create later. In this case we will use the `_PERMISSIONS` prefix.


```
DB_HOST_PERMISSIONS=127.0.0.1
DB_PORT_PERMISSIONS=3306
DB_DATABASE_PERMISSIONS=database_name
DB_USERNAME_PERMISSIONS=database_username
DB_PASSWORD_PERMISSIONS=database_password
```

#### Session keys

We need to set the session driver, connection and domain. This configuration must be the same in all the applications.

> You must to include a dot before your domain name.

```
SESSION_CONNECTION=permissions
SESSION_COOKIE=your_cookie_name
SESSION_DOMAIN=".your-domain.com"
```

## Update your Database Configuration

We need to add a new connection in all the applications that we will use. In the `config\database.php` file in the `connections` array we must add the new connection. Just copy one of your existing configs and change the name and parameters.

> In this case we use the name ***permissions*** for the connection and for the parameters we use the suffix ***_PERMISSIONS***.

```php
'permissions' => [
    'driver'         => 'mysql',
    'url'            => env('DATABASE_URL'),
    'host'           => env('DB_HOST_PERMISSIONS', '127.0.0.1'),
    'port'           => env('DB_PORT_PERMISSIONS', '3306'),
    'database'       => env('DB_DATABASE_PERMISSIONS', 'forge'),
    'username'       => env('DB_USERNAME_PERMISSIONS', 'forge'),
    'password'       => env('DB_PASSWORD_PERMISSIONS', ''),
    'unix_socket'    => env('DB_SOCKET', ''),
    'charset'        => 'utf8mb4',
    'collation'      => 'utf8mb4_unicode_ci',
    'prefix'         => '',
    'prefix_indexes' => true,
    'strict'         => true,
    'engine'         => null,
    'options'        => extension_loaded('pdo_mysql') ? array_filter([
        PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
    ]) : [],
    'dump'           => [
        'useSingleTransaction' => true,
    ]
],
```

## Configure your Common Models

We must update the connection of each of the models that are related to the common database. Just set the connection property with the name of the connection.

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable {
    // All your traits

    /**
     * The database associated with the model.
     */
    protected $connection = 'permissions';

    // The rest of your code.
}
```

You can do this in your migration classes too.

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    /**
     * Run the migrations.
     */
    public function up(): void {
        Schema::create('orders', function (Blueprint $table) {
            // Your other columns 
            $table->foreignId('created_by')->nullable()->constrained(config('database.connections.permissions.database') . '.users')->nullOnDelete();
            $table->foreignId('updated_by')->nullable()->constrained(config('database.connections.permissions.database') . '.users')->nullOnDelete();
        });
    }
};
```

If you used validation rules with the common models you have to update them with the connection name.

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest {
    public function authorize(): bool {
        return true;
    }

    public function rules(): array {
        $rules = request()->isMethod('post') ?
            ['name' => 'required|max:65|unique:permissions.users']
            : ['name' => "required|max:65|unique:permissions.users,name,$this->id"];

        return $rules;
    }
}
```

## Clear Cache

You can clear the cache of the applications with the command:

```powershell
php artisan optimize:clear
```

We also recommend clearing your browser's cache.