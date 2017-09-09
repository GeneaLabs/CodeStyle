# Models
## Structure
- Model class should only contain properties and relationship methods.
- All attribute methods should be extracted to an Attributes trait.
- All queriy methods should be extracted to an Query trait.

```
App
 |-Traits
 | |-Attributes
 | | \-Book
 | |
 | \-Queries
 |   \-Book
 \-Book
```

```php
<?php namespace App;

use App\Address;
use App\Event;
use App\Venue;
use App\Funding;
use App\GradeLevel;
use App\Instrument;
use App\Partner;
use App\Traits\Attributes\Contact as Attributes;
use App\Traits\Queries\Contact as Queries;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Notifications\Notifiable;

class Contact extends BaseModel
{
    use Attributes;
    use Notifiable;
    use Queries;

    protected $appends = [
        'searchKey',
        'searchUrl',
    ];
    protected $fillable = [
        'name',
        'title',
        'work_phone',
        'mobile_phone',
        'work_email',
        'private_email',
    ];

    public function address() : BelongsTo
    {
        return $this->belongsTo(Address::class);
    }

    public function contactTypes() : BelongsToMany
    {
        return $this->belongsToMany(ContactType::class);
    }

    public function eventsManaged() : HasMany
    {
        return $this->hasMany(Event::class)
            ->where('contact_id', $this->id);
    }

    public function eventsModerated() : BelongsToMany
    {
        return $this->belongsToMany(Event::class, 'event_moderator');
    }

    public function eventsPerformedAt() : BelongsToMany
    {
        return $this->belongsToMany(Event::class, 'event_musician');
    }

    public function fundings() : HasMany
    {
        return $this->hasMany(Funding::class);
    }

    public function gradeLevels() : BelongsToMany
    {
        return $this->belongsToMany(GradeLevel::class, 'teacher_grade_level', 'teacher_id', 'grade_level_id');
    }

    public function instruments() : BelongsToMany
    {
        return $this->belongsToMany(Instrument::class);
    }

    public function partnerships() : HasMany
    {
        return $this->hasMany(Partner::class);
    }

    public function venues() : BelongsToMany
    {
        return $this->belongsToMany(Venue::class);
    }

    public function toSearchableArray()
    {
        $data = $this->toArray();

        unset($data['address']);

        return $data;
    }
}
```

```php
<?php namespace App\Traits\Attributes;

trait Contact
{
    public function getGravatarAttribute() : string
    {
        $defaultAvatar = 'https://www.gravatar.com/avatar/?d=mm';
        $workAvatar = 'https://www.gravatar.com/avatar/' . md5(strtolower(trim($this->work_email))) . '?d=mm';
        $privateAvatar = 'https://www.gravatar.com/avatar/' . md5(strtolower(trim($this->private_email))) . '?d=mm';

        if (md5(file_get_contents($workAvatar)) !== md5(file_get_contents($defaultAvatar))) {
            return $workAvatar;
        }

        if (md5(file_get_contents($privateAvatar)) !== md5(file_get_contents($defaultAvatar))) {
            return $privateAvatar;
        }

        return $defaultAvatar;
    }

    public function getSearchKeyAttribute() : string
    {
        return $this->name;
    }

    public function getSearchUrlAttribute() : string
    {
        return route('contacts.show', $this->getKey());
    }
}
```

```php
<?php namespace App\Traits\Queries;

use Illuminate\Support\Collection;

trait Contact
{
    public function getAll() : Collection
    {
        return cache()->tags(['contact'])
            ->rememberForever("contact-{$this->id}-getAll", function () {
                return $this->orderBy('name')->get();
            });
    }

    public function getByTypes(array $types) : Collection
    {
        $key = implode('-', $types);

        return cache()->tags(['contact', 'contacttypes'])
            ->rememberForever("contact-{$this->id}-getByTypes-{$key}", function () use ($types) {
                return $this->with(['contactTypes' => function ($query) use ($types) {
                        $query->whereIn('title', $types);
                    }])
                    ->orderBy('name')
                    ->get();
            });
    }
}
```

## Eloquent Queries
All Eloquent queries should be processed in dedicated methods within the queries trait of the model. The default
 getter methods provided by Eloquent should not be used anywhere outside of the model or its traits, with the
 exception of the following:
  - find()
  - findOrFail()

## Relationship Properties
Don't query relationship properties on models directly. Instead expose the relationship property as a dynamic property in the model itself. This allows you to provide a default if the relationship doesn't exist, as well as limits interdependence of models to only the models themselves, and not through your code. For example, instead of `$book->author->name`, create a model attribute called `authorName`:

```php
<?php namespace App;

use App\Traits\Attributes\Book as Attributes;

class Book extends Model
{
    use Attributes;
    
    public function author() : HasOne
    {
        return $this->hasOne(Author::class);
    }
}
```

```php
<?php namespace App\Traits\Attributes;

trait Book
{
    public function getAuthorNameAttribute() : string
    {
        return $this->author->name ?? '';
    }
}
```

```php
// and somewhere in your business logic:
$book->authorName;
```

## Naming Conventions
### For Properties or Methods with Certain Return Types
- boolean props or methods that check if a certain condition is true should be named `has<Condition in past tense>`.

### For Query Methods
- methods returning a single instance should be prefixed with `find` followed by the name of the model instance it returns, for example `->findUserByName(string $name)`.
- methods returning a collection of instances should be prefixed with `get` followed by the name of the model instances is returns, for example `->getUsersByType(string $type)`.
