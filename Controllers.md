# Controllers

## Purpose
Controllers serve only three purposes:
1. Triggered by a request.
2. Gather the response required for a specific request.
3. Return the response in , well, response to the request.

Controllers should contain no business logic. Instead, various types of business logic should be extracted out to:
- form submission processing should be extracted to Form Request objects.
- JSON responses should be extracted to Resource objects.
- other business logic should be extracted to Response objects.

## Structure
Controllers should only contain RESTful methods. If additional methods seem to be needed,
a new RESTful controller with an appropriate name should be created to satisfy that need.
