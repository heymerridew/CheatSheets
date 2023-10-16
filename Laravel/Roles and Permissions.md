# Laravel Roles and Permissions

Roles and permissions are essential to create different types of users with specific roles and permissions.

Within an application, a user role denotes a particular set of duties or responsibilities given to a subset of users. Organizations can streamline their workflow by controlling the actions that each user is allowed to perform within the system through the assignment of user roles.

## Install Laravel Permission package

1. We need to install the Spatie package [laravel-permission](https://spatie.be/docs/laravel-permission/v5/installation-laravel), which will allow us to manage user permissions and roles in a database.

    ```
    composer require spatie/laravel-permission
    ```

2. The next step is to publish the migration and configuration file. You can do this by running the following command:
    ```powershell
    php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
    ```

3. After the config and migration have been published and configured, we should execute the migrate artisan command:

    ```powershell
    php artisan migrate
    ```

4. Finally, we need to add the `HasRoles` trait to the user model.

    ```php
    <?php
    // The rest of your imports...
    use Spatie\Permission\Traits\HasRoles;

    class User extends Authenticatable {
        use HasRoles; 
        // The rest of your code...
    }
    ```

5. If you want a "Super User" role to respond true to all permissions, without needing to assign all those permissions to a role, you can use Laravel's Gate::before() method.

    ```php
    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider {
        protected $policies = [];

        public function boot(): void {
            // Implicitly grant "Super Admin" role all permissions
            Gate::before(function ($user, $ability) {
                return $user->hasRole('Super Admin') ? true : null;
            });
        }
    }
    ```

## Install Vue Zo package
1. In order to use roles and permissions in the components we need to install [vue-zo](https://github.com/thonymg/vue-zo).

    ```powershell
    npm install vue-zo --save-dev
    ```

2. We need to import and add the plugin in app.js file.

    ```javascript
    import { VueZo } from 'vue-zo';

    createInertiaApp({
      setup({ el, App, props, plugin }) {
        return createApp({ render: () => h(App, props) })
          .use(VueZo)
          .mount(el);
      }
    });
    ```

## Inertia.js Configuration

1. In order to use the roles and permissions we must make a modification to the `HandleInertiaRequests.php` middleware within the `share` method.

    ```php
    public function share(Request $request): array {
        return parent::share($request) + [
            'authorization' => auth()->user() ? [
                'roles' => auth()->user()->getRoleNames(),
                'permissions' => auth()->user()->getAllPermissions()->pluck('name'),
            ] : null,
        ];
    }
    ```

2. The next step is to create a custom plugin, we will call it `Permissions.js` and place it in the `resources\js\Plugins` path.

    > With the props that we obtain from the middleware we initiate the variables of the vue-zo plugin, with this all the roles and permissions will be available in the application.

    ```javascript
    import { usePage } from '@inertiajs/vue3';

    export default {
      install: (app) => {
        app.mixin({
          mounted() {
            const authorization = usePage().props.authorization;

            if (authorization  !== null) {
              const { roles, permissions } = authorization;
              this.$zo.setRoles(roles);
              this.$zo.setPermissions(permissions);
            }
          },
        });
      },
    };
    ```

3. We must register the plugin we have created to be able to use it globally in the `app.js` file.

    > If you have a Super User role you can add it here.

    ```javascript
    // The rest of your imports
    import { VueZo } from 'vue-zo';
    import Permissions from './Plugins/Permissions';

    createInertiaApp({
      setup({ el, App, props, plugin }) {
        return createApp({ render: () => h(App, props) })
          .use(VueZo, { superRole: 'Super Admin' })
          .use(Permissions)
          // The rest of your code
          .mount(el);
      }
    });

    ```

With the changes we made above we can now use roles and permissions in our components. In a vue component we will use the [directives](https://github.com/thonymg/vue-zo/blob/master/docs/usage/directives.md) provided by the Vue Zo package.

```javascript
<template>
  <button v-can="'add articles'">Add Article</button>
  <button v-permission:any="'add articles|edit articles'">Configure</button>
</template>
```