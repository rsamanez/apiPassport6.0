<p align="center"><img src="https://res.cloudinary.com/dtfbvvkyp/image/upload/v1566331377/laravel-logolockup-cmyk-red.svg" width="400"></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/d/total.svg" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/v/stable.svg" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/license.svg" alt="License"></a>
</p>

## Create REST API with Passport authentication Laravel 6
```
composer create-project --prefer-dist laravel/laravel apiPassport6.0 "6.*"
composer require laravel/passport
```
Configure your Database access   
Sample SqLite3
```
// change in .env
DB_CONNECTION=sqlite

// add database file
touch database/database.sqlite

```
register providers. Open config/**app.php** . and put the bellow code :
```
'providers' => [
     ....
     /*
      * Package Service Providers...
      */
     Laravel\Passport\PassportServiceProvider::class,
     .....
```
app/providers/**AppServiceProvider.php** and put the two line of code inside a boot method :
```
namespace App\Providers;
Use Schema;
     .....
     public function boot()
    {
        Schema::defaultStringLength(191);
    }
```
Run Migration and Install Laravel Passport
```
php artisan migrate
php artisan passport:install
```
### Laravel Passport Configuration
Next open app/**User.php** file and put the below code
```
namespace App;
use Laravel\Passport\HasApiTokens;
    ....

class User extends Authenticatable
{
    use HasApiTokens,Notifiable;
    
 .....
```
Next Register passport routes in App/Providers/**AuthServiceProvider.php**
```
namespace App\Providers;
use Laravel\Passport\Passport;
    .....
    public function boot() 
    { 
        $this->registerPolicies(); 
        Passport::routes(); 
    }
    ......
```
Next goto config/**auth.php** and Change the api driver to session to passport
```
   .....
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
            'hash' => false,
        ],
    ],
   .....
```
Create API Routes.  
goto routes/**api.php** and create below routes here 
```
Route::prefix('v1')->group(function(){
    Route::post('login', 'Api\AuthController@login');
    Route::post('register', 'Api\AuthController@register');
    Route::group(['middleware' => 'auth:api'], function() {
        Route::get('getUser', 'Api\AuthController@getUser');
        Route::get('listUser', 'Api\AuthController@listUser');

    });
});
.....
```
Create an API controller
```
php artisan make:controller Api/AuthController
```
create methods in app/Http/Controllers/Api/**AuthController.php**
```
<?php
namespace App\Http\Controllers\Api;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use App\User;
use Illuminate\Support\Facades\Auth;
use Validator;

class AuthController extends Controller
{
    public $successStatus = 200;

    public function register(Request $request) {
        $validator = Validator::make($request->all(),
            [
                'name' => 'required',
                'email' => 'required|email',
                'password' => 'required',
                'c_password' => 'required|same:password',
            ]);
        if ($validator->fails()) {
            return response()->json(['error'=>$validator->errors()], 401);                        }
        $input = $request->all();
        $input['password'] = bcrypt($input['password']);
        $user = User::create($input);
        $success['token'] =  $user->createToken('AppName')->accessToken;
        return response()->json(['success'=>$success], $this->successStatus);
    }


    public function login(){
        if(Auth::attempt(['email' => request('email'), 'password' => request('password')])){
            $user = Auth::user();
            $success['token'] =  $user->createToken('AppName')-> accessToken;
            return response()->json(['success' => $success], $this-> successStatus);
        } else{
            return response()->json(['error'=>'Unauthorised'], 401);
        }
    }

    public function getUser() {
        $user = Auth::user();
        return response()->json(['success' => $user], $this->successStatus);
    }

    public function listUser(){
        $users = User::all();
        return response()->json(['success' => $users], $this->successStatus);
    }
}

```
Test your API using the Postman Collection.  
Postman/apiPassport6.0.postman_collection.json
```
php artisan serve
```


## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
