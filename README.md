# Gxapi

[![Code Climate](https://codeclimate.com/repos/5276603656b10215e9014c80/badges/ddae4ab98746f66abf45/gpa.png)](https://codeclimate.com/repos/5276603656b10215e9014c80/feed)

Gxapi interfaces with Google Analytics Experiments to retrieve data from
its API and determine which variant should be presented to a given user

## Installation
    % gem install gxapi_rails
    % rails g gxapi_rails

## Configuration

First, you must create a Google Analytics Service Account.
Information can be found here https://developers.google.com/accounts/docs/OAuth2ServiceAccount

Gxapi uses `#{Rails.root}/config/gxapi.yml` to configure variables.  You can
use different configurations per environment (development/test/production)

### Example Configuration
    development:
      google_analytics:
        account_id: ACCOUNT_ID
        profile_id: PROFILE_ID
        web_property_id: WEB_PROPERTY_ID
      google:
        email: SERVICE_ACCOUNT_EMAIL
        private_key_path: 'PATH_TO_SERVICE_ACCOUNT_PRIVATE_KEY'

### Where the F do I find all this stuff?

<a href="https://developers.google.com/accounts/docs/OAuth2ServiceAccount"
target="_blank">Some instructions</a>
on creating a Google Analytics Service Account

<a href="https://developers.google.com/analytics/resources/concepts/gaConceptsAccounts"
target="_blank">Some instructions</a> on where to find your
Google Analytics settings


## Usage

### Layout

First, add the experiment Javascript in your layout

    # app/views/layouts/application.html.erb
    <%= gxapi_experiment_js %>

This will render the Javascript tag to pull in the Google Analytics
Content Experiment JS source and the corresponding call to send your
experiment data in.  If you are specifying a custom variable name
(the default is 'variant'), you can add it here as well

    # app/views/layouts/application.html.erb
    <%= gxapi_experiment_js(:custom_var) %>

If you have multiple experiments on the same page, you can add this same
call for each experiment in a view

    # app/views/layouts/application.html.erb
    <%= gxapi_experiment_js %>

    # app/views/posts/index.html.erb
    <%= gxapi_experiment_js(:other_experiment_key) %>

### Controller

Once you have set up an experiment in Google Analytics, you can reference
it by name in your controller

    class PostsController < ApplicationController

      def index
        gxapi_get_variant("My Experiment")
      end

    end

This will use the default key `variant` for the experiment.  To customize
this name, you can pass another value.  This is useful when you are using
multiple experiments on the same page

    class PostsController < ApplicationController

      def index
        gxapi_get_variant("My Experiment", :exp1)
        gxapi_get_variant("Other Experiment", :exp2)
      end

    end

### Control Flow

In the controller and the view, you can access the variant name to determine
which path your code should take.  `gxapi_get_variant` returns a
`Celluloid::Future` to prevent blocking while the result is calculated.  To
get the value in the controller you can call `#value`.  In the view,
Gxapi provides a the `gxapi_variant_name` helper.

    class PostsController < ApplicationController

      def index
        gxapi_get_variant("My Experiment", :exp1)
      end

    end

    # app/views/posts/index.html.erb

    <h1>My Page</h1>

    <% if gxapi_variant_name(:exp1) == 'version_a' -%>
      Do some stuff...
    <% else -%>
      Do some other stuff...
    <% end -%>

### Testing and overriding

In order to view a specific version, you can pass `experiment_name=value`
into a GET request to the action. This will short-circuit the rendering
of the JavaScript call to Google so testing will not affect your results

    # GET /posts.html?variant=my_var

    # app/controllers/posts_controller.rb
    def index
      gxapi_get_variant("My Experiment")
    end

    # app/views/posts/index.html.erb
    gxapi_variant_name # Will always equal 'my_var'


    # GET /posts.html?custom_name=my_var

    # app/controllers/posts_controller.rb
    def index
      gxapi_get_variant("My Experiment", :custom_name)
    end

    # app/views/posts/index.html.erb
    gxapi_variant_name(:custom_name) # Will always equal 'my_var'


## Contributing to gxapi

* Check out the latest master to make sure the feature hasn't been implemented or the bug hasn't been fixed yet.
* Check out the issue tracker to make sure someone already hasn't requested it and/or contributed it.
* Fork the project.
* Start a feature/bugfix branch.
* Commit and push until you are happy with your contribution.
* Make sure to add tests for it. This is important so I don't break it in a future version unintentionally.
* Please try not to mess with the Rakefile, version, or history. If you want to have your own version, or is otherwise necessary, that is fine, but please isolate to its own commit so I can cherry-pick around it.

## Copyright

Copyright (c) 2013 Dan Langevin. See LICENSE.txt for
further details.
