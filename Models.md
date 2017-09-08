# Models
## Structure
- Model class should only contain properties and relationship methods.
- All attribute methods should be extracted to an Attributes trait.
- All queriy methods should be extracted to an Query trait.

## Eloquent Queries
All Eloquent queries should be processed in dedicated methods within the model itself,
 or in an extracted trait that serves to extend the model with traits. The default
 getter methods provided by Eloquent should not be used, with the exception of the following:
  - find()
  - findOrFail()

## Relationship Properties
Don't query relationship properties on models directly. Instead expose the relationship property as a dynamic property in the model itself. This allows you to provide a default if the relationship doesn't exist, as well as limits interdependence of models to only the models themselves, and not through your code. For example, instead of `$book->author->name`, create a model attribute called `authorName`:

```php
class Book extends Model
{
    use BookRepository;
    
    public function author() : HasOne
    {
        return $this->hasOne(Author::class);
    }
}

trait BookRepository
{
    public function getAuthorNameAttribute() : string
    {
        return $this->author->name ?? '';
    }
}
```

## Naming Conventions
### For Properties or Methods with Certain Return Types
- boolean props or methods that check if a certain condition is true should be named `has<Condition in past tense>`.
