**Services** are classes that contain **business logic** or **functionality that doesn't naturally belong to controllers, models, or other core layers**.

They help you write clean, maintainable code by following the **Single Responsibility Principle (SRP)** — separating concerns into distinct, reusable components.

##### Example Service
```php
namespace App\Services;

use App\Models\User;
use Illuminate\Support\Facades\Notification;

class NotificationService
{
    public function sendWelcomeNotification(User $user)
    {
        // Custom notification logic
        Notification::send($user, new \App\Notifications\WelcomeNotification());
    }
}
```

**Use it in controller**: 
```php
namespace App\Http\Controllers;

use App\Models\User;
use App\Services\NotificationService;

class UserController extends Controller
{
    protected $notificationService;

    public function __construct(NotificationService $notificationService)
    {
        $this->notificationService = $notificationService;
    }

    public function store()
    {
        $user = User::create([
            'name' => 'John Doe',
            'email' => 'john@example.com',
        ]);

        $this->notificationService->sendWelcomeNotification($user);

        return response()->json(['message' => 'User created and notified!']);
    }
}

```