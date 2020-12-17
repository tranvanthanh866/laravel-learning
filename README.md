# Polymorphic Relationships Example

## 1. One to One

1. Create some stuff to prepare.

-   model Post also factory and migrate

```shell
php artisan make:model Post -fm
```

-   model Image and migrate

```shell
php artisan make:model Image -m
```

2. Add and edit migrate and models

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

3. add polymorphic relationships in model

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

4. Now we are going to create data sample

```shell
php artisan tinker
>>> Post::factory()->count(1)->create()
>>> $post = Post::first();
>>> $post->image()->create(['url'=> 'path/image.png']);
>>> User::factory()->count(1)->create()
>>> $user = User::first();
>>> $user->image()->create(['url'=> 'path/image_user.png']);
```

5. enjoy your result

![](one-to-one.GIF)

above it's just about relationships one to one, so how about the relationships one to many in polymorphic?

let's find out by following bellow.

## One to many (Polymorphic)

1. create some stuff to prepare

-   Model and artisan

```shell
php artisan make:model Comment -m
```

```php
// migrate create comments
Schema::create('comments', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->text('content');
    $table->unsignedBigInteger('commentable_id');
    $table->string('commentable_type');
    $table->timestamps();
})
// Models Comment
protected $guarded = [];

public function commentable()
{
    return $this->morphTo();
}
// Models Post
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}


```

2. Now we are going to create data relationships example

```shell
php artisan tinker
>>> $post = Post::first();
>>> $post->comments()->create(['content'=> 'great content keep moving']);
>>> $post->comments()->create(['content' => 'this post helps me a lot thanks']);
```

3. one more example

```shell
php artisan make:model Video -mf
```

```php
// migrate videos and factory
Schema::create('videos', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('url');
    $table->timestamps();
});

public function definition()
{
    return [
        'url' => $this->faker->name,
    ];
}

// Video Model
protected $guarded = [];

public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}
```

```shell
php artisan migrate
php artisan tinker
>>> Video::factory()->count(1)->create()
>>> $video = Video::first();
>>> $video->comments()->create(['content'=>'cool comment 1']);
>>> $video->comments()->create(['content'=>'cool comment for video 1']);
```

## Many to many relationships (Polymorphic)

```shell
php artisan make:model Tag -m
php artisan make:migration creates_taggables_table --create taggables
```

```php
// migration
Schema::create('tags', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('name');
    $table->timestamps();
});
Schema::create('taggables', function (Blueprint $table) {
    $table->unsignedBigInteger('tag_id');
    $table->unsignedBigInteger('taggable_id');
    $table->string('taggable_type');
    $table->timestamps();
});


// Tag model
protected $guarded = [];

public function posts()
{
    return $this->morphedByMany(Post::class, 'taggable');
}

public function videos()
{
    return $this->morphedByMany(Video::class, 'taggable');
}

// Post model
public function tags()
{
    return $this->morphToMany(Tag::class, 'taggable');
}
// video Model

public function tags()
{
    return $this->morphToMany(Tag::class, 'taggable');
}
```

```shell
php artisan migrate
php artisan tinker
>>> $post = Post::first();
>>> $post->tags()->create(['name'=> 'PHP']);
>>> $post->tags()->create(['name'=> 'Eloquent']);
>>> $video->tags()->create(['name'=> 'Laravel']);
>>> $video->tags()->attach(1); # id if name tag have been exists
```
