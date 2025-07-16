### Libraries vs Frameworks
---

-   **Library**: A set of functions used within your project (e.g., React). You call the library code.
-   **Framework**: A boilerplate that provides a pre-built structure for a project (e.g., Laravel). The framework calls your code. It enforces a way of building things.

| Category       | Library | Framework   |
| -------------- | ------- | ----------- |
| **JavaScript** | React   | Next.js     |
| **PHP**        |         | **Laravel** |

_Note: Laravel itself is built upon powerful, standalone PHP components from the Symfony framework._

## Folder Structure
---

```
/ (root)
├── app/                  // Core application code
│   ├── Console/
│   ├── Exceptions/
│   ├── Http/
│   │   ├── Controllers/  // Handles request logic
│   │   └── Middleware/
│   ├── Models/           // Represents database tables (Eloquent ORM)
│   └── Providers/
│
├── bootstrap/
│   └── app.php           // Initializes the framework
│
├── config/
│   └── *.php             // Package & Application Configuration
│
├── database/
│   ├── factories/        // Define blueprints for test data
│   ├── migrations/       // Version control for your database schema
│   └── seeders/          // Populate the database with initial data
│
├── public/
│   └── index.php         // The single entry point for all HTTP requests
│
├── resources/
│   ├── js/
│   ├── css/
│   └── views/            // Contains Blade templates for Server-Side Rendering (SSR)
│       └── welcome.blade.php
│
├── routes/
│   ├── api.php           // For separate react/vue frontend
│   ├── channels.php
│   ├── console.php
│   └── web.php           // For web UIs using sessions & Blade views (SSR)
│
├── storage/              // For generated files, logs, and user uploads
│   ├── app/              // Private files (e.g., user contracts)
│   ├── framework/
│   └── logs/
│
├── tests/
│
├── vendor/               // Composer dependencies (don't touch!)
│
├── .env                  // Environment-specific configuration (SECRET!)
├── artisan               // The Laravel command-line tool
├── composer.json         // PHP package dependencies
└── package.json          // JS package dependencies
```

### SSR vs SPA
---

-   **SSR (Server-Side Rendering)**: The server renders the full HTML for each request and sends it to the browser. Laravel's Blade templates are used for this.
    -   **Pros**: Great for SEO.
    -   **Cons**: Higher server load.
-   **SPA (Single-Page Application)**: The app loads a single HTML shell, and JavaScript (e.g., React) fetches data via API calls and updates the page content dynamically.
    -   **Pros**: Faster user experience after initial load, reduced server load (server only sends data, not HTML).
    -   **Cons**: Can be more complex to set up for SEO.

### .env (Environment File)
---

This is one of the most critical files. It stores all environment-specific and sensitive configuration for your application.

-   **Purpose**: Holds variables that change depending on the environment (local, staging, production).
-   **Examples**: Database credentials (`DB_HOST`, `DB_USERNAME`, `DB_PASSWORD`), App URL, and secret API keys (e.g., Open AI, AWS).
-   **Security**: **NEVER commit the `.env` file to version control (e.g., Git).** It contains sensitive secrets. The `.gitignore` file that comes with Laravel already prevents this.
-   **Workflow**:
    1.  The `.env.example` file is a template that *is* committed to Git.
    2.  When a new developer joins the project, they copy `.env.example` to a new file named `.env`.
    3.  They then fill in their own local credentials (e.g., their local database password) into their personal `.env` file.

### MVC (Model-View-Controller)
---

MVC is a design pattern that separates application logic into three interconnected parts. This separation of concerns makes the code cleaner, more organized, and easier to maintain.

