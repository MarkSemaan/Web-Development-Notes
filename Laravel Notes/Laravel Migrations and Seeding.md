### Database Migrations
---

Migrations are like version control for your database. They allow you to define and modify your database schema in PHP files, making it easy to share and recreate the database structure across different development machines.

-   **Location**: `database/migrations/`
-   **How it works**: Each migration file contains an `up()` method (to apply changes, like creating a table) and a `down()` method (to reverse the changes).
-   **Tracking**: Laravel uses a special `migrations` table in your database to keep track of which migration files have already been run.
-   **Command**: `php artisan migrate` executes the `up()` method of all migrations that haven't run yet.
-  **Making a migration**: `php artisan make:migration` creates a migration, automatically includes `up()` method with a primary key `id`  and time `timestamps`
-  **Creating columns**: You can make columns by writing `$table->type("name")`
#### Example Migration

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id(); // Auto-incrementing primary key
            $table->string('name'); // String column for product name
            $table->text('description')->nullable(); // Text column for description, can be null
            $table->decimal('price', 8, 2); // Decimal column for price with 8 digits in total and 2 after the decimal point
            $table->timestamps(); // Adds created_at and updated_at columns
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('products'); // Drops the 'products' table if it exists
    }
};
```

#### Charbel's Opinion
##### Preferably do not use `references()`

```php
public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->integer("category_id")->references("product_categories");
            $table->string('name');
            $table->text('description')->nullable();
            $table->decimal('price', 8, 2);
            $table->timestamps();
        });
        
        Schema::create('product_categories', function (Blueprint $table) {
            $table->id();
            $table->string('name');
        });
    }
```

- References do not create the joins
- Adds unnecessary complexity

##### Migrations -> Models -> Controller
- Preferred order of creation
- Do not make all 3 in 1 command

##### Column Order
1. id (primary key)
2. other_table_id (foreign key)
3. Everything else
### Database Seeding
---

Database seeding is a crucial feature in Laravel that allows developers to populate their databases with initial or test data programmatically. This is invaluable for setting up a development environment, running tests, or populating initial data for a production application.

-   **Location**: Seeder classes are stored in the `database/seeders` directory. By default, a `DatabaseSeeder` class is provided.
-   **How it works**: A seeder class contains a `run()` method where you define the data to be inserted into the database. You can use Laravel's Query Builder or Eloquent model factories to insert data.
-   **Creating a Seeder**: Use the Artisan command `php artisan make:seeder <SeederName>TableSeeder` (e.g., `php artisan make:seeder UsersTableSeeder`).
-   **Running Seeders**:
    -   To run a specific seeder: `php artisan db:seed --class=UsersTableSeeder`.
    -   To run all seeders (by calling them from `DatabaseSeeder`'s `run()` method): `php artisan db:seed`.
    -   You can also run all seeders along with migrations using `php artisan migrate:fresh --seed` (this drops all tables, re-migrates, then seeds).

##### Example Seeder (With Factory)
```php
public function run(): void
{
    \App\Models\User::factory(10)->create();
}
```

And then run it with
```bash
php artisan db:seed
```
### Factories
---

-  Tools used to generate fake data for models, typically for testing and seeding the database using the faker library. 
-  **Faker**:  Generates fake but realistic data.

##### Example Factory
```php
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => $this->faker->name(),
            'email' => $this->faker->unique()->safeEmail(),
            'password' => bcrypt('password'),
        ];
    }
}
```

You call the library with
```php
User::factory()->count(10)->create();
```
