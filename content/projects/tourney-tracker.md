---
title: "Tourney Tracker"
date: 2023-04-22T23:14:00-07:00
draft: true
---

This project was made to practice/review some concepts discussed in Alex Edwards's _Let's Go_ and _Let's Go Further_ books. I also wanted to get a feel for how I wanted to structure my (web) applications in the future. Below is some discussion on the various sources that helped influence how this project came to be structured.

## _Let's Go_/Snippetbox

I repurposed a lot of the template-handling code from the [Snippetbox](https://github.com/ejacobg/snippetbox) project into the Tourney Tracker.

Storing templates in a `map[string]*template.Template` seems like the easiest way to access your views. I tried experimenting with a `Views` struct that looked something like this:

```go
package view

import "html/template"

type Views struct {
	Index, View, Edit *template.Template
}
```

But this ended up not working out as well as I'd hoped since I soon found out I would need a lot more views than what this struct was capable of representing.

Using a map to hold all my templates made rendering very simple with the use of the [`Server.Render()`](https://github.com/ejacobg/tourney-tracker/blob/f6c567123d6d2ebfd6754570333169318aca4c3c/http/helpers.go#L13) method:

```go
package http

import (
	"bytes"
	"fmt"
	"net/http"
)

// Render will execute the "name" template of "tmpl", then write it to the response with the given status code.
func (s *Server) Render(w http.ResponseWriter, status int, tmpl, name string, data any) {
	t, ok := s.Templates[tmpl]
	if !ok {
		ServerErrorResponse(w, fmt.Sprintf("The template %q does not exist.", tmpl))
		return
	}

	buf := new(bytes.Buffer)

	err := t.ExecuteTemplate(buf, name, data)
	if err != nil {
		ServerErrorResponse(w, fmt.Sprintf("Failed to render template: %s", err))
		return
	}

	w.WriteHeader(status)
	buf.WriteTo(w)
}
```

Similar to the Snippetbox project, I've placed all my templates in the [`ui/`](https://github.com/ejacobg/tourney-tracker/tree/main/ui) directory, and parse them with a [`newTemplateCache()`](https://github.com/ejacobg/tourney-tracker/blob/f6c567123d6d2ebfd6754570333169318aca4c3c/cmd/tournaments/main.go#L57) function.

## _Let's Go Further_/Greenlight

The [Greenlight](https://github.com/ejacobg/greenlight) project implemented a JSON API, so I couldn't directly make use of its code like I could with Snippetbox. However, I did find use for the error handling and database code.

I liked how the Greenlight project made use of [specialty handlers](https://github.com/ejacobg/greenlight/blob/main/cmd/api/errors.go) for the various error codes that were returned. What I didn't like was how these handlers were defined as methods on the `application` struct, when they would probably be more useful as standalone functions.

[My implementation](https://github.com/ejacobg/tourney-tracker/blob/main/http/errors.go) is currently quite limited (only takes error strings, and only writes to the standard logger), but provides a good starting point if I never need something more robust.

```go
package http

import (
	"log"
	"net/http"
)

// ErrorResponse responds with and logs the given error message to the console, alongside the given response code.
// Any error messages will be rendered inside the #error element of the page.
func ErrorResponse(w http.ResponseWriter, error string, code int) {
	log.Println(error)

	w.Header()["HX-Reswap"] = []string{"innerHTML"}
	w.Header()["HX-Retarget"] = []string{"#error"}
	http.Error(w, error, code)
}

func ServerErrorResponse(w http.ResponseWriter, error string) {
	ErrorResponse(w, error, http.StatusInternalServerError)
}

func BadRequestResponse(w http.ResponseWriter, error string) {
	ErrorResponse(w, error, http.StatusBadRequest)
}

func UnprocessableEntityResponse(w http.ResponseWriter, error string) {
	ErrorResponse(w, error, http.StatusUnprocessableEntity)
}

// More error handlers...
```

Both the Tourney Tracker and Greenlight projects made use of a PostgreSQL database, so the database code between the two projects ended up looking pretty similar. I copied the use of the [`golang-migrate/migrate`](https://github.com/golang-migrate/migrate) tool for use in the Tourney Tracker project.

## Phoenix

I'm currently working with the Phoenix Framework in one of my classes, and the default views made by the generator commands (e.g. `mix phx.gen.html`) made sense for what I needed, so I tried to make something similar for the Tourney Tracker. Ultimately, the UI is basically just a bunch of tables, so there isn't too much to say about the design. I reused the CSS on this site for the Tourney Tracker, so I didn't have to mess with that at all.

What I did have to mess with were the different inputs into each template. The standard `template` package doesn't provide a way to enforce what pieces of data are needed by a template. I believe that there are some templating packages that do provide this, but in my case I just settled for using comments.

```
{{- /*
  Renders a table showing all the tournaments with the given Tier.

  Data:
    .Tier:  Tier
    .Names: []Name
*/ -}}
```

The `{{template}}`, `{{block}}`, and `{{with}}` actions only take a single argument, making it difficult to render sub-templates that require multiple pieces of data. It is possible to pass in multiple values using a [custom function](https://stackoverflow.com/a/18276968), but in this case I opted to just copy over the components I needed. In the future, I will make use of this function.

## HTMX

I used HTMX to implement all of the `Edit` buttons in the application. This helped clean up the code for the templates since I could separate the editable components from the static ones. It also meant that I didn't have to wrap everything in a `<form>` element, and could be more granular with my changes. If I wanted to edit a player, I could just send the changes for that player rather than submitting the entire entrant list for every update.

By default, HTMX won't render any non-2XX responses. This can make it a little difficult to render any RESTful status codes when dealing with validation errors. I found it quite difficult to implement "conditional rendering" using HTMX, where a successful response would swap in the given content in one place, and an error response would swap in the content in another place. I spent a lot of time trying to figure this out, but ended up settling on just rendering all my errors in a single spot on the page, rather than rendering them inline. If I were to do this again, I would either (1) not bother trying to render errors or (2) just return the components with the error message attached rather than returning just the error message itself.

## Standard Package Layout

The Snippetbox and Greenlight projects make use of the so-called ["Fat Service"](https://www.alexedwards.net/blog/the-fat-service-pattern) pattern, which while easy to navigate, feels contradictory to [other](https://dave.cheney.net/practical-go/presentations/gophercon-singapore-2019.html#_package_design) [sources](https://rakyll.org/style-packages/) [I've](https://www.ardanlabs.com/blog/2017/02/design-philosophy-on-packaging.html) [read](https://go.dev/blog/package-names) on package organization. This isn't to say it doesn't work, just that it doesn't work for me. As stated before, the purpose of this project was to develop some kind of structure I would be comfortable working with. Since my UI was already Phoenix-based, I started this project off as an MVC application, but quickly ran into some import cycles after trying to split off the code that called the Challonge and start.gg APIs from the code that contained my models. After a little research on packaging, I found this [slide deck](https://go-talks.appspot.com/github.com/gophercon/2016-talks/BenJohnson-StructuringApplicationsForGrowth/main.slide#1) and associated [article](https://medium.com/@benbjohnson/standard-package-layout-7cdbc8391fc1) by Ben Johnson. The core ideas I took away from this article are threefold:

1. Define core types and interfaces in the "root" package. This package is import-only.
2. Packages represent dependencies. These dependencies provide concrete implementations of the interfaces defined in the root package (e.g. a PostgreSQL implementation of a service).
3. You're allowed to redefine external packages (including the standard library).

Ben Johnson also provided an example project ([WTF Dial](https://github.com/benbjohnson/wtf)) using this package layout, and I've taken a lot of inspiration from it.

### The Root Package

You technically don't have to put your source files for the domain types in the project root. What's important is that they're all together, and that any necessary service interfaces are defined alongside them.

Your root package files will all end up looking pretty similar. Here is my [`tier.go`](https://github.com/ejacobg/tourney-tracker/blob/f6c567123d6d2ebfd6754570333169318aca4c3c/tier.go) file:

```go
package tourney_tracker

// Tier represents the relative importance of a Tournament.
type Tier struct {
	ID         int64  `json:"id"`
	Name       string `json:"name"`
	Multiplier int    `json:"multiplier"`
}

// TierService represents a service for managing tiers.
type TierService interface {
	// GetTiers returns all tiers.
	GetTiers() ([]Tier, error)

	// GetTier returns a single Tier by ID.
	GetTier(id int64) (Tier, error)

	// GetTournamentTier returns the Tier for the given Tournament.
	GetTournamentTier(tournamentID int64) (Tier, error)

	// CreateTier adds the given Tier to the database.
	CreateTier(tier *Tier) error

	// UpdateTier updates the given Tier.
	UpdateTier(tier *Tier) error

	// DeleteTier deletes the given Tier.
	// Note that deleting a tier that still has tournaments attached to it should fail.
	// It is up to the user to ensure that all tournaments update their Tier before attempting to delete.
	DeleteTier(id int64) error
}
```

### Packages as Dependencies

The other packages in your project might provide implementations for the services defined in the root package. It is then the job of the `main` package to choose which implementations are going to be used, and to wire everything up correctly.

For example, my custom [`http.Server`](https://github.com/ejacobg/tourney-tracker/blob/f6c567123d6d2ebfd6754570333169318aca4c3c/http/server.go#L12) type contains fields representing the various services it makes use of:

```go
package http

import (
	tournament "github.com/ejacobg/tourney-tracker"
)

// Server provides several HTTP handlers for servicing tournament-related requests.
type Server struct {
	// Other fields...

	// Services used by the various HTTP routes.
	EntrantService    tournament.EntrantService
	PlayerService     tournament.PlayerService
	TierService       tournament.TierService
	TournamentService tournament.TournamentService
}
```

In my [`main`](https://github.com/ejacobg/tourney-tracker/blob/f6c567123d6d2ebfd6754570333169318aca4c3c/cmd/tournaments/main.go#L37), all I have to do is fill these services in with the correct implementation:

```go
package main

import (
	"github.com/ejacobg/tourney-tracker/http"
	"github.com/ejacobg/tourney-tracker/postgres"
)

func main() {
	// ...

	srv := http.NewServer(...)

	srv.EntrantService = postgres.EntrantService{db}
	srv.PlayerService = postgres.PlayerService{db}
	srv.TierService = postgres.TierService{db}
	srv.TournamentService = postgres.TournamentService{db}

	// ...
}
```

### Redefining Packages

As shown above, I replaced the `net/http`'s `Server` definition with my own, and using my own `http` package rather than the standard library's version. Ideally, all necessary interactions with the HTTP protocol should be provided by my redefined `http` package. This way redefined package will act as an adapter between your application and the original package. This can be useful if you're only using a modified subset of the original package, or if you would like to modify the ergonomics of the original package.

## Future Work

This project isn't perfect, but I've gotten what I want out of it, and probably won't touch this for a while. If I do decide to come back to it, here are some of the things I might add.

Testing using mocks, concurrency control during updates, logging, panic recovery, deployment.
