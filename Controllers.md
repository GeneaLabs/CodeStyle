# Controllers

## Purpose
Controllers serve only two purposes:
1. Process a specific request, handled by a request object (see below).
2. Return an appropriate repsonse, handled by a response object (see below).

Controllers should contain no business logic. Instead, various types of business logic should be extracted out to:
- form submission processing should be extracted to Form Request objects.
- JSON responses should be extracted to Resource objects.
- other business logic should be extracted to Response objects.

## Structure
Controllers should only contain RESTful methods. If additional methods seem to be needed,
a new RESTful controller with an appropriate name should be created to satisfy that need.

### Single-Method Controllers
If the controller only has a single method, use `__invoke()`.
```php
<?php namespace App\Http\Controllers;

use Illuminate\View\View;

class WelcomeController extends Controller
{
    public function __invoke() : View
    {
        return view('welcome');
    }
}
```

## Request Objects
- All controller methods that accept data should validate the incoming data
 using Request objects
- All controller methods that accept data should process the incoming data in a
 `process()` method that returns whatever object necessary to continue the logic
 in the controller method. For example:

  ```php
      public function store(CreateReport $request) : RedirectResponse
    {
        $search = $request->process();

        return redirect()->route('reports.show', ['addressSlug' => $search->address->slug]);
    }
  ```
  
## Response Objects
Resource objects should be utilized to build up and return all data and objects required by the view to the view.
```php
<?php namespace App\Http\Controllers\Reports;

use App\Http\Controllers\Controller;
use App\Http\Requests\Api\Reports\Event as Request;
use App\Http\Responses\Reports\Events\Csv;
use Illuminate\Contracts\Support\Responsable;

class Event extends Controller
{
    public function index(Request $request) : Responsable
    {
        return new Csv(
            $request->process(),
            'CEEd Events Report.csv'
        );
    }
}
```

```php
<?php namespace App\Http\Responses\Reports\Events;

use App\Http\Resources\BaseResourceCollection;
use Illuminate\Contracts\Support\Responsable;
use Illuminate\Http\Response;
use League\Csv\Writer;
use SplTempFileObject;

class Csv implements Responsable
{
    protected $downloadFilename;
    protected $events;

    public function __construct(
        BaseResourceCollection $events,
        string $downloadFilename
    ) {
        $this->downloadFilename = $downloadFilename;
        $this->events = $events;
    }

    /**
     * @SuppressWarnings(PHPMD.StaticAccess)
     * @SuppressWarnings(PHPMD.UnusedFormalParameter)
     */
    public function toResponse($request) : Response
    {
        $csv = Writer::createFromFileObject(new SplTempFileObject);
        $csv->insertOne([
            'Season',
            'Starts At',
            'Ends At',
            'Program',
            'Venue',
            'Moderators',
            'Musicians',
        ]);

        $this->events->each(function ($event) use ($csv) {
            $csv->insertOne([
                $event->seasonName,
                $event->starts_at,
                $event->ends_at,
                $event->programName,
                $event->venueName,
                $event->moderators->pluck('name')->implode(', '),
                $event->musicians->pluck('name')->implode(', '),
            ]);
        });

        $headers = [
            'Content-Type' => 'text/csv; charset=UTF-8',
            'Content-Disposition' => "attachment; filename=\"{$this->downloadFilename}\"",
        ];

        return response((string)$csv, 200, $headers);
    }
}
```
