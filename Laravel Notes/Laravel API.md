### APIs in Laravel:
---

-   **Routes**: Defined in `routes/api.php`.
-   **Prefix**: All routes are automatically prefixed with `/api`. So, `Route::get('/users', ...)` is accessible at `http://your-app.test/api/users`
-   **State**: APIs are **stateless**. They do not use sessions. Authentication is typically handled with tokens (e.g., Laravel Sanctum).
-   **Data Format**: They almost always return data in **JSON** format.

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\UserController; // Make sure to import your controller

Route::get('/users', [UserController::class, 'index']);
```

This route defines that when a GET request comes to `/users`, it should be handled by the `index` method of the `UserController`.