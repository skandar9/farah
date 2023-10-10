Web application, An association for supporting children with cancer. This national, independent, humanitarian, non-governmental, and non-profit organization aims to provide donation-based services and charitable support to children affected by cancer. Additionally, the platform utilizes Telegram to send notifications, ensuring timely updates and communication with supporters. The overarching goal of the web application is to facilitate the organization's efforts in raising awareness, providing essential services, and delivering support to children facing cancer and their families.

ADMIN		1	
2.	EMPLOYEE		1	
3.	DOCTOR		1	
4.	VOLUNTEER		1	
5.	GUARDIAN


Based on the information provided in the files you shared, the project appears to be a web application built using the Laravel framework, a popular PHP framework for web development. The project includes both frontend and API routes, indicating a separation between the user-facing interface and the backend API.

The project seems to be focused on user authentication and authorization, as seen in the routes related to user authentication, signup, and password management. It provides endpoints for user login, signup, password reset via email, and retrieving user information. There are also routes for updating user information, changing passwords, and managing notifications.

The API routes are protected by the 'auth:api' middleware, indicating that authentication is required to access them. This suggests that the project implements token-based authentication for API requests, where users need to authenticate and obtain a token to make authorized API calls.

The project involves different user roles, such as "Guardian" and "Doctor." Guardians have access to routes related to managing requests, transfers, medical profiles, emergency services, and more. Doctors have routes for managing their profiles, updating availability, interacting with patients, responding to requests, and generating reports.

The project also incorporates features like handling nested object validation, managing Docker containers, setting default values for GraphQL input arguments, managing multiple Node.js versions using NVM, installing specific versions of Ruby gems, and understanding algorithmic complexity using "Big O" notation.

Overall, the project demonstrates a comprehensive web application that includes user authentication, role-based access control, API functionality, and various features related to user management, healthcare, and software development. It showcases the usage of Laravel, PHP, Docker, GraphQL, Node.js, Ruby, and ReactJS technologies.


he project is a comprehensive web application developed using the Laravel framework, a powerful PHP framework known for its efficiency and flexibility. It encompasses both frontend and API routes, showcasing a clear separation between the user interface and the backend API.

At its core, the project emphasizes secure user authentication and authorization. It offers a seamless user experience with features like user registration, login, and password management, ensuring that only authorized individuals can access the application's functionalities. The project employs token-based authentication for API requests, granting users access to protected routes by validating their authentication tokens.

With a focus on user management, the project caters to various user roles, most notably "Guardians" and "Doctors." Guardians are provided with a range of features to manage requests, transfers, medical profiles, emergency services, and more. They can interact with the system to store requests, view processed and pending requests, and access vital medical information. Doctors, on the other hand, have dedicated routes to manage their profiles, update availability, communicate with patients, respond to requests, and generate detailed reports.

The project integrates several technologies and best practices to enhance its functionality and performance. It leverages Docker to facilitate containerization and streamline the deployment process, allowing for better scalability and portability. Additionally, the application embraces GraphQL, a query language for APIs, enabling seamless integration with clients and providing efficient data fetching capabilities.

The inclusion of Node.js showcases the project's versatility in managing multiple Node.js versions using NVM (Node Version Manager). This feature allows developers to switch between different Node.js versions effortlessly, ensuring compatibility and flexibility when developing and deploying Node.js applications.

Furthermore, the project demonstrates proficiency in Ruby development, showcasing the ability to install specific versions of Ruby gems. This capability ensures that the project can leverage the desired functionality provided by specific gem versions, enhancing compatibility and maintaining code quality.

An essential aspect of the project is its utilization of algorithmic analysis through "Big O" notation. By understanding algorithmic efficiency, the application optimizes performance and resource utilization, ensuring that operations are executed in the most efficient manner possible.

From a frontend perspective, the project incorporates ReactJS, a popular JavaScript library for building user interfaces. This allows for the creation of dynamic and interactive components, enabling a seamless user experience and responsive design.

Overall, this project serves as a testament to your expertise in web development, showcasing your proficiency in Laravel, PHP, Docker, GraphQL, Node.js, Ruby, and ReactJS. Its focus on user authentication, role-based access control, API development, and integration with various technologies highlights your ability to create robust and versatile web applications. By emphasizing the project's features and technologies in your CV and README.md file, you can effectively communicate your skills in backend and frontend development, API design, database management, and the integration of cutting-edge technologies.

