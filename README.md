## Polymorphic Relationships Example

### 1. One to One

Create some stuff to prepare.

1. model Post also factory and migrate

```shell
php artisan make:model Post -fm
```

2. model Image and migrate

```shell
php artisan make:model Image -m
```

4. add and edit some things

```php
// migrate create posts
Schema::create('posts', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('title');
    $table->text('content');
    $table->timestamps();
});
// factory of posts
public function definition()
{
    return [
        'title' => $this->faker->sentence,
        'content' => $this->faker->paragraph,
    ];
}

// migrate create images
Schema::create('images', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('url');
    $table->unsignedBigInteger('imageable_id');
    $table->string('imageable_type');
    $table->timestamps();
});

```

5. add polymorphic relationships in model

```php
// Post and User model add
protected $guarded = []; // only for Post models

public function image()
{
    return $this->morphOne(Image::class, 'imageable');
}

// Image model add
protected $guarded = [];

public function imageable()
{
    return $this->morphTo();
}

```

6. Now we are going to create data sample

```shell
php artisan tinker
>>> Post::factory()->count(1)->create()
>>> $post = Post::first();
>>> $post->image()->create(['url'=> 'path/image.png']);
>>> User::factory()->count(1)->create()
>>> $user = User::first();
>>> $user->image()->create(['url'=> 'path/image_user.png']);
```

7. enjoy your result

![](one-to-one.GIF)
