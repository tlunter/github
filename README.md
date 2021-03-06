[![Application icon](https://github.com/peter-murach/github/raw/master/icons/github_api.png)][icon]
[icon]: https://github.com/peter-murach/github/raw/master/icons/github_api.png
# github_api
[![Gem Version](https://badge.fury.io/rb/github_api.png)][gem]
[![Build Status](https://secure.travis-ci.org/peter-murach/github.png?branch=master)][travis]
[![Dependency Status](https://gemnasium.com/peter-murach/github.png?travis)][gemnasium]
[![Code Climate](https://codeclimate.com/github/peter-murach/github.png)][codeclimate]
[![Coverage Status](https://coveralls.io/repos/peter-murach/github/badge.png?branch=master)][coveralls]

[gem]: http://badge.fury.io/rb/github_api
[travis]: http://travis-ci.org/peter-murach/github
[gemnasium]: https://gemnasium.com/peter-murach/github
[codeclimate]: https://codeclimate.com/github/peter-murach/github
[coveralls]: https://coveralls.io/r/peter-murach/github

[Website](http://peter-murach.github.io/github/) | [Wiki](https://github.com/peter-murach/github/wiki) | [RDocs](http://rubydoc.info/github/peter-murach/github/master/frames)

A Ruby client for the official GitHub API.

Supports all the API methods. It's built in a modular way. You can either instantiate the whole API wrapper Github.new or use parts of it i.e. Github::Client::Repos.new if working solely with repositories is your main concern. Intuitive query methods allow you easily call API endpoints.

## Features

* Intuitive GitHub API interface navigation.
* It's comprehensive. You can request all GitHub API resources.
* Modular design allows for working with parts of API.
* Fully customizable including advanced middleware stack construction.
* Supports OAuth2 authorization.
* Flexible argument parsing. You can write expressive and natural queries.
* Requests pagination with convenient DSL and automatic options.
* Easy error handling split for client and server type errors.
* Supports multithreaded environment.
* Custom media type specification through the 'media' parameter.
* Request results caching
* Fully tested with unit and feature tests hitting the live api.

## Installation

Install the gem by running

```ruby
gem install github_api
```

or put it in your Gemfile and run `bundle install`

```ruby
gem "github_api"
```

## Contents

* [1. Usage](#1-usage)
    * [1.1 API Navigation](#11-api-navigation)
    * [1.2 Modularity](#12-modularity)
    * [1.3 Arguments](#13-arguments)
    * [1.4 Response Querying](#14-respone-querying)
* [2. Configuration](#2-configuration)
    * [2.1 Basic](#21-configuration)
    * [2.2 Advanced](#22-configuration)
* [3. Authentication](#3-authentication)
    * [3.1 Basic](#31-basic)
    * [3.2 Application OAuth](#32-application-oauth)
    * [3.3 Authorizations API](#33-authorizations-api)
* [5. SSL](#5-ssl)
* [6. API](#6-api)
* [7. Media Types](#7-media-types)
* [8. Hypermeida](#8-hypermedia)
* [9. Configuration](#9-configurations)
* [10. Pagination](#10-pagination)
* [11. Caching](#11-caching)
* [12. Debugging](#12-debugging)
* [13. Error Handling](#13-error-handling)
* [14. Response Message](#14-response-message)
* [15. Examples](#15-examples)
* [16. Testing](#16-testing)

## 1 Usage

To start using the gem, you can either perform requests directly on `Github` namespace:

```ruby
Github.repos.list user: 'peter-murach'
```

or create a new client instance like so

```ruby
github = Github.new
```

and then call api methods, for instance, to list a given user repositories do

```ruby
github.repos.list user: 'peter-murach'
```

### 1.1 API Navigation

The **github_api** closely mirrors the [GitHub API](https://developer.github.com/v3/) hierarchy. For example, if you want to create a new file in a repository, look up the GitHub API spec. In there you will find contents sub category underneath the repository category. This would translate to the request:

```ruby
github = Github.new
github.repos.contents.create 'peter-murach', 'finite_machine', 'hello.rb',
                             path: 'hello.rb',
                             content: "puts 'hello ruby'"
```

The whole library reflects the same api navigation. Therefore, if you need to list releases for a repository do:

```ruby
github.repos.releases.list 'peter-murach', 'finite_machine'
```

or to list a user's followers:

```ruby
github.users.followers.list 'peter-murach'
```

The code base has been extensively documented with examples of how to use each method. Please refer to the [documentation](http://rubydoc.info/github/peter-murach/github/master/frames) under the `Github::Client` class name.

### 1.2 Modularity

The code base is modular. This means that you can work specifically with a given part of GitHub API. If you want to only work with activity starring API do the following:

```ruby
starring = Github::Client::Activity::Starring.new
starring.star 'peter-murach', 'github'
```

Please refer to the [documentation](http://rubydoc.info/github/peter-murach/github/master/frames) and look under `Github::Client` to see all available classes.

### 1.3 Arguments

The **github_api** library allows for flexible argument parsing.

Arguments can be passed directly inside the method called. The `required` arguments are passed in first, followed by optional parameters supplied as hash options:

```ruby
issues = Github::Issues.new
issues.milestones.list 'peter-murach', 'github', state: 'open'
```

In the previous example, the order of arguments is important. However, each method also allows you to specify `required` arguments using hash symbols and thus remove the need for ordering. Therefore, the same example could be rewritten like so:

```ruby
issues = Github::Issues.new
issues.milestones.list user: 'peter-murach', repo: 'github', state: 'open'
```

Furthermore, `required` arguments can be passed during instance creation:

```ruby
issues = Github::Issues.new user: 'peter-murach', repo: 'github'
issues.milestones.list state: 'open'
```

Similarly, the `required` arguments for the request can be passed inside the current scope such as:

```ruby
issues = Github::Issues.new
issues.milestones(user: 'peter-murach', repo: 'github').list state: 'open'
```

But why limit ourselves? You can mix and match arguments, for example:

```ruby
issues = Github::Issues.new user: 'peter-murach'
issues.milestones(repo: 'github').list
issues.milestones(repo: 'tty').list
```

You can also use a bit of syntactic sugar whereby "username/repository" can be passed as well:

```ruby
issues = Github::Issues.new
issues.milestones('peter-murach/github').list
issues.milestones.list 'peter-murach/github'
```

Finally, use the `with` scope to clearly denote your requests

```ruby
issues = Github::Issues.new
issues.milestones.with(user: 'peter-murach', repo: 'github').list
```

Please consult the method [documentation](http://rubydoc.info/github/peter-murach/github/master/frames) or [GitHub specification](https://developer.github.com/v3/) to see which arguments are required and what are the option parameters.

### 1.4 Response Querying

The response is of type `Github::ResponseWrapper` and allows traversing all the json response attributes like method calls.
In addition, if the response returns more than one resource, these will be automatically yielded to the provided block one by one.

```ruby
repos = Github::Client::Repos.new 
repos.branches user: 'peter-murach', repo: 'github' do |branch|
  puts branch.name
end
```

## 2 Configuration

The **github_api** provides ability to specify global configruation options. These options will be available to all api calls.

### 2.1 Basic

The configuration options can be set by using the `configure` helper

```ruby
Github.configure do |c|
  c.basic_auth = "login:password"
  c.adapter    = :typheous
  c.user       = 'peter-murach'
  c.repo       = 'finite_machine'
end
```

Alternatively, you can configure the settings by passing a block to an instance like:

```ruby
Github.new do |c|
  c.endpoint    = 'https://github.company.com/api/v3'
  c.site        = 'https://github.company.com'
end
```

or simply by passing hash of options to an instance like so

```ruby
github = Github.new basic_auth: 'login:password',
                    adapter: :typheous,
                    user: 'peter-murach',
                    repo: 'finite_machine'
```

The following is the full list of available configuration options:

```ruby
adapter            # Http client used for performing requests. Default :net_http
auto_pagination    # Automatically traverse requests page links. Default false
basic_auth         # Basic authentication in form login:password.
client_id          # Oauth client id.
client_secret      # Oauth client secret.
connection_options # Hash of connection options.
endpoint           # Enterprise API endpoint. Default: 'https://api.github.com'
oauth_token        # Oauth authorization token.
org                # Global organization used in requests if none provided
per_page           # Number of items per page. Max of 100. Default 30.
repo               # Global repository used in requests in none provided
site               # enterprise API web endpoint
ssl                # SSL settings in hash form.
user               # Global user used for requests if none provided
user_agent         # Custom user agent name. Default 'Github API Ruby Gem'
```

### 2.2 Advanced

The **github_api** will use the default middleware stack which is exposed by calling `stack` on a client instance. However, this stack can be freely modified with methods such as `insert`, `insert_after`, `delete` and `swap`. For instance, to add your `CustomMiddleware` do:

```ruby
Github.configure do |c|
  c.stack.insert_after Github::Response::Helpers, CustomMiddleware
end
```

Furthermore, you can build your entire custom stack and specify other connection options such as `adapter` by doing:

```ruby
Github.new do |c|
  c.adapter :excon

  c.stack do |builder|
    builder.use Github::Response::Helpers
    builder.use Github::Response::Jsonize
  end
end
```



## 3 Authentication

### 3.1 Basic

To start making requests as authenticated user you can use your GitHub username and password like so

```ruby
Github.new basic_auth: 'login:password'
```

Though this method is convenient you should strongly consider using `OAuth` for improved security reasons.

### 3.2 Application OAuth

In order to authenticate your app through OAuth2 on GitHub you need to

* Visit https://github.com/settings/applications/new and register your app.
  You will need to be logged in to initially register the application.

* Authorize your credentials https://github.com/login/oauth/authorize

You can use convenience methods to help you achieve this using **GithubAPI** gem:

```ruby
github = Github.new client_id: '...', client_secret: '...'
github.authorize_url redirect_uri: 'http://localhost', scope: 'repo'
# => "https://github.com/login/oauth/authorize?scope=repo&response_type=code&client_id='...'&redirect_uri=http%3A%2F%2Flocalhost"
```
After you get your authorization code, call to receive your access_token

```ruby
token = github.get_token( authorization_code )
```

Once you have your access token, configure your github instance following instructions under Configuration.

**Note**: If you are working locally (i.e. your app URL and callback URL are localhost), do not specify a ```:redirect_uri``` otherwise you will get a ```redirect_uri_mismatch``` error.

### 3.3 Authorizations API

#### 3.3.1 For an User

To create an access token through the GitHub Authrizations API, you are required to pass your basic credentials and scopes you wish to have for the authentication token.

```ruby
github = Github.new basic_auth: 'login:password'
github.oauth.create scopes: ['repo']
```

You can add more than one scope from the `user`, `public_repo`, `repo`, `gist` or leave the scopes parameter out, in which case, the default read-only access will be assumed (includes public user profile info, public repo info, and gists).

#### 3.3.2 For an App

Furthermore, to create auth token for an application you need to pass `:app` argument together with `:client_id` and `:client_secret` parameters.

```ruby
github = Github.new basic_auth: 'login:password'
github.oauth.app.create 'client-id', scopes: ['repo']
```

In order to revoke auth token(s) for an application you must use basic authentication with `client_id` as login and `client_secret` as password.

```ruby
github = Github.new basic_auth: "client_id:client_secret"
github.oauth.app.delete 'client-id'
```

Revoke a specific app token.

```ruby
github.oauth.app.delete 'client-id', 'access-token'
```

### 3.4 Scopes

You can check OAuth scopes you have by:

```ruby
  github = Github.new :oauth_token => 'token'
  github.scopes.list    # => ['repo']
```

To list the scopes that the particular GitHub API action checks for do:

```ruby
  repos = Github::Repos.new
  res = repos.list :user => 'peter-murach'
  res.headers.accepted_oauth_scopes    # => ['delete_repo', 'repo', 'public_repo', 'repo:status']
```

To understand what each scope means refer to [documentation](http://developer.github.com/v3/oauth/#scopes)

## 5 SSL

By default requests over SSL are set to OpenSSL::SSL::VERIFY_PEER. However, you can turn off peer verification by

```ruby
  Github.new ssl: { verify: false }
```

If your client fails to find CA certs, you can pass other SSL options to specify exactly how the information is sourced

```ruby
  ssl: {
    client_cert: "/usr/local/www.example.com/client_cert.pem"
    client_key:  "/user/local/www.example.com/client_key.pem"
    ca_file:     "example.com.cert"
    ca_path:     "/etc/ssl/"
  }
```

For instance, download CA root certificates from Mozilla [cacert](http://curl.haxx.se/ca/cacert.pem) and point ca_file at your certificate bundle location.
This will allow the client to verify the github.com ssl certificate as authentic.

## 6 API

Main API methods are grouped into the following classes that can be instantiated on their own

```ruby
Github         - full API access

Github::Gists           Github::GitData    Github::Repos             Github::Search
Github::Orgs            Github::Issues     Github::Authorizations
Github::PullRequests    Github::Users      Github::Activity
```

Some parts of GitHub API v3 require you to be authenticated, for instance the following are examples of APIs only for the authenticated user

```ruby
Github::Users::Emails
Github::Users::Keys
```

All method calls form Ruby like sentences and allow for intuitive API navigation, for instance

```ruby
github = Github.new :oauth_token => '...'
github.users.followers.following 'wycats'  # => returns users that 'wycats' is following
github.users.followers.following? 'wycats' # => returns true if following, otherwise false
```

For specifications on all available methods, go to http://developer.github.com/v3/ or
read the rdoc. All methods are documented there with examples of usage.

Alternatively, you can find out which methods are supported by calling `actions` on a class instance in your `irb`:

```ruby
>> Github::Repos.actions                    >> github.issues.actions
---                                         ---
|--> all                                    |--> all
|--> branches                               |--> comments
|--> collaborators                          |--> create
|--> commits                                |--> edit
|--> contribs                               |--> events
|--> contributors                           |--> find
|--> create                                 |--> get
|--> downloads                              |--> labels
|--> edit                                   |--> list
|--> find                                   |--> list_repo
|--> forks                                  |--> list_repository
|--> get                                    |--> milestones
|--> hooks                                  ...
...
```

## 7 Media Types

You can specify custom media types to choose the format of the data you wish to receive. To make things easier you can specify the following shortcuts
`json`, `blob`, `raw`, `text`, `html`, `full`. For instance:

```ruby
github = Github.new
github.issues.get 'peter-murach', 'github', 108, media: 'text'
```

This will be expanded into `application/vnd.github.v3.text+json`

If you wish to specify the version, pass `media: 'beta.text'` which will be converted to `application/vnd/github.beta.text+json`.

Finally, you can always pass the whole accept header like so

```ruby
github.issues.get 'peter-murach', 'github', 108, accept: 'application/vnd.github.raw'
```

## 8 Hypermedia

TODO

## 10 Pagination

Any request that returns multiple items will be paginated to 30 items by default. You can specify custom `page` and `per_page` query parameters to alter default behavior. For instance:

```ruby
repos = Github::Repos.new
repos.list user: 'wycats', per_page: 10, page: 5
```

Then you can query the pagination information included in the link header by:

```ruby
res.links.first  # Shows the URL of the first page of results.
res.links.next   # Shows the URL of the immediate next page of results.
res.links.prev   # Shows the URL of the immediate previous page of results.
res.links.last   # Shows the URL of the last page of results.
```

In order to iterate through the entire result set page by page, you can use convenience methods:

```ruby
res.each_page do |page|
  page.each do |repo|
    puts repo.name
  end
end
```

or use `has_next_page?` and `next_page` like in the following:

```ruby
while res.has_next_page?
  ... process response ...
  res.next_page
end
```

Alternatively, you can retrieve all pages in one invocation by passing the `auto_pagination` option like so:

```ruby
  github = Github.new auto_pagination: true
```

Depending at what stage you pass the `auto_pagination` it will affect all or only a single request:

```ruby
  Github::Repos.new auto_pagination: true         # affects Repos part of API

  Github::Repos.new.list user: '...', auto_pagination: true  # affects a single request
```

One can also navigate straight to the specific page by:

```ruby
res.count_pages  # Number of pages
res.page 5       # Requests given page if it exists, nil otherwise
res.first_page   # Get first page
res.next_page    # Get next page
res.prev_page    # Get previous page
res.last_page    # Get last page
```

## 11 Caching

Caching is supported through the [`faraday-http-cache` gem](https://github.com/plataformatec/faraday-http-cache).

Add the gem to your Gemfile:

    gem 'faraday-http-cache'

You can now configure cache parameters as follows

```ruby
Github.configure do |config|
  config.stack do |builder|
    builder.use Faraday::HttpCache, store: Rails.cache
  end
end
```

More details on the available options can be found in the gem's own documentation: https://github.com/plataformatec/faraday-http-cache#faraday-http-cache

## 12 Debugging

run with ENV['DEBUG'] flag  or include middleware by passing `debug` flag

## 13 Error Handling

The generic error class `Github::Error::GithubError` will handle both the client (`Github::Error::ClientError`) and service (`Github::Error::ServiceError`) side errors. For instance in your code you can catch errors like

```ruby
begin
  # Do something with github_api gem
rescue Github::Error::GithubError => e
  puts e.message

  if e.is_a? Github::Error::ServiceError
    # handle GitHub service errors such as 404
  elsif e.is_a? Github::Error::ClientError
    # handle client errors e.i. missing required parameter in request
  end
end
```

## 14 Response Message

Each response comes packaged with methods allowing for inspection of HTTP start line and headers. For example, to check for rate limits and status codes, call

```ruby
res = Github::Repos.new.branches 'peter-murach', 'github'
res.headers.ratelimit_limit     # "5000"
res.headers.ratelimit_remaining # "4999"
res.headers.status              # "200"
res.headers.content_type        # "application/json; charset=utf-8"
res.headers.etag                # "\"2c5dfc54b3fe498779ef3a9ada9a0af9\""
res.headers.cache_control       # "public, max-age=60, s-maxage=60"
```

## 15 Examples

Some API methods require input parameters. These are simply added as a hash of properties, for instance

```ruby
issues = Github::Issues.new user:'peter-murach', repo: 'github-api'
issues.milestones.list state: 'open', sort: 'due_date', direction: 'asc'
```

Other methods may require inputs as an array of strings

```ruby
users = Github::Users.new oauth_token: 'token'
users.emails.add 'email1', 'email2', ..., 'emailn' # => Adds emails to the authenticated user
```

If a method returns a collection, you can iterate over it by supplying a block parameter,

```ruby
events = Github::Activity::Events.new
events.public do |event|
  puts event.actor.login
end
```

Query requests return boolean values instead of HTTP responses

```ruby
github = Github.new
github.orgs.members.member? 'github', 'technoweenie', public: true # => true
```

### 15.1 Rails Example

A Rails controller that allows a user to authorize their GitHub account and then performs a request.

```ruby
class GithubController < ApplicationController

  attr_accessor :github
  private :github

  def authorize
    github  = Github.new client_id: '...', client_secret: '...'
    address = github.authorize_url redirect_uri: 'http://...', scope: 'repo'
    redirect_to address
  end

  def callback
    authorization_code = params[:code]
    access_token = github.get_token authorization_code
    access_token.token   # => returns token value
  end
end
```

## 16 Testing

The test suite is split into two groups, `live` and `mock`.

The `live` tests are the ones in `features` folder and they simply exercise the GitHub API by making live requests and then being cached with VCR in directory named `features\cassettes`. For details on how to get set up, please navigate to the `features` folder.

The `mock` tests are in the `spec` directory and their primary concern is to test the gem internals without the hindrance of external calls.

## Development

Questions or problems? Please post them on the [issue tracker](https://github.com/peter-murach/github/issues). You can contribute changes by forking the project and submitting a pull request. You can ensure the tests are passing by running `bundle` and `rake`.

## Copyright

Copyright (c) 2011-2014 Piotr Murach. See LICENSE.txt for further details.
