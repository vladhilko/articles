## Overview

This is the first article in the Clean Rails API series. In this piece, we will provide a brief introduction to Rails API. We'll discuss what it is, how it looks, its key components, and share simple examples. We'll also demonstrate how request specs can be added. Furthermore, we'll provide a direction for API evolution and highlight the problems that have to be solved in the future to provide a high-quality solution.

## Introduction

API (Application Programming Interface) is a set of rules and protocols that allow one application to interact with another application. There are several API paradigms that fall into two main categories: **Request–Response APIs** and **Event-Driven APIs**:

- **Request–Response APIs**
  - REST API (Representational State Transfer)
  - RPC (Remote Procedure Call)
  - GraphQL
- **Event-Driven APIs**
  - Polling
  - WebHooks
  - WebSockets
  - HTTP Streaming

In the context of this introduction to Rails API, we will be working with REST API. The name REST ("Representational State Transfer") encapsulates the essence of this architectural style: **representing** resources and their **states** and **transferring** data between clients and servers using HTTP.

## Implementation

Let's review our plan for the Rails API introduction. We'll begin with the **Primitive Implementation**. Following that, we will add several **CRUD Endpoints** and cover them using **RSpec Request Tests**. Finally, we will highlight areas for **Improvements** that we'll consider in the next article of this series.

### Primitive Implementation

At this part, we're going to add a very simple endpoint just to demonstrate how the API may look. We'll add this endpoint to the existing Rails application, but if you want to start from scratch, just create a new app with the `rails new api_sample --api` command. We will start by providing the **Desired Interface**, then we will add the **Implementation**, and after that, **check that everything works as expected**.

#### Desired Interface

First of all, let's decide how our interface should look like and what data we expect to receive. Let's assume that we want to send a GET request to `http://localhost:3000/api/hello-world` and expect to receive the following response:

```js
// Status: 200 OK

{
  "data": "Hello World!"
}
```

#### Implementation

To add such an API endpoint, we have to follow two simple steps:

1. Create a new controller and method that will return `"data": "Hello World!"` in JSON format.
2. Add a new route to bind our HTTP request to the created controller method.

To implement the first step, we will add a new `HelloWorldController` with a `hello_world` method inside:

```rb
# app/controllers/api/hello_world_controller.rb

# frozen_string_literal: true

module Api
  class HelloWorldController < ApplicationController

    def hello_world
      render json: { data: 'Hello World!' }
    end

  end
end
```

To implement the second step, we will add `get 'api/hello-world'` to `config/routes.rb`, which will bind the HTTP request and the controller method.

```rb
# config/routes.rb

# frozen_string_literal: true

Rails.application.routes.draw do
  get 'api/hello-world', action: :hello_world, controller: 'api/hello_world'
end
```

#### Verify Functionality

After completing our implementation, we can run the server and check its functionality:

```
rails s
```

When you open your browser and visit `http://localhost:3000/api/hello-world`, you should see the following:

```js
// Status: 200 OK

{
  "data": "Hello World!"
}
```

That's it! The simplest API has been added. In the next chapter, we will try to add more HTTP requests to cover CRUD actions.

### Adding CRUD Endpoints

In this chapter, we are going to add new API endpoints for CRUD HTTP requests. We'll strive to keep everything as simple as possible, focusing solely on the core components. We will begin by defining the **Desired Interface**, then proceed with the **Implementation**, and finally, we'll **verify that everything works as expected** and cover all endpoints with **Request Specs**.

### Desired Interface

We've demonstrated how to create the simplest API in the Hello World example. Let's go further and create more endpoints with different HTTP methods. I'd like to have the following 4 endpoints and expect them to return the following data:

- `GET http://localhost:3000/api/countries`

```js
// Status: 200 OK

[
  {
    "id": 1,
    "name": "Spain"
  }
]
```

- `POST http://localhost:3000/api/countries`

```js
// Status: 201 Created

{
    "id": 1,
    "name": "Spain"
}
```

- `PUT http://localhost:3000/api/countries`

```js
// Status: 204 No Content
```

- `DELETE http://localhost:3000/api/countries/1`

```js
// Status: 204 No Content
```

### Implementation

To add these 4 endpoints, we will need to follow the same 2 steps as in the previous part, plus one improvement. Let's start with the improvement. In our enhancement, we want to create a new wrapper class that we will use as the parent class for all our API controllers. This new class will look like this:

```rb
# app/controllers/api/application_controller.rb

# frozen_string_literal: true

module Api
  class ApplicationController < ActionController::API
  end
end
```

Then, instead of inheriting from the `ApplicationController` class, we'll have all API controllers inherit from `Api::ApplicationController`. This change will provide us with more flexibility and reduce the risk of breaking non-API controllers if we have any.

```rb
# app/controllers/api/hello_world_controller.rb

# frozen_string_literal: true

module Api
  class HelloWorldController < Api::ApplicationController

    def hello_world
      render json: { data: 'Hello World!' }
    end

  end
end
```

Now let's create a new controller with 4 methods to handle all the requests mentioned above:

