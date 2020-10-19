# GeneaLab's Laravel Code Style Guide
A living document of notes and thoughts on code style and organization.

## General
- [Conventions](https://github.com/GeneaLabs/CodeStyle/blob/master/Models.md)
- [Patterns](https://github.com/GeneaLabs/CodeStyle/blob/master/Models.md)
- [Exceptions](https://github.com/GeneaLabs/CodeStyle/blob/master/Models.md)
- [Classes](https://github.com/GeneaLabs/CodeStyle/blob/master/Classes.md)
- [Testing](https://github.com/GeneaLabs/CodeStyle/blob/master/Models.md)

## Laravel
- [Environment Configuration](https://github.com/GeneaLabs/CodeStyle/blob/master/EnvironmentConfiguration.md)
- [Routes](https://github.com/GeneaLabs/CodeStyle/blob/master/Routes.md)
- [Controllers](https://github.com/GeneaLabs/CodeStyle/blob/master/Controllers.md)
- [Models](https://github.com/GeneaLabs/CodeStyle/blob/master/Models.md)
- [Migrations](https://github.com/GeneaLabs/CodeStyle/blob/master/Migrations.md)
- [Seeders](https://github.com/GeneaLabs/CodeStyle/blob/master/Seeders.md)
- [Eloquent](https://github.com/GeneaLabs/CodeStyle/blob/master/Eloquent.md)

## Exceptions
- Catch errors and exceptions as soon as possible. Use type hinting and return types as one aspect toward achievng this.

## Type Hinting & Return Types
- Type hint all method parameters and return values.
- Type hints serve as documentation, making the code more fluent and readable.
- Type hints and return types prevent some logic errors from propogating, catching them as close as possible to their source.

## Config and Env
- Always assign env variables to config properties in the config file that is most appropriate (or create a new config file it that makes more sense).
- Always refer to the config variable in your code, never reference `env('xxxxx', 'abc)` outside of config files.

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
- View route controllers should be named after the blade view folder they are
 responsible for.
- View route controller methods should be named after the action they are
 responsible for.
- View route controllers should only be responsible for a single view type, but
 responsible for all sub-views, like:
  - /resources/views/reports/index.blade.php
  - /resources/views/reports/create.blade.php
  - /resources/views/reports/show.blade.php
  - etc...

## Models
### Eager Loading
- Avoid eager loading relationships in the `protected $with = [];` variable.
- Try to explicitly load relationships at the point they are used.

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
- Do not use the generic eloquent CRUD methods `save()`, `update()`, `create()`,
 `delete()`, etc. outside of the model. Instead create descriptive methods that
 explain exactly what is happening. This decouples the business domain from the
 persistence domain of the app, as well as makes code so much more humanly readable.
 For instance: instead of `$user->save()` create a method that handles a specific
 situation, like `$agent->addListingInfo($listingInfo);` and then handle all the
 data parsing and assignment in the method, at the end of which `$this->save()`
 is called to persist the changes.

## Tests
- Always write unit and integration tests, testing for success and failure for
 each scenario.
- Only test public methods of classes.
- Write tests so that they cover all protected and private methods of the class,
 accessed through the public methods. If there are non-public methods that aren't
 covered, they are either inaccessible, or the tests are comprehensive enough.
 If they are inaccessible, those methods should be removed.
- Tests should document the functionality of classes and their methods.
- Mock any external interfaces you don't control, and test for both successes
 and failures.
- Do not mock classes that you control.

## NULL
- Avoid the use of `null`, both as a negative return value from a method, but also
 as default parameters.
- Instead of `null`, use empty classes of the same type expected to be returned
 from methods (see NULL-Objects section).

## Testing
### Unit Tests
- Write unite tests before implementing classes (only implement classes, never
 procedural code).
- When starting an app, start where you would start with writing code. The first
 test doesn't have to be elegant, or even correct. The most important thing is
 just to get started.
- Always do Red/Green/Refactor TDD. This means writing tests for the code you
 would like to see in an optimal world. Then make the failing test pass using the
 minimum amount of code. Write another test to expand on the first test, which
 again makes the existing code fail. Refactor your code to make the second test
 pass. Rinse and repeat until you have the minimum necessary functionality for
 your MVP (minimum viable product).
- During the red/green/refactor process, keep in mind that you need to develop
 from two different perspectives:
  1. When writing tests, keep the larger picture of the application and business
   domain in mind.
  2. When writing code to satisfy tests, only think about the test that needs to
   be satisfied. DO NOT THINK ABOUT BUSINESS LOGIC, ONLY FOCUS ON MAKING TESTS
   GREEN.
- Never add code that wont be used.
- Remove any code that is not used.
- Use cyclomatic complexity as a guide for the number of tests needed to achieve
 full test coverage of your code.
- As your tests get more specific, your code should become more generic.
- Goal of tests is to get as quickly as possible to "Shameless Green", which means
 that no matter how ugly your code is, it satisfies all tests, thus is "green".
- One of the principles of Shameless Green is that code is written for understanding,
 rather than extreme adherence to any and all patterns. The human is the focus.
- Consider Robert Martin's Transformation Priority Premise
 (https://8thlight.com/blog/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html)
 when writing functional code to keep code complexity at a minimum. Try to opt
 for the highest ranked option.
- Abstract logic into methods only if they are used multiple times. Otherwise this
 might introduce mental debt and increase code complexity.
- Consider waiting to dry out duplicated code until after a few tests cover it.
 This way the correct abstraction might reveal itself, rather than prematurely
 abstracting it out incorrectly.


## Patterns
### NULL-Objects
Do not use null-objects as a substitute for using `null` for negative results
 (btw, don't use `null` anywhere in code, as it is semantically void). Instead,
 add a `__toString()` method that returns an empty string for empty models, and
 non-empty value for valid objects. For example:

  ```php
  public function __toString()
  {
      return "{$this->getPrimaryKey()}";
  }
  ```

This has the following benefits:
- Less cluttered code by eliminating noisy null-object classes (which serve no
 other purpose than to eliminate null-checks).
- Ease of maintenance.
- Adherence to single responsibility principle, in that a method only returns a
 single type of data.

### DRY (Do Not Repeat Yourself)
- Do not apply the dry principle to code that deals with content, or even content
 itself.
- Apply DRY only to code that deals with logic.
- If you find wet code, but it doesn't appear that it can be dried out, you might
 have an architectural flaw. Restructuring or refactoring your classes could help.
- Only start drying out your code if you actually have duplication.
- DO NOT dry out code that isn't duplicated, as this leads to premature abstraction,
 and is almost certain to introduce both mental and technical debt.

### Collections
- Use collections for all the things.
- **TBD** Consider using classes as collection transformers:

  > is there any intent to use more of magic method `__invoke` in future versions of Laravel? I found an interesting possibly to try things like: `$collection->map(new MyModelTransformer());`

## Conventions
### Avoid Conditionals
- Try to avoid conditionals where possible (they increase cyclomatic complexity,
 which in turn increases mental and technical debt).
- Never use `else` if you end up using a conditional.
- One alternative to conditionals is interpolation.
- Try to keep all code within the 80 character per line soft-limit.
- Absolutely always keep all code under 120 characters per line. However, exceeding
 80 characters can be an indication of too deeply indented code. Consider
 refactoring out to one or more methods.
- Adhere to PSR1 and PSR2, no exceptions.
- Code should not have any default PHPMD, PHPCS, and PHPCPD warnings. (**TBD**
 Try not to customize the rule sets, if at all possible.)

## Blade
### Directives
- All blade directives shouls have a space between the directive and the parameters, mirroring the way PSR2 handles the equivalent PHP statements (if, for, etc.).

## NPM Dependencies
- Install all npm packages that will be pre-compiled as dev requirements when
 using them in Laravel.

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
