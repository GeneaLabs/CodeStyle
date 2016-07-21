# CodeStyle
A living document of notes and thoughts on code style and organization.

## Exceptions
- Catch errors and exceptions as soon as possible. Use type hinting and return types as one aspect toward achievng this.

## Type Hinting & Return Types
- Type hint all method parameters and return values.
- Type hints serve as documentation, making the code more fluent and readable.
- Type hints and return types prevent some logic errors from propogating.

## Classes
### Introspection and Type Casting
- Avoid introspection (checking the type of the class to determine the outcome of a condition).
- Avoid type casting.
- Both imply that logic should be encapsulated or refactored, likely resulting in rearranging or creation of classes.

### Final or Abstract, Never Neither
- Final classes cannot be extended. They are the final version of that class.
- Abstract classes need to be extended, and cannot be used alone.
- Avoid open-ended classes that are neither final nor abstract. Since you control the code, and it is unlikely to be used willy-nilly by anyone else free-standing, and class that is not abstract should by default be final.

### Mutability
- **TBD** Classes should be immutable. If something is changed, they should return a new instance with the changed values. (Does this even make since in a Laravel context? I think this really only comes into play for Value Objects that aren't models.)

### Naming
Name classes after what they are, and not what they do. For example, the following is corrent, and the later not so much:

```php
class Name
{
  private $firstName;
  private $lastName;
  
  function print()
  {
    return "{$this->firstName} {$this->lastName}";
  }
}
```

```php
class NamePrinter
{
  private $firstName;
  private $lastName;
  
  function print()
  {
    return "{$this->firstName} {$this->lastName}";
  }
}
```
### Contracts (Interfaces)
- Contracts aim to loosen coupling of objects in an application. It is important to remember that coupling is shifted from concrete implementations to abstract contracts (which have no logic, but only specify method interfaces).
- Contracts aren't always necessary.
- They should be used with any class that is used in _dependency injection_ or _inversion of control_ containers. This lets us easily switch out implementations, especially in third-party-packages that might require some customization.

### Statics
Avoid static classes. Classes are intended to be instanciated and be identifiable. Static classes do not have an identity are are not true objects, thus are a stow-away from the procedural era.

Further, static classes and methods are little more than modern equivalents of `GOTO` statments, procedural in nature. OOP goes beyond that, the object is the defining principle, not the code. Static classes and mothods are a crutch used to think procedurally under the guise of seeming object-oriented.

**We want our classes to be declarative, not imperative.**

From wikipedia: 
> Declarative programming is a programming paradigm—a style of building the structure and elements of computer programs—that expresses the logic of a computation without describing its control flow.

> Imperative programming is a programming paradigm that uses statements that change a program's state. In much the same way that the imperative mood in natural languages expresses commands, an imperative program consists of commands for the computer to perform. Imperative programming focuses on describing how a program operates.

Static classes consisting of collections of static methods, also known as utility classes, are nothing more than a bunch of non-OOP helper methods grouped together.

### Constructors
- Prefer many constructors of many public methods, when possible.
- Use one primary constructor, along with multiple secondary (named) constructors that all make use of the primary constructor.
- Constructors should not include any functionality or logic, but merely assign values to object properties. If logic needs to be performed, this is an indication that the information passed in should actually be another object. The reason behind this is that any code in the constructor will be parsed every time an object is created, regardless if it is necessary or not. That can't be optimized. Instead of only assignments are handled in constructors, optimization can be controlled, and only the necessary code performed.

### Properties
- Keep classes to fewer than 4 properties where possible (Models are the exception).
- Avoid classes that do not encapsulate any data (for example purely static classes), possibly with the exception of null-object class instances.
- **TBD** Avoid public properties where possible. Class properties should only be set through constructors and manipulated through methods. (Are getters and settings preferrable over public properties -- what dangers are there to consider? What is the underlying reason this is considered such a bad thing?)
- Avoid setting properties to null. (And conversely avoid null-checks.)

### Methods
- Name methods according to what they return. Their names should be self-documenting.
- Methods that perform an action should be a verb, and return `null`.
- Methods that return objects should be nouns and named after the object they return (they can be prefixed with adjectives that help better describe the object being returned).
- Consider method names that are actions, but where an object is returned, like `save()`. In those cases, instead of returning `null`, return the object instance instead.
- Methods should only return one type of data, never mixed.
- Methods should have a declared parameter list, and not use a dynamic one (exceptions exist of course for magic methods).

#### Getters and Setters
- Do not use getters and setters, other than the special methods used in models (`getPropertyAttribute()` and `setPropertyAttribute($value)`).
- 

## Controllers & Routes
### APIs
- API routes should be within an `API` route namespace.
- API routes should use resource controllers: https://laravel.com/docs/5.2/controllers#restful-resource-controllers.
- API route controllers should be named after the model they act on.
- API route controller methods should implement the restful naming scheme.
- API route controllers should only be responsible for a single model.

### Views
- View routes should have no namespace.
- View routes should use implicit controllers: https://laravel.com/docs/5.1/controllers#implicit-controllers.
- View route controllers should be named after the blade view folder they are responsible for.
- View route controller methods should be named after the action they are responsible for.
- View route controllers should only be responsible for a single view type, but responsible for all sub-views, like:
  - /resources/views/reports/index.blade.php
  - /resources/views/reports/create.blade.php
  - /resources/views/reports/show.blade.php
  - etc...

### Requests
- All controller methods that accept data should validate the incoming data through Requests.
- All controller methods that accept data should process the incoming data in a `process()` method that returns whatever object necessary to continue and logic in the controller method. For example:

  ```php
      public function store(CreateReport $request) : RedirectResponse
    {
        $search = $request->process();

        return redirect()->route('reports.show', ['addressSlug' => $search->address->slug]);
    }
  ```

## Models

### Organization
1. List `use` classes.
2. List the public, protected, and private properties, each group in alphabetical order.
3. List relationship methods.
4. List getter and setter methods.
5. List persistence methods.
6. List protected methods.
7. List private methods.

#### Persistence Methods
- Laravel models are the de-facto persistence repository, especially in eloquent.
- Do not use the generic eloquent CRUD methods `save()`, `update()`, `create()`, `delete()`, etc. outside of the model. Instead create descriptive methods that explain exactly what is happening. This decouples the business domain from the persistence domain of the app, as well as makes code so much more humanly readable. For instance: instead of `$user->save()` create a method that handles a specific situation, like `$agent->addListingInfo($listingInfo);` and then handle all the data parsing and assignment in the method, at the end of which `$this->save()` is called to persist the changes.

## Tests
- Always write unit and integration tests, testing for success and failure for each scenario.
- Only test public methods of classes.
- Write tests so that they cover all protected and private methods of the class, accessed through the public methods. If there are non-public methods that aren't covered, they are either inaccessible, or the tests are comprehensive enough. If they are inaccessible, those methods should be removed.
- Tests should document the functionality of classes and their methods.
- Mock any external interfaces you don't control, and test for both successes and failures.
- Do not mock classes that you control.

## NULL
- Avoid the use of `null`, both as a negative return value from a method, but also as default parameters.
- Instead of `null`, use null-classes of the same type expected to be returned from methods.
- Use null-objects as default parameters if none are passed in to methods.


## NPM Dependencies
- Install all npm packages that will be pre-compiled as dev requirements when using them in Laravel.

## Vue
### Components
- Use the `vueify` npm package to create components in a single file with a `.vue` extension.
- Components should be structured in the following order:
  
  ```
  <script>
      // Logic: place JavaScript for this component here
  </script>
  
  <template>
      <!-- Layout: place HTML for this component here -->
  </template>
  
  <style lang="sass" scoped>
      /* Design: place SASS styles for this component here */
  </style>
  ```
  
  The reason for this is that each section builds on the knowledge of the previous section. To Understand the layout, you need to understand the logic, and to understand the design, you need to understand the layout.