```rb
# app/controllers/api/countries_controller.rb

# frozen_string_literal: true

module Api
  class CountriesController < Api::ApplicationController

    def index
      render json: [ { id: 1, name: 'Spain' } ]
    end

    def create
      render json: { id: 1, name: 'Spain' }, status: :created
    end

    def update
      head :no_content
    end

    def destroy
      head :no_content
    end

  end
end
```

And update the `routes.rb` file to bind HTTP requests and controller methods:

```rb
# frozen_string_literal: true

Rails.application.routes.draw do
  get 'api/countries', action: :index, controller: 'api/countries'
  post 'api/countries', action: :create, controller: 'api/countries'
  put 'api/countries', action: :update, controller: 'api/countries'
  delete 'api/countries', action: :destroy, controller: 'api/countries'
end
```

### Verify Functionality

After everything has been implemented, let's check if it works:

- `GET http://localhost:3000/api/countries`

```js
// Status: 200 OK

[
  {
    "id": 1,
    "name": "Spain"
  }
]
```

The first one works as expected, and we know how to check it in the browser, but what about the others? How can we send different HTTP requests? To solve this problem, we're going to use Postman. Postman is like an extended browser version that can send any HTTP request based on your instructions. Postman looks like this:

<img width="929" alt="Screenshot 2023-08-08 at 15 07 55" src="https://user-images.githubusercontent.com/12089948/259108422-b29ae1a3-5fd6-4feb-b176-3f2d2a6875d2.png">


Let's check our other HTTP requests:

- `POST http://localhost:3000/api/countries`

```js
// Status: 201 Created

{
    "id": 1,
    "name": "Spain"
}
```

- `PUT http://localhost:3000/api/countries`

```js
// Status: 204 No Content
```

- `DELETE http://localhost:3000/api/countries`

```js
// Status: 204 No Content
```

As you can see, everything works as expected.

### Adding Request Tests

After all requests have been added, we need to cover them using request specs to ensure that everything will work as expected in the future as well. To add specs, we will use the [rspec-rails gem](https://github.com/rspec/rspec-rails). Our plan is to cover the 4 HTTP request methods that have been added above:

- `GET http://localhost:3000/api/countries`
- `POST http://localhost:3000/api/countries`
- `PUT http://localhost:3000/api/countries`
- `DELETE http://localhost:3000/api/countries`

- ### `GET http://localhost:3000/api/countries`

```rb
# spec/requests/countries/list_spec.rb

# frozen_string_literal: true

require 'rails_helper'

RSpec.describe 'Listing Countries' do
  subject(:send_request) { get request_uri }

  let(:request_uri) { '/api/countries' }

  it 'lists all countries' do
    send_request

    expect(response).to have_http_status(:successful)
    expect(JSON.parse(response.body)).to eq([{ 'id' => 1, 'name' => 'Spain' }])
  end
end
```

- ### `POST http://localhost:3000/api/countries`

```rb
# spec/requests/countries/create_spec.rb 

# frozen_string_literal: true

require 'rails_helper'

RSpec.describe 'Creating Countries' do
  subject(:send_request) { post request_uri }

  let(:request_uri) { '/api/countries' }

  it 'creates a country and returns it' do
    send_request

    expect(response).to have_http_status(:created)
    expect(JSON.parse(response.body)).to eq({ 'id' => 1, 'name' => 'Spain' })
  end
end
```

- ### `PUT http://localhost:3000/api/countries`

```rb
# spec/requests/countries/update_spec.rb

# frozen_string_literal: true

require 'rails_helper'

RSpec.describe 'Updating Countries' do
  subject(:send_request) { put request_uri }

  let(:request_uri) { '/api/countries' }

  it 'updates the country and returns nothing' do
    send_request

    expect(response).to have_http_status(:no_content)
    expect(response.body).to be_empty
  end
end
```

- ### `DELETE http://localhost:3000/api/countries`

```rb
# spec/requests/countries/destroy_spec.rb 

# frozen_string_literal: true

require 'rails_helper'

RSpec.describe 'Destroying Countries' do
  subject(:send_request) { delete request_uri }

  let(:request_uri) { '/api/countries' }

  it 'deletes the country and returns nothing' do
    send_request

    expect(response).to have_http_status(:no_content)
    expect(response.body).to be_empty
  end
end

```

That's it. You can run `rspec spec/requests` to ensure that they work.

## Improvements

We have demonstrated the simplest examples, but there's room for many improvements. I would consider at least the following points:

- Update Routing
- Add Representers/Serializers
- Add logic to Create/Update/Delete resources
- Add Error Handling
- Add Request Tests/RSwag Documentation
- Apply Authorization/Authentication rules
- Support URL queries (limits/pagination/filtering)
- Set up Middleware (Rate Limits/CORS/etc...)
- Add API Versioning

## Conclusion

To sum it up, this article is the beginning of our Clean Rails API series. We've talked about what Rails API is, shown some examples, and explained how to test it. In the future, we will improve our solution by addressing each improvement point one by one.
