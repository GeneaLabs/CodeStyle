# CodeStyle
A living document of notes and thoughts on code style and organization.

## NPM Dependencies
- Install all npm packages that will be pre-compiled as dev requirements when using them in Laravel.

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
