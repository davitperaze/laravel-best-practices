![Laravel-ის საუკეთესო პრაქტიკები](/images/logo-english.png?raw=true)

Translations:

[한국어](https://github.com/xotrs/laravel-best-practices) (by [cherrypick](https://github.com/xotrs))

[English](english.md)

[Русский](russian.md)

[Português](https://github.com/jonaselan/laravel-best-practices) (by [jonaselan](https://github.com/jonaselan))

[Tiếng Việt](https://chungnguyen.xyz/posts/code-laravel-lam-sao-cho-chuan) (by [Chung Nguyễn](https://github.com/nguyentranchung))


ეს არ არის SOLID პრინციპების, პატერნების თუ სხვა დანარჩენის ადაპტაცია Laravel-ისთვის. აქ თავმოყრილია ის, რასაც ხშირ შემთხვევაში, Laravel-ის პროექტებზე მუშაობისას დეველოპერები თავს არიდებენ, თუმცა გამოყენება საუკეთესო შედეგს იძლევა.

<a name="სარჩევი"></a>

## სარჩევი

[ერთი პასუხისმგებლობის პრინციპი (Single responsibility principle)](#ერთი-პასუხისმგებლობის-პრინციპი-(Single-responsibility-principle))

[მძიმე მოდელები, მსუბუქი კონტროლერები](#მძიმე-მოდელები,-მსუბუქი-კონტროლერები)

[ვალიდაცია](#ვალიდაცია)

[ბიზნეს ლოგიკა სერვისის კლასებში](#ბიზნეს-ლოგიკა-სერვისის-კლასებში)

[ნუ გამეორდები (DRY)](#ნუ-გამეორდები-dry)

[ამჯობინე Eloquent-ის გამოყენება Query Builder-ისა და SQL მოთხოვნებს. ამჯობინე კოლექციები მასივებს](#prefer-to-use-eloquent-over-using-query-builder-and-raw-sql-queries-prefer-collections-over-arrays)

[გამოიყენე მასობრივი შევსება (mass assignment)](#mass-assignment)

[არ შეასრულო მოთხოვნები Blade ფაილებში და გამოიყენე სრული ჩატვირთვა (პრობლემა N + 1)](#do-not-execute-queries-in-blade-templates-and-use-eager-loading-n--1-problem)

[დაწერე კომენტარები, თუმცა ამჯობინე აღწერილობითი სახელები მეთოდებისთვის და ცვლადებისთვის](#comment-your-code-but-prefer-descriptive-method-and-variable-names-over-comments)

[არ გამოიყენო JS და CSS თარგებში (template) და არ გამოიყენო HTML-ი PHP კლასებში](#do-not-put-js-and-css-in-blade-templates-and-do-not-put-any-html-in-php-classes)

[კოდში ტექსტის ნაცვლად გამოიყენე კონფიგურაციის და ენების ფაილები, და მუდმივები](#use-config-and-language-files-constants-instead-of-text-in-the-code)

[გამოიყენე საზოგადოების მიერ მიღებული Laravel-ის ინსტრუმენტები](#use-standard-laravel-tools-accepted-by-community)

[მიყევი Laravel-ის სახელდების შეთანხმებებს](#follow-laravel-naming-conventions)

[გამოიყენე მოკლე და კითხვადი სინტაქტი, სადაც შესაძლებელია](#use-shorter-and-more-readable-syntax-where-possible)

[გამოიყენე IoC კონტეინერი ან ფასადები new Class-ის ნაცვლად](#use-ioc-container-or-facades-instead-of-new-class)

[პირდაპირ ნუ გამოიყენებ მონაცემებს `.env` ფაილიდან](#do-not-get-data-from-the-env-file-directly)

[შეინახე თარიღები სტანდარტულ ფორმატში. გამოიყენე წამკითხველები და კონვერტორები ფორმატის შესაცვლელელად](#store-dates-in-the-standard-format-use-accessors-and-mutators-to-modify-date-format)

[სხვა რჩევები](#other-good-practices)

### ერთი პასუხისმგებლობის პრინციპი (Single responsibility principle)

თითოეული კლასი და მეთოდი უნდა ასრულებდეს მხოლოდ ერთ ფუნქციას.

ცუდია:

```php
public function getFullNameAttribute()
{
    if (auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified()) {
        return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
    } else {
        return $this->first_name[0] . '. ' . $this->last_name;
    }
}
```

კარგია:

```php
public function getFullNameAttribute()
{
    return $this->isVerifiedClient() ? $this->getFullNameLong() : $this->getFullNameShort();
}

public function isVerifiedClient()
{
    return auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified();
}

public function getFullNameLong()
{
    return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
}

public function getFullNameShort()
{
    return $this->first_name[0] . '. ' . $this->last_name;
}
```

[🔝 სარჩევი](#სარჩევი)

### მძიმე მოდელები, მსუბუქი კონტროლერები

მონაცემთა ბაზასთან დაკავშირებული ყველა ლოგიკა გაიტანე Eloquent მოდელებში, ან რეპოზიტორიის კლასებში თუ იყენებ Query Builder-ს ან მშრალ SQL მოთხოვნებს.

ცუდია:

```php
public function index()
{
    $clients = Client::verified()
        ->with(['orders' => function ($q) {
            $q->where('created_at', '>', Carbon::today()->subWeek());
        }])
        ->get();

    return view('index', ['clients' => $clients]);
}
```

კარგია:

```php
public function index()
{
    return view('index', ['clients' => $this->client->getWithNewOrders()]);
}

class Client extends Model
{
    public function getWithNewOrders()
    {
        return $this->verified()
            ->with(['orders' => function ($q) {
                $q->where('created_at', '>', Carbon::today()->subWeek());
            }])
            ->get();
    }
}
```

[🔝 სარჩევი](#სარჩევი)

### ვალიდაცია

მიყევი მსუბუქი კონტროლერების და SRP პრინციპებს და ვალიდაციები კონტროლერებიდან Request კლასებში გაიტანე

ცუდია:

```php
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

    ....
}
```

კარგია:

```php
public function store(PostRequest $request)
{    
    ....
}

class PostRequest extends Request
{
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'publish_at' => 'nullable|date',
        ];
    }
}
```

[🔝 სარჩევი](#სარჩევი)

### ბიზნეს ლოგიკა სერვისის კლასებში

კონტროლერმა მხოლოდ თავისი პირდაპირი მოვალეობა უნდა შეასრულოს, ამიტომ ბიზნეს ლოგიკა ცალკე კლასებად და სერვის კლასებად გაიტანე.

ცუდია:

```php
public function store(Request $request)
{
    if ($request->hasFile('image')) {
        $request->file('image')->move(public_path('images') . 'temp');
    }
    
    ....
}
```

კარგია:

```php
public function store(Request $request)
{
    $this->articleService->handleUploadedImage($request->file('image'));

    ....
}

class ArticleService
{
    public function handleUploadedImage($image)
    {
        if (!is_null($image)) {
            $image->move(public_path('images') . 'temp');
        }
    }
}
```

[🔝 სარჩევი](#სარჩევი)

### ნუ გამეორდები (DRY)

ხელმეორედ გამოიყენე კოდი, როცა შესაძლებელია. SRP გეხმარება, თავიდან აიცილო დუბლირება. განმეორებით გამოიყენე თარგები, Eloquent მოთხოვნები და ა.შ.

ცუდია:

```php
public function getActive()
{
    return $this->where('verified', 1)->whereNotNull('deleted_at')->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->where('verified', 1)->whereNotNull('deleted_at');
        })->get();
}
```

კარგია:

```php
public function scopeActive($q)
{
    return $q->where('verified', 1)->whereNotNull('deleted_at');
}

public function getActive()
{
    return $this->active()->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->active();
        })->get();
}
```

[🔝 სარჩევი](#სარჩევი)

### ამჯობინე Eloquent-ის გამოყენება Query Builder-ისა და SQL მოთხოვნებს. ამჯობინე კოლექციები მასივებს

Eloquent-ი გაძლევს შესაძლებლობას, წერო მაქსიმალურად წაკითხვადი და სიცოცხლის უნარიანი კოდი. მას ასევე აქვს შესანიშნავი, ჩაშენებული ინსტრუმენტები მსუბუქი წაშლისთვის, ივენთებისთვის, მოთხოვნებისთვის და ა.შ.

ცუდია:

```sql
SELECT *
FROM `articles`
WHERE EXISTS (SELECT *
              FROM `users`
              WHERE `articles`.`user_id` = `users`.`id`
              AND EXISTS (SELECT *
                          FROM `profiles`
                          WHERE `profiles`.`user_id` = `users`.`id`) 
              AND `users`.`deleted_at` IS NULL)
AND `verified` = '1'
AND `active` = '1'
ORDER BY `created_at` DESC
```

კარგია:

```php
Article::has('user.profile')->verified()->latest()->get();
```

[🔝 სარჩევი](#სარჩევი)

### გამოიყენე მასობრივი შევსება (mass assignment)

ცუდია:

```php
$article = new Article;
$article->title = $request->title;
$article->content = $request->content;
$article->verified = $request->verified;
// დაამატე კატეგორია სტატიას
$article->category_id = $category->id;
$article->save();
```

კარგია:

```php
$category->article()->create($request->all());
```

[🔝 სარჩევი](#სარჩევი)

### არ შეასრულო მოთხოვნები Blade ფაილებში და გამოიყენე სრული ჩატვირთვა (პრობლემა N + 1)

Bad (100 მომხმარებლისთვის შესრულდება მონაცემთა მოთხოვნის 101 ბრძანება):

```php
@foreach (User::all() as $user)
    {{ $user->profile->name }}
@endforeach
```

Good (100 მომხმარებლისთვის შესრულდება მონაცემთა მოთხოვნის 2 ბრძანება):

```php
$users = User::with('profile')->get();

...

@foreach ($users as $user)
    {{ $user->profile->name }}
@endforeach
```

[🔝 სარჩევი](#სარჩევი)

### დაწერე კომენტარები, თუმცა ამჯობინე აღწერილობითი სახელები მეთოდებისთვის და ცვლადებისთვის

ცუდია:

```php
if (count((array) $builder->getQuery()->joins) > 0)
```

უკეთესია:

```php
// განსაზღვრე, არის თუ არა კავშირები.
if (count((array) $builder->getQuery()->joins) > 0)
```

კარგია:

```php
if ($this->hasJoins())
```

[🔝 სარჩევი](#სარჩევი)

### არ გამოიყენო JS და CSS თარგებში (template) და არ გამოიყენო HTML-ი PHP კლასებში

ცუდია:

```php
let article = `{{ json_encode($article) }}`;
```

უკეთესია

```php
<input id="article" type="hidden" value="@json($article)">

Or

<button class="js-fav-article" data-article="@json($article)">{{ $article->name }}<button>
```

In a Javascript file:

```javascript
let article = $('#article').val();
```

The best way is to use specialized PHP to JS package to transfer the data.

[🔝 სარჩევი](#სარჩევი)

### კოდში ტექსტის ნაცვლად გამოიყენე კონფიგურაციის და ენების ფაილები, და მუდმივები

ცუდია:

```php
public function isNormal()
{
    return $article->type === 'normal';
}

return back()->with('message', 'Your article has been added!');
```

კარგია:

```php
public function isNormal()
{
    return $article->type === Article::TYPE_NORMAL;
}

return back()->with('message', __('app.article_added'));
```

[🔝 სარჩევი](#სარჩევი)

### გამოიყენე საზოგადოების მიერ მიღებული Laravel-ის ინსტრუმენტები

უმჯობესია, გამოიყენო Laravel-ის ჩაშენებული ფუნქციონალი და საზოგადოების მიერ მიღებული პაკეტები, ვიდრე მე-3 მხარის პაკეტები და ინსტრუმენტები. ნებისმიერი დეველოპერი, ვისაც მოუწევს შენს აპლიკაციაზე მუშაობა, მოუწევს ახალი ინსტრუმენტების გამოყენების სწავლა. გარდა ამისა, შანსი, მიიღო დახმარება Laravel-ის საზოგადოებისგან, მნიშვნელოვნად დაბალია, როცა იყენებ მე-3 მხარის პაკეთებსა თუ ინსტრუმენტებს. ნუ აიძულებ დამკვეთს ზედმეტის გადახდას ამ ყველარის გამო.


ამოცანა | სტანდარტული ინსტრუმენტი | მე-3 მხარის ინსტრუმენტი
------------ | ------------- | -------------
ავტორიზაცია | Policies | Entrust, Sentinel and other packages
ასეტების კომპილაცია | Laravel Mix | Grunt, Gulp, 3rd party packages
სამუშაო გარემო | Homestead | Docker
Deployment | Laravel Forge | Deployer and other solutions
იუნიტ ტესტირება | PHPUnit, Mockery | Phpspec
ბრაუზერ ტესტირება | Laravel Dusk | Codeception
მონაცემთა ბაზა | Eloquent | SQL, Doctrine
თარგები | Blade | Twig
მონაცემებთან მუშაობა | Laravel collections | Arrays
ფორმების ვალიდაცია | Request classes | 3rd party packages, validation in controller
აუტენთიფიკაცია | Built-in | 3rd party packages, your own solution
API აუტენთიფიკაცია | Laravel Passport | 3rd party JWT and OAuth packages
API-ის შექმა | Built-in | Dingo API and similar packages
მუშაობა მონაცემთა ბაზების სტრუქტურასთან | Migrations | Working with DB structure directly
ლოკალიზაცია | Built-in | 3rd party packages
Realtime user interfaces | Laravel Echo, Pusher | 3rd party packages and working with WebSockets directly
სატესტო მონაცემების გენერირება | Seeder classes, Model Factories, Faker | Creating testing data manually
ამოცანების დაგეგმვა | Laravel Task Scheduler | Scripts and 3rd party packages
მონაცემთა ბაზა | MySQL, PostgreSQL, SQLite, SQL Server | MongoDB

[🔝 სარჩევი](#სარჩევი)

### მიყევი Laravel-ის სახელდების შეთანხმებებს

 მიყევი [PSR სტანდარტებს](http://www.php-fig.org/psr/psr-2/).
 
 ასევე მიყევი Laravel-ის საზოგადოების მიერ მიღებულ სახელდების შეთანხმებებს:

| რა | როგორ | კარგი | ცუდი |
| ------------ | ------------- | ------------- | ------------- |
| Controller | მხოლობითი | ArticleController | ~~ArticlesController~~ |
| Route | მრავლობითი | articles/1 | ~~article/1~~ |
| Named route | snake\_case წერტილიანი ნოტაციით | users.show_active | ~~users.show-active, show-active-users~~ |
| Model | მხოლობითი | User | ~~Users~~ |
| hasOne or belongsTo relationship | მხოლობითი | articleComment | ~~articleComments, article_comment~~ |
| All other relationships | მრავლობითი | articleComments | ~~articleComment, article_comments~~ |
| Table | მრავლობითი | article\_comments | ~~article_comment, articleComments~~ |
| Pivot table | მხოლობითი მოდელის სახელები ანბანური თანმიმდევრობით | article\_user | ~~user\_article, articles\_users~~ |
| Table column | snake\_case მოდელის სახელის გარეშე | meta\_title | ~~MetaTitle; article\_meta\_title~~ |
| Model property | snake\_case | $model->created\_at | ~~$model->createdAt~~ |
| Foreign key | მხოლობითი მოდელის სახელი \_id-ის სუფიქსით | article\_id | ~~ArticleId, id\_article, articles\_id~~ |
| Primary key | - | id | ~~custom\_id~~ |
| Migration | - | 2017\_01\_01\_000000\_create\_articles\_table | ~~2017\_01\_01\_000000\_articles~~ |
| Method | camelCase | getAll | ~~get\_all~~ |
| Method in resource controller | [ცხრილი](https://laravel.com/docs/master/controllers#resource-controllers) | store | ~~saveArticle~~ |
| Method in test class | camelCase | testGuestCannotSeeArticle | ~~test\_guest\_cannot\_see\_article~~ |
| Variable | camelCase | $articlesWithAuthor | ~~$articles\_with\_author~~ |
| Collection | აღწერილობითი, მრავლობითი | $activeUsers = User::active()->get() | ~~$active, $data~~ |
| Object | აღწერილობითი, მხოლობითი | $activeUser = User::active()->first() | ~~$users, $obj~~ |
| Config and language files index | snake\_case | articles\_enabled | ~~ArticlesEnabled; articles-enabled~~ |
| View | snake\_case | show\_filtered.blade.php | ~~showFiltered.blade.php, show-filtered.blade.php~~ |
| Config | snake\_case | google\_calendar.php | ~~googleCalendar.php, google-calendar.php~~ |
| Contract (interface) | ზედსართვი ან არსებითი სახელი | Authenticatable | ~~AuthenticationInterface, IAuthentication~~ |
| Trait | ზედსართვი სახელი | Notifiable | ~~NotificationTrait~~ |

[🔝 სარჩევი](#სარჩევი)

### გამოიყენე მოკლე და კითხვადი სინტაქტი, სადაც შესაძლებელია

ცუდია:

```php
$request->session()->get('cart');
$request->input('name');
```

კარგია:

```php
session('cart');
$request->name;
```

მეტი მაგალითები:

ზოგადი სინტაქსი | უფრო მოკლე და კითხვადი სინტაქსი
------------ | -------------
`Session::get('cart')` | `session('cart')`
`$request->session()->get('cart')` | `session('cart')`
`Session::put('cart', $data)` | `session(['cart' => $data])`
`$request->input('name'), Request::get('name')` | `$request->name, request('name')`
`return Redirect::back()` | `return back()`
`is_null($object->relation) ? null : $object->relation->id` | `optional($object->relation)->id`
`return view('index')->with('title', $title)->with('client', $client)` | `return view('index', compact('title', 'client'))`
`$request->has('value') ? $request->value : 'default';` | `$request->get('value', 'default')`
`Carbon::now(), Carbon::today()` | `now(), today()`
`App::make('Class')` | `app('Class')`
`->where('column', '=', 1)` | `->where('column', 1)`
`->orderBy('created_at', 'desc')` | `->latest()`
`->orderBy('age', 'desc')` | `->latest('age')`
`->orderBy('created_at', 'asc')` | `->oldest()`
`->select('id', 'name')->get()` | `->get(['id', 'name'])`
`->first()->name` | `->value('name')`

[🔝 სარჩევი](#სარჩევი)

### გამოიყენე IoC კონტეინერი ან ფასადები new Class-ის ნაცვლად


new Class სინტაქსი ქმნის მჭიდრო კავშირს კლასებს შორის და ართულებს ტესტირებას. გამოიყენე  IoC კონტეინერი ან ფასადები.

ცუდია:

```php
$user = new User;
$user->create($request->all());
```

კარგია:

```php
public function __construct(User $user)
{
    $this->user = $user;
}

....

$this->user->create($request->all());
```

[🔝 სარჩევი](#სარჩევი)

### პირდაპირ ნუ გამოიყენებ მონაცემებს .env ფაილიდან

გადაეცი მონაცემები კონფიგურაციის ფაილებს და გამოიყენე `config()` დამხმარე ფუნქცია ამ მონაცემებით სარგებლობისათვის.

ცუდია:

```php
$apiKey = env('API_KEY');
```

კარგია:

```php
// config/api.php
'key' => env('API_KEY'),

// გამოიყენე მონაცემები
$apiKey = config('api.key');
```

[🔝 სარჩევი](#სარჩევი)

### შეინახე თარიღები სტანდარტულ ფორმატში. გამოიყენე წამკითხველები და კონვერტორები ფორმატის შესაცვლელელად

ცუდია:

```php
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->toDateString() }}
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->format('m-d') }}
```

კარგია:

```php
// მოდელი
protected $dates = ['ordered_at', 'created_at', 'updated_at']
public function getSomeDateAttribute($date)
{
    return $date->format('m-d');
}

// თარგი
{{ $object->ordered_at->toDateString() }}
{{ $object->ordered_at->some_date }}
```

[🔝 სარჩევი](#სარჩევი)

### სხვა რჩევები

არასდროს ჩატენო ლოგიკა მარშრუტის ფაილებში.

მინიმუმამდე დაიყვანე მშრალი PHP-ის გამოყენება თარგის ფაილებში.

[🔝 სარჩევი](#სარჩევი)
