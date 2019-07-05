# Memorb

Memoize instance methods more succinctly.

## Why memoize instance methods?

Sometimes you want to execute an instance method and have its result cached for future calls. You may want this because the method:

- is computationally expensive and caching its result would increase application performance
- returns a newly instantiated object which shouldn't be recreated on subsequent calls
- makes calls to an external service which should be limited for performance, API rate limiting, etc.

## What's the problem?

Below is a simple, contrived `Rectangle` class that will be used for demonstration. The methods represent computationally expensive methods that can't be calculated before instantiation, but whose results won't change for the lifetime of the instance and can therefore be cached.

```ruby
class Rectangle

  def initialize(width:, height:)
    @width = width
    @height = height
  end

  attr_reader :width, :height

  def area
    width * height
  end

  def perimeter
    2 * (width + height)
  end

  def square?
    width == height
  end

  def memory_address
    (object_id << 1).to_s(16)
  end

end
```

A common way of accomplishing memoization in most cases is to memoize the result in an instance variable:

```ruby
def square?
  @square ||= width == height
end
```

But this simplistic approach has a few problems:

- if the result is falsey, the cached value is bypassed and the computation re-executed on subsequent calls
- method calls aren't distinguished by their arguments
- concurrent calls to the method could result in more than multiple executions when that may not be desirable (race condition between checking instance variable and running the computation)
- having many methods saved in instance variables could make inspection of the instance a harder to read
- if the chosen variable name is long, it could cause line wrapping when that would otherwise be unnecessary
- the instance variable name is often chosen to match the name of the method, but method name punctuation can make this impossible

The falsey result problem could be solved by checking if the instance variable is `defined?` instead:

```ruby
def square?
  defined?(@square) ? @square : @square = width == height
end
```

But this approach gets a bit repetitive with the instance variable, is harder to read, and doesn't resolve any of the other problems.

There must be a better way...

## The Solution

Memorb solves all of these problems and more! At a minimum, just specify which methods should be cached and you're done:

```ruby
include Memorb[:area, :perimeter, :square?, :memory_address]
```

Each registered method will execute once and have its result cached to be returned immediately on every call thereafter.

## Advisories

### Registering methods that aren't (yet) defined

Memorb will allow you to register a method to be memoized before that method is actually defined. In fact, this is the normal behavior when specifying method registrations as part of the Memorb module inclusion. However, since registration of a method adds it to the prepended mixin for the class, `respond_to?` will return true on all instances, even if the method never actually gets defined on them.

**There are plans to change automatic method overriding upon registration which would resolve this.**
