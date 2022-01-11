Laravel 8 REST API with Passport Authentication

Follow the following steps and create api rest with laravel 8 passport authentication:

Step 1: Download Laravel 8 App
Step 2: Database Configuration
Step 3: Install Passport Auth
Step 4: Passport Configuration
Step 5: Run Migration
Step 6: Create APIs Route
Step 7: Create Passport Auth Controller
Step 8: Now Test Laravel REST API in Postman

Step 1: Download Laravel 8 App
composer create-project --prefer-dist laravel/laravel LaraRestPassport

Step 2: Database Configuration

 DB_CONNECTION=mysql 
 DB_HOST=127.0.0.1 
 DB_PORT=3306 
 DB_DATABASE=here your database name here
 DB_USERNAME=here database username here
 DB_PASSWORD=here database password here


 Step 3: Install Passport Auth

 composer require laravel/passport

 After successfully install laravel passport, register providers. Open config/app.php . and put the bellow code :

   // config/app.php

'providers' =>[
 Laravel\Passport\PassportServiceProvider::class,
 ],

Step 5: Run Migration

 php artisan migrate

 Now, you need to install laravel to generate passport encryption keys. This command will create the encryption keys needed to generate secure access tokens:

 php artisan passport:install

 Step 4: Passport Configuration

 In this step, Navigate to App/Models directory and open User.php file. Then update the following code into User.php:

 <?php
 
namespace App\Models;
 
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;
 
class User extends Authenticatable
{
    use Notifiable, HasApiTokens;
 
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password',
    ];
 
    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];
 
    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];
}

Next Register passport routes in App/Providers/AuthServiceProvider.php, Go to App/Providers/AuthServiceProvider.php and update this line => Register Passport::routes(); inside of boot method:
<?php
namespace App\Providers;
use Laravel\Passport\Passport;
use Illuminate\Support\Facades\Gate;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
 
 
class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\Model' => 'App\Policies\ModelPolicy',
    ];
 
 
    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();
    }
}

Next, Navigate to config/auth.php and open auth.php file. Then Change the API driver to the session to passport. Put this code ‘driver’ => ‘passport’, in API :

  [ 
         'web' => [ 
             'driver' => 'session', 
             'provider' => 'users', 
         ], 
         'api' => [ 
             'driver' => 'passport', 
             'provider' => 'users', 
         ], 
     ],

Step 6: Create APIs Route

In this step, you need to create rest API routes for laravel restful authentication apis with passport project.

So, navigate to the routes directory and open api.php. Then update the following routes into api.php file:

use App\Http\Controllers\API\PassportAuthController;
 
Route::post('register', [PassportAuthController::class, 'register']);
Route::post('login', [PassportAuthController::class, 'login']);
  
Route::middleware('auth:api')->group(function () {
    Route::get('get-user', [PassportAuthController::class, 'userInfo']);
});

Step 7: Create Passport Auth Controller

 php artisan make:controller Api\PassportAuthController


 After that, you need to create some methods in PassportAuthController.php. So navigate to app/http/controllers/API directory and open PassportAuthController.php file. After that, update the following methods into your PassportAuthController.php file:

 
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
<?php
 
namespace App\Http\Controllers\API;
 
use Illuminate\Http\Request;
 
use App\Models\User;
 
class AuthController extends Controller
{
    /**
     * Registration Req
     */
    public function register(Request $request)
    {
        $this->validate($request, [
            'name' => 'required|min:4',
            'email' => 'required|email',
            'password' => 'required|min:8',
        ]);
  
        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => bcrypt($request->password)
        ]);
  
        $token = $user->createToken('Laravel8PassportAuth')->accessToken;
  
        return response()->json(['token' => $token], 200);
    }
  
    /**
     * Login Req
     */
    public function login(Request $request)
    {
        $data = [
            'email' => $request->email,
            'password' => $request->password
        ];
  
        if (auth()->attempt($data)) {
            $token = auth()->user()->createToken('Laravel8PassportAuth')->accessToken;
            return response()->json(['token' => $token], 200);
        } else {
            return response()->json(['error' => 'Unauthorised'], 401);
        }
    }
 
    public function userInfo() 
    {
 
     $user = auth()->user();
      
     return response()->json(['user' => $user], 200);
 
    }
}
