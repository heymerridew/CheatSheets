# Laravel Migrations Naming Convention

[Migrations](https://laravel.com/docs/10.x/migrations) are used to define and modify the structure of your database tables. To maintain consistency and organization in your project, it's important to follow a naming convention for your migration files.

## Create a Table

When making a new database table, the migration file should be named in a way that clearly indicates what the table is intended to be used for. The name needs to be plural and written in snake_case.

```powershell
php artisan make:migration create_users_table
```

## Update a Table

If you are making changes to an already-existing table (adding columns, changing the structure, etc.), the migration file name needs to express the reason for the change.

```powershell
php artisan make:migration add_two_factor_columns_to_users_table
php artisan make:migration update_pieces_to_tasks_table
php artisan make:migration remove_pallets_to_sales_table
```
## Drop a Table

To delete a table, you need to create a new migration for the table removal. 

```powershell
php artisan make:migration drop_users_table 
```

## Pivot Tables

For pivot tables used in many-to-many relationships, the naming convention typically involves using the singular names of both related models in alphabetical order, separated by an underscore.

```powershell
php artisan make:migration create_post_user_table
```

It's important to note that the timestamp in the migration file name helps Laravel to determine the order in which migrations should be executed. Migrations are executed in the order of their timestamps, with earlier migrations being executed first.