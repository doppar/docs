## Database Seeder
### Introduction
Database seeding is a crucial aspect of web application development, especially when populating your database with sample data for testing or development. Doppar supports database seeding, allowing you to easily generate data using custom seed classes.

Seed classes are stored in the `database/seeders` directory, and you can use them to define the data that should be inserted into your database tables.

### Create New Seeder
To create a new seeder class, run the following command:
```bash
php pool make:seeder UserSeeder
```
This will generate a new UserSeeder class in the `database/seeders` directory, where you can define the logic for seeding the users table with data. Now it will generate a file like.
```php
<?php

namespace Database\Seeders;

use Phaseolies\Database\Migration\Seeder;

class UserSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run(): void
    {
        //
    }
}
```

### Update The Seeder Class
Now you can seed data from this class by updating run method. You can update `run` method like
```php
<?php

namespace Database\Seeders;

use App\Models\User;
use Phaseolies\Database\Migration\Seeder;

class UserSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run(): void
    {
        User::create([
            'name' => fake()->name(),
            'email' => fake()->email(),
            'password' => bcrypt('password')
        ]);
    }
}
```
Remember that `fake()` is a global helper function, so you can use it in your entire application where you want.

### Registering The Seeder Class
In Doppar, the `Database\Seeders\DatabaseSeeder.php` class is the entry point for running all your seeders. Inside this class, you can register other seeder classes that should be executed when running the seeding command.

Here's how you can set up and use the `Database\Seeders\DatabaseSeeder.php` class:
```php
<?php

namespace Database\Seeders;

use Phaseolies\Database\Migration\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        // Register the UserSeeder class
        $this->call(UserSeeder::class);
    }
}
```
In this example, the DatabaseSeeder calls the UserSeeder class within the `run()` method. When you execute `php pool db:seed`, it will run the DatabaseSeeder, which in turn calls UserSeeder to populate the users table.

### Running the Seeder
Once you've created your seeder classes, you can easily run them using the Pool Console commands. There are two primary ways to run your seeders:

To run all seeders and populate your database with data defined in all seeder classes:
```bash
php pool db:seed
```

To run a specific seeder (e.g., UserSeeder) and populate the relevant data for a particular table:
```bash
php pool db:seed UserSeeder
```
This allows you to selectively seed specific data, which can be helpful when testing or when you only need to insert certain records.