1.  **Route**: A request first hits a URL defined in a route file (`routes/web.php` or `routes/api.php`).
2.  **Controller**: The route forwards the request to a specific method on a Controller. The controller processes the request input, asks the Model for necessary data, and decides what to do next.
3.  **Model**: The Model interacts directly with the database. It represents a single database table (e.g., a `User` model maps to the `users` table) and handles all database queries (Create, Read, Update, Delete) using the Eloquent ORM.
4.  **View / Response**: After the Controller gets the data from the Model, it passes this data to a **View** (a Blade template) to be rendered as HTML (for SSR). For an API, it formats the data as **JSON** and returns it as a response.

**Example Flow (API):**

`GET /api/users/1` -> `routes/api.php` -> `UserController@show()` -> `User::find(1)` -> `return response()->json($user)`

#### **MVC Code Examples:**

**1. Route (`routes/web.php`)**

This route defines that when a GET request comes to `/users`, it should be handled by the `index` method of the `UserController`.

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\UserController; // Make sure to import your controller

Route::get('/users', [UserController::class, 'index']);
```

**2. Controller (`app/Http/Controllers/UserController.php`)**

The `UserController`'s `index` method fetches all users from the database using the `User` Model and then passes them to a Blade view named `users.index` to display them.

```php
<?php

namespace App\Http\Controllers;

use App\Models\User; // Import the User Model
use Illuminate\Http\Request;

class UserController extends Controller
{
    // Display list of users
    public function index()
    {
        $users = User::all(); // Fetch all users using the User Model
        return view('users.index', ['users' => $users]); // Pass data to the view
    }
    
    // Store user in database
    public function store(Request $request)
    {
        $validatedData = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users|max:255',
            'password' => 'required|string|min:8',
        ]);

        $user = User::create([ // Create a new user using the User Model
            'name' => $validatedData['name'],
            'email' => $validatedData['email'],
            'password' => bcrypt($validatedData['password']), // Hash the password
        ]);

        return redirect('/users')->with('success', 'User created successfully!');
    }
}
```

`php artisan make:controller`

**3. Model (`app/Models/User.php`)**

The `User` model extends `Illuminate\Database\Eloquent\Model`. By default, it maps to the `users` table in your database. Eloquent automatically provides methods like `all()`, `find()`, `create()`, etc., for interacting with this table. The `$fillable` array specifies which attributes are mass assignable.

`php artisan make:model`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use HasFactory, Notifiable;
     // The attributes that are mass assignable.
    protected $fillable = [
        'name',
        'email',
        'password',
    ];
}
```

### Blade Templating Engine
---

Blade is Laravel's powerful server-side templating engine. It lets you write clean PHP inside your HTML using simple syntax, but it will not be compiled into the final HTML sent to the browser.

##### Old PHP in HTML:

```php
<div>
    <?php foreach($users as $user): ?>
        <p><?php echo $user->name; ?></p>
    <?php endforeach; ?>
</div>
```

##### Blade Syntax:

```php
<div>
    @foreach($users as $user)
        <p>{{ $user->name }}</p>
    @endforeach
</div>
```

### Artisan Console
---

Artisan is Laravel's built-in command-line interface (CLI). It provides a huge number of helpful commands for building and managing your application.

-   `php artisan serve`: Starts a local development server.
-   `php artisan make:controller PostController`: Creates a new controller file in `app/Http/Controllers/`.
-   `php artisan make:model Movie`: Creates a new Eloquent model in `app/Models/`.
-   `php artisan make:migration create_movies_table`: Creates a new database migration file.
-   `php artisan migrate`: Runs all outstanding migrations to update the database schema.
### Application Lifecycle
---

##### Application Lifecycle:

1.  A request hits the single entry point: `public/index.php`.
2.  The framework is bootstrapped in `bootstrap/app.php`, creating the application instance.
3.  The request is sent through the HTTP Kernel, which processes middleware (for tasks like authentication).
4.  The router matches the request URL to a route defined in `routes/web.php` or `routes/api.php`.
5.  The matched route calls a controller method.
6.  The controller interacts with models, services, etc.
7.  The controller returns a Response object (e.g., a rendered Blade view or a JSON response) which is sent back to the browser.