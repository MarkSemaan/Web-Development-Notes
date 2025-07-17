*   **Authentication:** Verifies **who** you are. It confirms your identity (e.g., by checking your username and password).

*   **Authorization:** Determines **what** you are allowed to do. It grants or denies access to specific resources or actions based on your identity and assigned permissions.

**Relationship:** Authentication always happens **before** authorization. You must first prove *who* you are before the system can decide *what* you can access or do.

## JWT
---
**Laravel JWT** refers to using **JSON Web Tokens (JWT)** for **API authentication** in Laravel applications.

Instead of session-based authentication, JWT allows **stateless authentication**, which is ideal for **APIs, SPAs, and mobile apps**.

JWT is a **token-based authentication standard**. It issues a token after a user logs in, which the client sends on each request to verify identity, **without needing sessions or cookies**.

#### Common JWT packages:

1. tymondesigns/jwt-auth (most popular)
2. Laravel Sanctum (not JWT but does provide token based auth with session capablilites)

#### Setup JTW

1. `composer require tymon/jwt-auth`
2. `php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"`
3. `php artisan jwt:secret`
4. Add secret key to `.env`
5. `JWT_SECRET=your_generated_secret`
6. Update Auth Guard in `config/auth.php`
```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

7. Generate AuthController

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Auth;

class AuthController extends Controller
{
    public function login(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (!$token = Auth::guard('api')->attempt($credentials)) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }

        return $this->respondWithToken($token);
    }

    public function me()
    {
        return response()->json(Auth::guard('api')->user());
    }

    public function logout()
    {
        Auth::guard('api')->logout();
        return response()->json(['message' => 'Successfully logged out']);
    }

    public function refresh()
    {
        return $this->respondWithToken(Auth::guard('api')->refresh());
    }

    protected function respondWithToken($token)
    {
        return response()->json([
            'access_token' => $token,
            'token_type' => 'bearer',
            'expires_in' => Auth::guard('api')->factory()->getTTL() * 60
        ]);
    }
}
```

8. Define routes

```php
use App\Http\Controllers\AuthController;

Route::post('/login', [AuthController::class, 'login']);

Route::middleware('auth:api')->group(function () {
    Route::get('/me', [AuthController::class, 'me']);
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::post('/refresh', [AuthController::class, 'refresh']);
});
```

9. Make sure User Model uses JWT

```php
use Tymon\JWTAuth\Contracts\JWTSubject;

class User extends Authenticatable implements JWTSubject
{
    // ...

    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    public function getJWTCustomClaims()
    {
        return [];
    }
}
```