## Contents

- [Authentication](#authentication)
    - [Sign Up](#SignUp)
    - [Update Password](#updatePassword)
  
[Admin routes definition](#backend-routes)

- [Admin middleware](#admin-middleware)
    - [Define the middleware](#define-the-middleware)
    - [Register the middleware](#register-the-middleware)

[Store Request](#store-request)

[pushNotifications (trait function)](#push-notifications)

[Get all patients for specific doctor](#get-all-patients)

[Add video to news](#news-video)

[Store video](#store-video)

### **authentication**

[🔝 Back to contents](#contents)

### **SignUp**

`app\Http\Controllers\Api\AuthController.php`

### constructor method

```php
protected $user_repo;
public function __construct(UserInterface $user_repo)
{
    $this->user_repo=$user_repo;
}
```

- The constructor method takes an argument named `$user_repo` of type `UserInterface`.
- Inside the constructor, the value of the `$user_repo` argument is assigned to the `$user_repo` property using the `$this` keyword. This allows the class to store and access the provided instance of `UserInterface` throughout its methods.


```php
public function signUp(UserRequest $request)
{
    $data = $request->all();
    $new_user = $this->user_repo->store($data);

    $tokenResult = $new_user->createToken('Personal Access Token');
    $token = $tokenResult->token;
    if ($request->remember_me)
        $token->expires_at = Carbon::now()->addMinute(2);
    $token->save();
    return response()->json([
        'access_token' => $tokenResult->accessToken,
        'token_type' => 'Bearer',
        'expires_at' => Carbon::parse(
            $tokenResult->token->expires_at
        )->toDateTimeString()
    ]);
}
```
The `signUp` method in the given code is responsible for creating a new user and generating an access token for the user to authenticate with.

- The `signUp` method receives a `UserRequest` object as a parameter, that is contains the data for creating a new user.
  
- The code calls the `store` method on the `user_repo` repository, passing in the `$data` variable to create and store the new user. The repository is responsible for interacting with the database and handling user-related operations.
  
- After creating the new user, the code generates a personal access token for the user using the `createToken` method on the `$new_user` object. The token is named "Personal Access Token".
  
- The code then retrieves the token from the `$tokenResult` object by accessing the `token` property.
  
- Next, it checks if the "remember me" option is enabled by checking the value of `$request->remember_me`.
  
- If the "remember me" option is enabled, the code sets the token's expiration to be 10 minutes from the current time using the `addMinute` method from the Carbon library.
  
- Finally, the code constructs a JSON response using the `response()->json()` method. It includes the access token (`$tokenResult->accessToken`), the token type (set as 'Bearer'), and the expiration date of the token. The expiration date is parsed using the Carbon library to format it as a string.

[🔝 Back to contents](#contents)

### **updatePassword**

`app\Http\Controllers\Api\AuthController.php`

```php
public function updatePassword(Request $request)
{
    $this->validate($request, [
        'password' => 'required',
        'new_password' => 'different:password',
    ]);
    $user = $this->user_repo->getById(auth()->user()->id);
    if (Hash::check($request->password, $user->password)) {
        $user->fill([
        'password' => Hash::make($request->new_password)
        ])->save();
        $tokens = auth()->user()->tokens;
        foreach($tokens as $token) {
            $token->revoke();
        }
    return response()->json(true);
    }
    return response()->json(false);
}
```

The `updatePassword` method in the provided code snippet is responsible for updating the password of the authenticated user.

- The code calls the `validate` method to validate the incoming request data. It checks that the `password` field is required and the `new_password` field is different from the `password` field. This ensures that the new password is not the same as the current password. 
  
- The code retrieves the authenticated user's details by calling the `getById` method on the `$user_repo` repository and passing in the authenticated user's ID. This assumes that the repository has a method to fetch a user by their ID.
 
- It then uses `Hash::check` to compare the provided password (`$request->password`) with the user's current hashed password (`$user->password`). If they match, it proceeds with updating the password.
  
- Inside the `if` block, the user's password is updated by calling the `fill` method on the user model instance. The `fill` method sets the `password` attribute to the hashed value of the new password (`Hash::make($request->new_password)`). The updated user is then saved to the database using the `save` method.
  
- The code retrieves all the access tokens associated with the authenticated user by accessing the `tokens` property on `auth()->user()`. This assumes that the user model has a relationship with the access tokens.

- It iterates over each token and calls the `revoke` method to revoke the token, effectively invalidating it.
  
- Finally, a JSON response is returned indicating a successful password update by passing `true` to `response()->json()`. If the provided password did not match the user's current password, a JSON response with `false` is returned.

[🔝 Back to contents](#contents)

 ### **backend-routes**

`routes\web.php`

```php
Route::group(['prefix' => 'admin', 'as' => 'admin.', 'middleware' => ['admin']], function () {
    RouteHelper::include_route_files(__DIR__.'/backend/');
});
```

By using this code, I organized my routes related to the admin section under the `/admin` URL prefix, apply the [admin middleware](#admin-middleware) to those routes, and include route files from the specified directory (`__DIR__.'/backend/'`).

1. The `Route::group()` method is used to define a route group. The first parameter is an array that contains the group configuration options.
   - `prefix` specifies the URL prefix for all routes within the group. In this case, the prefix is `'admin'`, so all routes within this group will have URLs starting with `/admin`.
   - `as` specifies the route name prefix. It means that all route names within this group will be prefixed with `'admin.'`. For example, if a route is named `'dashboard'`, it will have the name `'admin.dashboard'`.
   - `middleware` specifies the middleware(s) to be applied to the routes within the group. In this case, the [admin middleware](#admin-middleware) is applied.

2. Inside the route group, the `RouteHelper::include_route_files()` method is called. This method is responsible for including route files from a specific directory. The `__DIR__.'/backend/'` parameter represents the directory path where the route files are located. It's assumed that the `RouteHelper` class has a static method called `include_route_files()` that handles the inclusion of route files.

[🔝 Back to contents](#contents)

### **admin-middleware**

### **define-the-middleware**

`app\Http\Middleware\AdminMiddleware.php`

```php
class AdminMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        if(auth()->check())
        {
            $user_id=auth()->user();
            if ($request->user()->roles->name == "ADMIN" ){
                return $next($request);
            }
            elseif ($request->user()->roles->name == "EMPLOYEE" ){
                return $next($request);
            }
            else
            {
                abort(403);
            }
        }
        else
        {
            abort(404);
        }
    }
}
```

The `AdminMiddleware` class checks if the authenticated user has the role of "ADMIN" or "EMPLOYEE" before allowing access to the requested route. If the user does not have either of these roles, it aborts the request with a status code of 403 (Forbidden).

- If the user is authenticated, it retrieves the authenticated user ID using `auth()->user()` and assigns it to the `$user_id` variable.
  
- It then checks if the user has the role of "ADMIN" or "EMPLOYEE" by accessing the `roles` relationship on the `$request->user()` object and comparing the `name` property with the role names.
  
- If the user has either of the roles, it allows the request to proceed to the next middleware or the requested route by calling `$next($request)`.
  
- If the user does not have the required roles, it aborts the request with a 403 Forbidden status code using the `abort(403)` function.
  
- If the user is not authenticated, it aborts the request with a 404 Not Found status code using the `abort(404)` function.

### **register-the-middleware**

`app\Http\Kernel.php`

```php
class Kernel extends HttpKernel
{
.
.
.

    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
            \App\Http\Middleware\LanguageMiddleware::class,
        ],

        'api' => [
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
        'admin'=>[
            AdminMiddleware::class,
        ]
    ];
}
```

The `'admin'` middleware group is defined with a single middleware, `AdminMiddleware::class`. When this middleware group is applied to a route or route group, the `AdminMiddleware` will be executed for each incoming request.

[🔝 Back to contents](#contents)

### **store-request**

`app\Http\Controllers\Api\GuardianApiController.php`

```php
public function storeRequest(GuardianRequest $request)
{
    $id = auth()->user()->id;
    $data = $request->all();
    $data ['user_id'] = $id;
    if ($data['place'] == 'home')
    {
        if(isset($data['transfer_id']) != null){
            unset($data['transfer_id']);
        }
        $data['type'] = "Medical Counseling";
    }
    else if ($data['place'] == 'other')
    {
        if(isset($data['transfer_id']) != null){
            unset($data['transfer_id']);
        }
        $data['type'] = "Supplies & Others";
    }
    else if ($data['place'] == 'hospital')
    {
        $data['type'] = "Medical Transfer";
    }
    unset($data['image']);
    $guardian_request = $this->guardian_repo->store($data);
    if ($request->has('image'))
    {
        foreach($request->file('image') as $image)
        {
            $image_name = time(). $image->getClientOriginalName();
            $image->move(public_path('dist/img/'), $image_name);
            $guardian_request->photos()->create([
                'type' => 'guardian_request',
                'origin' => 'photo',
                'image' => $image_name
            ]);
        }
    }
    $admin = Role::where('name' , 'ADMIN')->first();
    $user = $this->user_repo->getById($id);
    $ids = [$user->doctors->id , $id];
    $admins = User::where('role_id' ,$admin->id )->get();
    foreach ($admins as $key) {
        $ids [] = $key->id;
    }
    $data['title'] = $guardian_request->type;
    $data['body'] = $guardian_request->description;
    $data['url']=route("admin.guardians.show" , $guardian_request->id);
    $this->pushNotifications($ids , $data);
    return response()->json(true , 200);
}
```

The `storeRequest` method in the provided code snippet is responsible for storing a Guardian request and performing various related actions.

Here's a step-by-step breakdown of how the code works:

- The `storeRequest` method receives a [GuardianRequest](#guardian-request) object as a parameter, which contains data related to the Guardian request.
  
- The code retrieves the ID of the authenticated user using `auth()->user()->id` and stores it in the `$id` variable.
  
- Depending on the value of the `place` field in the `$data` array, the `type` field is set accordingly. If the `place` is "home," the type is set to "Medical Counseling." If the `place` is "other," the type is set to "Supplies & Others." If the `place` is "hospital," the type is set to "Medical Transfer."
  
- The Guardian request is stored using the `store` method of `$guardian_repo` repository, passing in the `$data` array.
  
- If the request contains uploaded images (`$request->has('image')`), the code iterates over each image, moves it to the specified public path, and creates a new record in the `photos` table associated with the `$guardian_request`. The `type` is set to 'guardian_request', the `origin` is set to 'photo', and the `image` field is set to the image filename.
  
- The code retrieves the admin role by searching for the role with the name 'ADMIN' and assigns it to the `$admin` variable.
  
- The user details are retrieved using the `getById` method of `$user_repo` repository, passing in the `$id` of the authenticated user.
  
- An array `$ids` is created with the IDs of the user's doctor (`$user->doctors->id`) and the authenticated user's ID (`$id`).

- The `pushNotifications` method is called, passing in the `$ids` array and the `$data` array to send push notifications to the specified user IDs.
  
[🔝 Back to contents](#contents)

### **guardian-request**

`app\Http\Requests\GuardianRequest.php`

```php
public function rules()
{
    return [
        'place'=>[
            'required',
            'string',
            'in:hospital,home,other'
        ],
        'type'=>[
            'required',
            'string',
            'in:Medical Transfer,Medical Counseling,Supplies & Others',
        ],
        'description'=>[
            'required',
            'string',
        ],
        "image"=>[
            'array',
            'distinct',
        ],
        "image.*"=>[
            "required",
            'image',
            'mimes:jpeg,png,jpg,gif,svg',
            'max:2048',
        ],
        "transfer_id" => [
            'required_if:place,hospital',
        ],
    ];
}
```

This class is responsible for validating the incoming request data before it is processed into["storeRequest" function](#storeRequest).
The rules method defines the validation rules that will be applied to the request data. It returns an array where each key represents a field in the request data, and the corresponding value is an array of validation rules for that field.

[🔝 Back to contents](#contents)

### **pushNotifications**

`app\Traits\notifications.php`

```php
public function pushNotifications($ids , $data)
{
    if (isset($data['url'])){
    $notifications = Notification::create([
        "title"=>$data['title'],
        "body"=>$data['body'],
        "url"=>$data['url'],
    ]);
    }else{
    $notifications = Notification::create([
        "title"=>$data['title'],
        "body"=>$data['body'],
    ]);
    }
    foreach ($ids as $id) {
        $notifications->users()->attach($id);
    }

    $firebaseToken=[];
    foreach ($ids as $id) {
        $user = User::where('id',$id)->whereNotNull('device_token')->pluck('device_token')->first();
        if ($user != null){
            $firebaseToken []=$user;
        }
    }
    $SERVER_API_KEY = 'AAAAGuU0AwQ:APA91bGJISfEAoIhIq_BX-rQpb2nA3pFK5oFXzven5Gy1a72iw9BXcVzpMQBL0IZUjVVZKrFmmkhLESB_LDlyYke_bCp93bCTKbOsbuU1raaOJlCtoFpENqCKMTGyf49li3K4RWrzdMl';
    $data = [
        "registration_ids" => $firebaseToken,
        "notification" => [
            "title" => $data['title'],
            "body" => $data['body'],
            "icon" => asset('LOGO.png'),
            "sound" =>'default',
            "color"=>"#FFC93C",
        ]
    ];
    $dataString = json_encode($data);
    $headers = [
        'Authorization: key=' . $SERVER_API_KEY,
        'Content-Type: application/json',
    ];
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, 'https://fcm.googleapis.com/fcm/send');
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $dataString);
    $response = curl_exec($ch);
}
```

- The function receives an array of user IDs (`$ids`) and the data for the push notification (`$data`).
  
- Depending on whether the `$data` array contains the `url` key, a new record is created in the `Notification` model table with the provided title, body, and URL (if available).
  
- The notification record is attached to the specified user IDs using the `attach` method.
  
- The function retrieves the Firebase tokens for the specified user IDs by querying the `User` model. Only users with a non-null `device_token` field are considered. The tokens are stored in the `$firebaseToken` array.
  
- The data for the FCM request is prepared, including the `registration_ids` (Firebase tokens) and notification details (title, body, icon, sound, color).
  
- The data is converted to a JSON string using `json_encode`.
  
- The headers for the FCM request are set, including the authorization key and content type.
  
- The cURL options for the FCM request are set, including the URL(https://fcm.googleapis.com/fcm/send), HTTP method (POST), headers, SSL verification, return transfer, and the data to be sent.
  
- The cURL request is executed using `curl_exec`, and the response is stored in the `$response` variable.

[🔝 Back to contents](#contents)

### **get-all-patients**

`app\Http\Controllers\Api\DoctorApiController.php`

```php
public function getAllPatients()
{
    $patients = $this->user_repo->getPatients()->where('deceased' , 0);
    foreach ($patients as $key) {
        $key->age = Carbon::parse($key->birth)->diff(\Carbon\Carbon::now())->format('%y');
        $key->ar_place = $key->places->ar_name;
        $key->en_place = $key->places->en_name;
        if ($key->doctor_id != null)
        {
            $key->doctor_name = $key->doctors->first_name . " " . $key->doctors->last_name;
        }
        if($key->cancer_types_id != null)
        {
            $key->cancer_type = $key->cancers->name;
        }else{
            $key->cancer_type = null;
        }
        unset($key->places);
        unset($key->doctors);
    }
    return response()->json($patients , 200);
}
```

The `getAllPatients` function retrieves all patients who are not deceased and performs some additional transformations before returning the response.
The code assumes the existence of relationships between the `User` model (representing patients), the `Place` model, the `Doctor` model, and the `Cancer` model. 

- The function starts by retrieving all patients who are not marked as deceased using the `getPatients` method from the `$user_repo` object.
  
- The function then iterates over each patient using a `foreach` loop.
  Inside the loop, the age of each patient is calculated based on their birth date using the `Carbon` library.

The Arabic and English names of the place associated with the patient are retrieved and assigned to the `$key->ar_place` and `$key->en_place` properties, respectively.

- If the patient is assigned to a doctor (i.e., the `doctor_id` property is not null), the full name of the assigned doctor is concatenated and assigned to the `$key->doctor_name` property.
  
- If the patient has a cancer type assigned (i.e., the `cancer_types_id` property is not null), the name of the assigned cancer type is retrieved and assigned to the `$key->cancer_type` property. Otherwise, the `$key->cancer_type` property is set to null.

- The `places` and `doctors` relationships are removed from the patient object using the `unset` function to avoid including unnecessary data in the response.
   
[🔝 Back to contents](#contents)

### **news-video**

`resources\views\news\index.blade.php`

![App Logo](/images/all-news.png)

```html
.
.
.
    @foreach ($news as $new)
        <tbody>
            <tr>
                <td>{{ $new->id }}.</td>
                <td>
                    <div class="btn-group btn-group-sm">
                        <a class="font-awsome btn-info btn btn-danger"
                            href="{{ route('admin.news.show', $new->id) }}"><i
                                class="nav-icon far fa-images"></i></a>
                    </div>
                    <div class="btn-group btn-group-sm">
                        <a class="font-awsome btn-info btn btn-danger"
                            href="{{ route('admin.news.videos', $new->id) }}"><i
                                class="nav-icon fas fa-film"></i></a>
                    </div>
                </td>
                <td>{{ $new->title }}</td>
                <td>{{ $new->content }}</td>
                <td>
                    <i class="bg-warning"></i>
                </td>
                <td class="text-right">
                    <div class="dropdown">
                        <a class="btn btn-sm btn-icon-only" href="#" role="button"
                            data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                            <i class="fas fa-ellipsis-v" style="color: sandybrown;"></i>
                        </a>
                        <div class="dropdown-menu dropdown-menu-right dropdown-menu-arrow">
                            <a class="dropdown-item"
                                href="{{ route('admin.news.edit', $new->id) }}">{{ __('table.button.edit') }}</a>
                            <form method="POST"
                                action="{{ route('admin.news.destroy', $new->id) }}">
                                @csrf
                                @method('delete')
                                <button
                                    class="dropdown-item">{{ __('table.button.delete') }}</button>
                            </form>
                        </div>
                    </div>
                </td>
            </tr>
        </tbody>
    @endforeach
.
.
.
```

This code generates a table with rows representing news items. It displays various information about each news item and provides options to view associated images and videos, edit the news item, and delete it.

It contains two buttons for viewing images and videos associated with a news item:

- The first button:
   It has an `href` attribute that uses the `route()` helper function to generate a URL for the "admin.news.show" route, which is expected to display images associated with the news item. The news item's ID is passed as a parameter to the route.

- The second button:
   It has an `href` attribute that uses the `route()` helper function to generate a URL for the ![admin.news.videos](display-videos-route) route, which is expected to display videos associated with the news item. The news item's ID is passed as a parameter to the route

### **display-videos-route**

```php
Route::get('/newsVideo/{id}', [Controllers\NewsController::class, 'newsVideo'])->name('news.videos');
```
This line defines a GET route named `'news.videos'` that maps to the `newsVideo` method of the `NewsController` class. It expects a parameter `{id}` in the URL and handles the request to display videos associated with a news item based on its ID.

### **browse-video-to-specific-news**

![App Logo](/images/add-video.png)

`resources\views\news\video.blade.php`


This Blade page appears to be used for displaying a news item and managing its associated videos.

When the "Create" button is clicked (after browsing the video), the form is submitted and the corresponding route (admin.newsVideosStore) is called:

`routes\backend\admin.php`

```php
Route::post('/newsVideosStore', [Controllers\NewsController::class, 'newsVideosStore'])->name('newsVideosStore');
```

### **store-video**

`app\Http\Controllers\NewsController.php`

```php
public function newsVideosStore(Request $request)
{
    $data = $request->all();
    foreach ($request->file('image') as $image) {
        $image_name = time() . $image->getClientOriginalName();
        $image->move(public_path('dist/img/'), $image_name);
        $data['image'] = $image_name;
        $this->news_repo->storeVideos($data);
    }
    return redirect()->back();
}
```

The provided code snippet appears to be a method named `newsVideosStore` within a controller. Here's an explanation of its functionality:

```php
public function newsVideosStore(Request $request)
{
    $data = $request->all();
    foreach ($request->file('image') as $image) {
        $image_name = time() . $image->getClientOriginalName();
        $image->move(public_path('dist/img/'), $image_name);
        $data['image'] = $image_name;
        $this->news_repo->storeVideos($data);
    }
    return redirect()->back();
}
```

This method handles the storage of uploaded news videos by moving them to a designated directory, storing the relevant data in the database, and then redirecting the user back to the previous page.

- It iterates over each uploaded file using a `foreach` loop on `$request->file('image')`. This assumes that the file input field has the name attribute set to `'image[]'`, allowing multiple videos to be uploaded.
- 
- Inside the loop, it generates a unique image name for each file by appending the current timestamp to the original name of the file using `time()` and `$image->getClientOriginalName()`.
  
- It moves the uploaded video file to the `public/dist/img/` directory using the `move()` method, which takes the destination path and the image name as arguments.
  
- It calls the `storeVideos()` method of the `$news_repo` object (that is a repository class) and passes the `$data` array to store the video in the database.

[🔝 Back to contents](#contents)