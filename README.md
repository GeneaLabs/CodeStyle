# CodeStyle
A living document of notes and thoughts on code style and organization.

## NPM Dependencies
- Install all npm packages that will be pre-compiled as dev requirements when using them in Laravel.

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
