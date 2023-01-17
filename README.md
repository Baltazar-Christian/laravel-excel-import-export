Steps for Laravel 9 Import Export Excel & CSV File to Database Example:

    Step 1: Installing fresh new Laravel 9 Application
    Step 2: Creating Database and .env Configuration
    Step 3: Installing maatwebsite/excel Package
    Step 4: Creating Dummy Records
    Step 5: Creating Import Class
    Step 6: Creating Export Class
    Step 7: Creating Controller
    Step 8: Creating Routes
    Step 9: Creating Blade File
    Step 10: Testing
    Step 11: Conclusion

Also Read: How to Upload Image in Laravel 9?
Step 1: Installing fresh new Laravel 9 Application

Firstly, we are going to install a fresh new Laravel 9 Application. To install a laravel 9 application run the following code in terminal.

composer create-project laravel/laravel example-app
cd example-app

Note: “example-app” is the our application name.
Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example
Step 2: Creating Database and .env Configuration

After installing our laravel 9 application. we need to create our database so create a new database in phpmyadmin with a name “example-app” as show in the below image.
Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example

Now we have to update database details to our .env file of laravel application. So update the database as shown below.

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=example-app
DB_USERNAME=root
DB_PASSWORD=

Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example

Also Read: Laravel 9 Form Validation With Error Messages
Step 3: Installing maatwebsite/excel Package

After database configuration, we are going to install a maatwebsite/excel package. To install the package run the following command in terminal.

composer require psr/simple-cache:^1.0 maatwebsite/excel

If, you are using less the laravel 9 versions then use below command:

composer require maatwebsite/excel

Now run the migration:

php artisan migrate

Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example

Also Read: Laravel 9 Remove Public from URL using htaccess
Step 4: Creating Dummy Records

In this step, we will create some dummy records for users table, so we can export them with that users. so let’s run bellow tinker command:

php artisan tinker
User::factory()->count(10)->create()

Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example
Step 5: Creating Import Class

In maatwebsite 3 version provide way to built import class and we have to use in controller. So it would be great way to create new Import class. So you have to run following command and change following code on that file:

php artisan make:import UsersImport --model=User

app/Imports/UsersImport.php

<?php
namespace App\Imports;
use App\Models\User;
use Maatwebsite\Excel\Concerns\ToModel;
use Maatwebsite\Excel\Concerns\WithHeadingRow;
use Hash;
class UsersImport implements ToModel, WithHeadingRow
{
    /**
    * @param array $row
    *
    * @return \Illuminate\Database\Eloquent\Model|null
    */
    public function model(array $row)
    {
        return new User([
            'name'     => $row['name'],
            'email'    => $row['email'], 
            'password' => Hash::make($row['password']),
        ]);
    }
}

You can download demo csv file from here: Demo CSV File.

Also Read: Laravel 9 Get env Variable in Blade File Example
Step 6: Creating Export Class

The maatwebsite 3 version provide way to built export class and we have to use in controller. So it would be great way to create new Export class. So you have to run following command and change following code on that file:

php artisan make:export UsersExport --model=User

app/Exports/UsersExport.php

<?php
namespace App\Exports;
use App\Models\User;
use Maatwebsite\Excel\Concerns\FromCollection;
use Maatwebsite\Excel\Concerns\WithHeadings;
class UsersExport implements FromCollection, WithHeadings
{
    /**
    * @return \Illuminate\Support\Collection
    */
    public function collection()
    {
        return User::select("id", "name", "email")->get();
    }
    /**
     * Write code on Method
     *
     * @return response()
     */
    public function headings(): array
    {
        return ["ID", "Name", "Email"];
    }
}

Also Read: How to Use Inner Join In Laravel 9
Step 7: Creating Controller

In this step, we will create UserController with index(), export() and import() method. so first let’s create controller by following command and update code on it.

php artisan make:controller UserController

app/Http/Controllers/UserController.php

<?phpnamespace App\Http\Controllers;use Illuminate\Http\Request;
use App\Exports\UsersExport;
use App\Imports\UsersImport;
use Maatwebsite\Excel\Facades\Excel;
use App\Models\User;class UserController extends Controller
{
    /**
    * @return \Illuminate\Support\Collection
    */
    public function index()
    {
        $users = User::get();
  
        return view('users', compact('users'));
    }
        
    /**
    * @return \Illuminate\Support\Collection
    */
    public function export() 
    {
        return Excel::download(new UsersExport, 'users.xlsx');
    }
       
    /**
    * @return \Illuminate\Support\Collection
    */
    public function import() 
    {
        Excel::import(new UsersImport,request()->file('file'));
        return back();
    }
}

Step 8: Creating Routes

In this step, we need to create routes for list of users, import users and export users. so open your “routes/web.php” file and add following route.

routes/web.php

<?phpuse Illuminate\Support\Facades\Route;use App\Http\Controllers\UserController;/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| contains the "web" middleware group. Now create something great!
|
*/Route::get('/', function () {
    return view('welcome');
});Route::controller(UserController::class)->group(function(){
    Route::get('users', 'index');
    Route::get('users-export', 'export')->name('users.export');
    Route::post('users-import', 'import')->name('users.import');
});

Also Read: Laravel 9 User Roles and Permissions Tutorial Example
Step 9: Creating Blade File

In Last step, let’s create users.blade.php (resources/views/users.blade.php) for layout and we will write design code here and put following code:

resources/views/users.blade.php

<!DOCTYPE html>
<html>
<head>
    <title>Laravel 9 Import Export Excel & CSV File to Database Example - LaravelTuts.com</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
     
<div class="container">
    <div class="card mt-3 mb-3">
        <div class="card-header text-center">
            <h4>Laravel 9 Import Export Excel & CSV File to Database Example - LaravelTuts.com</h4>
        </div>
        <div class="card-body">
            <form action="{{ route('users.import') }}" method="POST" enctype="multipart/form-data">
                @csrf
                <input type="file" name="file" class="form-control">
                <br>
                <button class="btn btn-primary">Import User Data</button>
            </form>
  
            <table class="table table-bordered mt-3">
                <tr>
                    <th colspan="3">
                        List Of Users
                        <a class="btn btn-danger float-end" href="{{ route('users.export') }}">Export User Data</a>
                    </th>
                </tr>
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                </tr>
                @foreach($users as $user)
                <tr>
                    <td>{{ $user->id }}</td>
                    <td>{{ $user->name }}</td>
                    <td>{{ $user->email }}</td>
                </tr>
                @endforeach
            </table>
  
        </div>
    </div>
</div>
     
</body>
</html>

Also Read: Laravel 9 Vue JS Form Validation Example
Step 10: Testing

All steps have been done, now you have to type the given command and hit enter to run the laravel app:

php artisan serve

Now, you have to open web browser, type the given URL and view the app output:

http://127.0.0.1:8000/users

Previews:
Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example
Laravel 9 Import Export Excel & CSV File to Database Example
