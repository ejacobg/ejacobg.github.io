---
title: "Testing Services"
date: 2023-05-11T22:19:39-07:00
draft: false
---

Let's say that some parts of your code want to make use of a key-value store that looks something like this:

```go
package kv

// Store stores positive integers.
type Store interface {
	Get(key int) int    // Returns -1 if the key is not present.
	Put(key, value int) // Inserts a new item, updating it if present.
	Delete(key int)     // No-op if key is not present.
}
```

You might then have some concrete implementations for your store:

```go
package slices

import "golang.org/x/exp/slices"

// Store represents a kv.Store backed by a slice.
type Store struct {
	s []item
}

type item struct {
	key, value int
}

func NewStore(cap int) *Store {
	return &Store{
		s: make([]item, cap),
	}
}

func (s *Store) Get(key int) int {
	// ...
}

// More methods...
```

```go
package maps

// Store represents a kv.Store backed by a map.
type Store struct {
	s map[int]int
}

func NewStore() *Store {
	return &Store{
		s: make(map[int]int),
	}
}

func (s *Store) Get(key int) int {
	if value, ok := s.s[key]; ok {
		return value
	}

	return -1
}

// More methods...
```

You then realize that you should probably start testing your implementations, especially if you plan on making more of them. You come up with a set of [acceptance tests](https://quii.gitbook.io/learn-go-with-tests/testing-fundamentals/intro-to-acceptance-tests#acceptance-tests) that every `Store` should pass.

```go
package kvtest

import (
	"github.com/ejacobg/kv"
	"testing"
)

func TestStore(t *testing.T, s kv.Store) {
	var key, value, got, want int

	// Can read what was inserted.
	key, value = 1, 1
	s.Put(key, value)

	got, want = s.Get(key), 1
	if got != want {
		t.Errorf("inserted key %d returns %d, want %d", key, got, want)
	}

	// Can read what was updated.
	key, value = 1, 2
	s.Put(key, value)

	got, want = s.Get(key), 2
	if got != want {
		t.Errorf("updated key %d returns %d, want %d", key, got, want)
	}

	// Cannot read after a delete operation.
	key = 1
	s.Delete(key)

	got, want = s.Get(key), -1
	if got != want {
		t.Errorf("deleted key %d returns %d, want %d", key, got, want)
	}
}
```

To run these tests, all you have to do is inject your implementation into them:

```go
package maps

import (
	"github.com/ejacobg/kv/kvtest"
	"testing"
)

func TestAcceptance(t *testing.T) {
	kvtest.TestStore(t, NewStore())
}
```

This way, you can be sure that your stores are working as intended, and you don't have to copy over the same set of tests for each implementation. You are also free to define any other local tests if needed since the acceptance tests are black-box by nature.

You can find the full code for the above examples in this repository: https://github.com/ejacobg/kv

What if you have multiple tests, and need to perform some setup and teardown operations between each one? In this case, we might want to switch to a table-driven approach:

```go
package kvtest

import (
	"github.com/ejacobg/kv"
	"testing"
)

type Suite struct {
	S kv.Store

	// Optional setup and teardown.
	BeforeEach func(*testing.T)
	AfterEach  func(*testing.T)
}

func (s *Suite) TestStore(t *testing.T) {
	tests := []struct {
		name string
		fn   func(*testing.T, kv.Store)
	}{
		{"Store", TestStore},
	}

	if s.BeforeEach == nil {
		s.BeforeEach = func(t *testing.T) {}
	}

	if s.AfterEach == nil {
		s.AfterEach = func(t *testing.T) {}
	}

	for _, test := range tests {
		t.Run(test.name, func(t *testing.T) {
			s.BeforeEach(t)
			test.fn(t, s.S)
			s.AfterEach(t)
		})
	}
}

// Acceptance tests from the previous examples.
func TestStore(t *testing.T, s kv.Store) {
	// ...
}
```

To use these tests, you simply need to instantiate a `Suite` object:

```go
package maps

import (
	"github.com/ejacobg/kv/kvtest"
	"testing"
)

func TestSuite(t *testing.T) {
	suite := kvtest.Suite{}

	suite.BeforeEach = func(_ *testing.T) {
		suite.S = NewStore()
	}

	suite.TestStore(t)
}
```

You can perform both a one-time setup and teardown for all tests, as well as setup for each individual test using the [`BeforeEach` and `AfterEach` methods](https://www.calhoun.io/more-effective-ddd-with-interface-test-suites/#:~:text=//%20Optional%20%2D%20useful%20for%20resetting%20a%20DB%20perhaps.%0A%20%20BeforeEach%20func()%0A%20%20AfterEach%20func()).

```go
func TestSuite(t *testing.T) {
	// Set up suite...
	
	// Run suite...
	{
            suite := Suite{}
		
            // Set up test...
            suite.BeforeEach = func(t *testing.T) {
                // ...
                if err != nil {
                    t.Fatalf("setup failed: %v", err)
                }
            }
		
            // Tear down test...
            suite.AfterEach = func(t *testing.T) {
                // ...	
            }
            
            suite.Test(t)
        }
	
	// Tear down suite...
}
```

The [_Hands-On Software Engineering with Golang_](/notes/hands-on-software-engineering-with-golang/) book used the [`gocheck`](https://labix.org/gocheck) package in order to scaffold and run tests on the concrete implementations. You could also use something like [`testify`](https://github.com/stretchr/testify), however I would prefer `gocheck` since it is [integrated into GoLand](https://www.jetbrains.com/help/go/testing.html#packages-used-for-testing) by default.

However, if all you need is to simply gather tests into a suite, this technique works just fine. More importantly, it uses only the standard library to achieve this, as outlined in Google's [best practices for tests](https://google.github.io/styleguide/go/best-practices#tests).
