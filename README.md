# GitHub.jl

[![GitHub](http://pkg.julialang.org/badges/GitHub_0.4.svg)](http://pkg.julialang.org/?pkg=GitHub)
[![GitHub](http://pkg.julialang.org/badges/GitHub_0.5.svg)](http://pkg.julialang.org/?pkg=GitHub)
[![Build Status](https://travis-ci.org/JuliaWeb/GitHub.jl.svg?branch=master)](https://travis-ci.org/JuliaWeb/GitHub.jl)
[![Build status](https://ci.appveyor.com/api/projects/status/gmlm8snv03aw5pwq/branch/master?svg=true)](https://ci.appveyor.com/project/jrevels/github-jl-lj49i/branch/master)
[![Coverage Status](https://coveralls.io/repos/JuliaWeb/GitHub.jl/badge.svg?branch=master&service=github)](https://coveralls.io/github/JuliaWeb/GitHub.jl?branch=master)

## Quick start

```julia
julia> Pkg.add("GitHub")

julia> using GitHub

julia> my_auth = authenticate("an_access_token_for_your_account")
GitHub Authorization (8caaff**********************************)

julia> star("JuliaWeb/GitHub.jl"; auth = my_auth)  # :)
```

## API

### Keyword Arguments

The following table describes the various keyword arguments accepted by API methods that make requests to GitHub:

| keyword        | type                    | default value            | description                                                                                    |
|----------------|-------------------------|--------------------------|------------------------------------------------------------------------------------------------|
| `auth`         | `GitHub.Authorization`  | `GitHub.AnonymousAuth()` | The request's authorization                                                                    |
| `params`       | `Dict`                  | `Dict()`                 | The request's query parameters                                                                 |
| `headers`      | `Dict`                  | `Dict()`                 | The request's headers                                                                          |
| `handle_error` | `Bool`                  | `true`                   | If `true`, a Julia error will be thrown in the event that GitHub's response reports an error.  |
| `page_limit`   | `Real`                  | `Inf`                    | The number of pages to return (only applies to paginated results, obviously)                   |

### `GitHubType`s

GitHub's JSON responses are parsed into types `G<:GitHub.GitHubType`. Here's some useful information about these types:

- All their fields are `Nullable`.
- Their field names generally match the corresponding field in GitHub's JSON representation (the exception is `"type"`, which has the corresponding field name `typ` to avoid the obvious language conflict).
- These types can be passed as arguments to API methods in place of (and in combination with) regular identifying values (e.g. `create_status(repo, sha)` could be called as `create_status(::GitHub.Repo, ::AbstractString)`, or `create_status(::GitHub.Repo, ::GitHub.Commit)`, etc.)

Here's a table that matches up the provided `GitHubType`s with their corresponding API documentation:

| type          | link(s) to documentation                                                                                |
|---------------|---------------------------------------------------------------------------------------------------------|
| `Owner`       | [organizations](https://developer.github.com/v3/orgs/), [users](https://developer.github.com/v3/users/) |
| `Repo`        | [repositories](https://developer.github.com/v3/repos/)                                                  |
| `Commit`      | [repository commits](https://developer.github.com/v3/repos/commits/)                                    |
| `Content`     | [repository contents](https://developer.github.com/v3/repos/contents/)                                  |
| `Comment`     | [repository comments](https://developer.github.com/v3/repos/comments/)                                  |
| `Status`      | [commit statuses](https://developer.github.com/v3/repos/statuses/)                                      |
| `PullRequest` | [pull requests](https://developer.github.com/v3/pulls/)                                                 |
| `Issue`       | [issues](https://developer.github.com/v3/issues/)                                                       |

### Methods

Here are the methods exported by GitHub.jl, along with their return types and links to the corresponding GitHub API requests:

| signature                         | return type           | documentation                                                                                                                                   |
|-----------------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| `owner(login, isorg = false)`     | `Owner`               | [get a user](https://developer.github.com/v3/users/#get-a-single-user), [get an org](https://developer.github.com/v3/orgs/#get-an-organization) |
| `orgs(owner)`                     | `Vector{Owner}`       |
| `followers(owner)`                | `Vector{Owner}`       |
| `following(owner)`                | `Vector{Owner}`       |
| `repos(owner, isorg = false)`     | `Vector{Repo}`        |
| `repo(owner)`                     | `Repo`                |
| `create_fork(repo)`               | `Repo`                |
| `forks(repo)`                     | `Vector{Repo}`        |
| `contributors(repo)`              | `Vector{Owner}`       |
| `collaborators(repo)`             | `Vector{Owner}`       |
| `iscollaborator(repo, user)`      | `Bool`                |
| `add_collaborator(repo, user)`    | `HttpCommon.Response` |
| `remove_collaborator(repo, user)` | `HttpCommon.Response` |
| `stats(repo, stat, attempts = 3)` | `HttpCommon.Response` |
| `commit(repo, sha)`               | `Commit`              |
| `commits(repo)`                   | `Vector{Commit}`      |
| `file(repo, path)`                | `Content`             |
| `directory(repo, path)`           | `Vector{Content}`     |
| `create_file(repo, path)`         | `Dict`                |
| `update_file(repo, path)`         | `Dict`                |
| `delete_file(repo, path)`         | `Dict`                |
| `readme(repo)`                    | `Content`             |
| `create_status(repo, sha)`        | `Status`              |
| `statuses(repo, ref)`             | `Vector{Status}`      |
| `pull_requests(repo)`             | `Vector{PullRequest}` |
| `pull_request(repo, pr)`          | `PullRequest`         |
| `issue(repo, number)`             | `Issue`               |
| `issues(repo)`                    | `Vector{Issue}`       |
| `create_issue(repo)`              | `Issue`               |
| `edit_issue(repo, number)`        | `Issue`               |
| `issue_comments(repo, number)`    | `Vector{Comment}`     |
| `star(repo)`                      | `HttpCommon.Response` |
| `unstar(repo)`                    | `HttpCommon.Response` |
| `stargazers(user)`                | `Vector{Owner}`       |
| `starred(user)`                   | `Vector{Repo}`        |
| `watchers(repo)`                  | `Vector{Owner}`       |
| `watched(owner)`                  | `Vector{Repo}`        |
| `watch(repo)`                     | `HttpCommon.Response` |
| `unwatch(repo)`                   | `HttpCommon.Response` |

All API methods accept a keyword `auth` of type `GitHub.Authorization`. By default, this parameter will be an instance of `AnonymousAuth`, and the API request will be made without any privileges.

If you would like to make requests as an authorized user, you need to `authenticate`.

```julia
authenticate(token::String)
```
- `token` is an "access token" which you can [read about generating here](https://help.github.com/articles/creating-an-access-token-for-command-line-use)


### Users

The `User` type is used to represent GitHub accounts. It contains lots of interesting information about an account, and can be used in other API requests.

```julia
user(username; auth = AnonymousAuth())
```
- `username` is the GitHub login
- if you provide `auth` potentially much more user information will be returned

```julia
followers(user::String)
followers(user::User)
```
```julia
following(user::String)
following(user::User)
```
- `user` is either a GitHub login or `User` type
- the returned data will be an array of `User` types


### Organizations

Organizations let multiple users manage repositories together.

```julia
org(name; auth = AnonymousAuth())
```
- `name` is the GitHub organization login name

```julia
orgs(user::String; auth = AnonymousAuth())
```
- `user` is the GitHub account about which you are curious


### Repos

The `Repo` type is used to represent a repository hosted by GitHub. It contains all sorts of useful information about a repositories usage and history.

```julia
repo(owner, repo_name; auth = AnonymousAuth())
```
- `owner` is the GitHub login of the `User` or `Organization` that manages the repo
- `repo_name` is the repositories name on GitHub


```julia
repos(owner::Owner; auth = AnonymousAuth(),
                   typ = nothing,
                   sort = nothing,
                   direction = nothing)
```
- `owner` is a `User` or `Organization`
- `typ` is "all", "member", or "owner" (the default) for User
- `typ` is "all" (the default), "public", "private", "forks", "sources", or "member".
- `sort` is "created", "updated", "pushed", or "full_name" (the default).
- `direction` is "asc" or "desc" (the default).


```julia
contributors(owner, repo; auth = AnonymousAuth()
                          include_anon = false)
```
- `owner` is the GitHub login of the `User` or `Organization` that manages the repo
- `repo` is the repositories name on GitHub
- `include_anon` will tell GitHub to include anonymous contributions


### Statistics

Repository statistics are interesting bits of information about activity. GitHub caches this data when possible, but sometimes a request will trigger regeneration and come back empty. For this reason all statistics functions have an argument `attempts` which will be the number of tries made before admitting defeat.

```julia
contributor_stats(owner, repo, attempts = 3; auth = AnonymousAuth())
```
```julia
commit_activity(owner, repo, attempts = 3; auth = AnonymousAuth())
```
```julia
code_frequency(owner, repo, attempts = 3; auth = AnonymousAuth())
```
```julia
participation(owner, repo, attempts = 3; auth = AnonymousAuth())
```
```julia
punch_card(owner, repo, attempts = 3; auth = AnonymousAuth())
```
- `owner` is a GitHub login
- `repo` is a repository name
- `attempts` is the number of tries made before admitting defeat


### Forks

```julia
forks(owner, repo; auth = AnonymousAuth())
```
```julia
fork(owner, repo, organization = ""; auth = AnonymousAuth())
```
- `owner` is a GitHub login
- `repo` is a repository name


### Starring

```julia
stargazers(owner, repo; auth = AnonymousAuth())
```
```julia
starred(user; auth = AnonymousAuth())
```
```julia
star(owner, repo; auth = AnonymousAuth())
```
```julia
unstar(owner, repo; auth = AnonymousAuth())
```
- `owner` is a GitHub login
- `repo` is a repository name
- `user` is a GitHub login


### Watching

```julia
watchers(owner, repo; auth = AnonymousAuth())
```
```julia
watched(user; auth = AnonymousAuth())
```
```julia
watching(owner, repo; auth = AnonymousAuth())
```
```julia
watch(owner, repo; auth = AnonymousAuth())
```
```julia
unwatch(owner, repo; auth = AnonymousAuth())
```
- `owner` is a GitHub login
- `repo` is a repository name
- `user` is a GitHub login


### Collaborators

Collaborators are users that work together and share access to a repository.

```julia
collaborators(owner, repo; auth = AnonymousAuth())
```
```julia
iscollaborator(owner, repo, user; auth = AnonymousAuth())
```
```julia
add_collaborator(owner, repo, user; auth = AnonymousAuth()
```
```julia
remove_collaborator(owner, repo, user; auth = AnonymousAuth())
```
- `owner` is a GitHub login
- `repo` is a repository name
- `user` is the GitHub login being inspected, added, or removed

#### Examples
```julia
julia> using GitHub

julia> collaborators("JuliaLang","Julia")
26-element Array{Any,1}:
 User - amitmurthy
 User - andreasnoackjensen
 ⋮
 User - tshort
 User - vtjnash

julia> o = org("JuliaLang")
User - JuliaLang (The Julia Language, http://julialang.org/)

julia> collaborators(o,"julia")
26-element Array{Any,1}:
 User - amitmurthy
 User - andreasnoackjensen
 ⋮
 User - tshort
 User - vtjnash

julia> r = repo("JuliaLang","julia")
Repo - JuliaLang/julia (http://julialang.org/)
"The Julia Language: A fresh approach to technical computing."

julia> collaborators(r)
26-element Array{Any,1}:
 User - amitmurthy
 User - andreasnoackjensen
 ⋮
 User - tshort
 User - vtjnash
```


### Issues

The `Issue` type is used to represent issues and pull requests made against repositories.

```julia
issue(owner, repo, num; auth = AnonymousAuth())
```
- `owner` is a GitHub login or `User` type
- `repo` is the name of a repository
- `num` is the issue numer

```julia
issues(owner, repo; auth = AnonymousAuth(),
                    milestone = nothing,
                    state = nothing,
                    assignee = nothing,
                    creator = nothing,
                    mentioned = nothing,
                    labels = nothing,
                    sort = nothing,
                    direction = nothing,
                    since = nothing)
```
- `owner` is a GitHub login or `User` type
- `repo` is a repository name
- `milestone` can be an int or string ("*" matches all milestones, "none" returns issues with no milestone)
- `state` can be "open" or "closed"
- `assignee` can be the name of a user ("*" matches all users, "none" returns issues with no assignee)
- `creator` can be the user that created the issue
- `mentioned` is for any user mentioned in the issue
- `labels` is an array of labels to match
- `sort` can be "created", "updated", or "comments" (defaults to "created")
- `direction` can be "asc" or "desc" (defaults to "desc")
- `since` can be an ISO 8601 format (YYYY-MM-DDTHH:MM:SSZ) string

```julia
create_issue(owner, repo, title; auth = AnonymousAuth(),
                                 body = nothing,
                                 assignee = nothing,
                                 milestone = nothing,
                                 labels = nothing)
```
- `owner` is a GitHub login or `User` type
- `repo` is a repository name
- `title` is the title of your new issue
- `body` can be a text description of your issue
- `assignee` is a GitHub login
- `milestone` is the milestone number
- `labels` is an array of label strings

```julia
edit_issue(owner, repo, num; auth = AnonymousAuth(),
                             title = nothing,
                             body = nothing,
                             assignee = nothing,
                             state = nothing,
                             milestone = nothing,
                             labels = nothing)
```
- `owner` is a GitHub login or `User` type
- `repo` is a repository name
- `num` is the issue number
- `title` can be a new title for the issue
- `body` can be a new body for the issue
- `assignee` can be the new assignee
- `state` can be "open" or "closed"
- `milestone` can be the milestone number
- `labels` can be an array of label strings

### Comments

The `Comment` type is used to represent comments on Github issues.

```julia
comments(owner, repo, num; auth = AnonymousAuth())
```
- `owner` is a GitHub login or `User` type
- `repo` is a repository name
- `num` is the issue number
