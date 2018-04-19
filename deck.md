footer: Instrumenting your way to success :: Chris Johnson :: @johnsonch
autoscale: true

# Instrumenting your way to success

---
## About Me

![fit right](http://www.johnsonch.com/images/me.jpg)

* Husband and Father
* Operations Manager @ healthfinch
* Owner at JohnsonCH, LLC
* @johnsonch => most places on the internet

---
# Disclaimers/Ground Rules

* These are **my** experiences and opinions, your mileage may vary
* I'm not here to argue, if you want to do that buy me a beer afterwards
* Ask questions, but again I'm not looking for a fight

![full](images/pexels-photo-209954.jpeg)

---
# What are we going to learn about?
* Why I keep saying "Instrumenting"
* What is "Instrumenting"
* What to "Instrument"
* Where to "Instrument"
* Why you should "Instrument"

---
# The TL;DR; is intentional structured logging

---
# Why is it important?

* You will look smarter when you can quickly
  * diagnose issues
  * validate features
* Being intentional in your logging helps stick to providing business value

^ How are we going to be better after we learn this?
    - We're going to have an idea going forward about how to not only
    think about error logging but success logging, metric logging and
    organizing those logs into a consistant parsable form
  What aren't we going to learn about?
    - How to analyze logs and aggregate data, there just isn't enough time
  What should we take home from this session?
    - A passion for tracking your applications metrics

---
# Why I keep saying "Instrumenting"
## Words are hard

^ The air quotes are because I hate so many terms in software development,
but lack the passion to come up with better ways to describe them. And If
I would have called this a structured logging session, would you have come?

---
# What is instrumenting

> instrumentation refers to an ability to monitor or measure the level of a product's performance, to diagnose errors and to write trace information
-- wikipedia

^ Boil this down to it's an intentional act to make sure you know what your app is doing both good and bad

---
# What should we instrument?
 * Legal info
 * Benchmarks
 * User metrics
 * Errors
 * Anything that would be useful

---
# How do we get from print statements to structured logs?

---
# Slowly and Deliberately

---
# Raw print statements inter mingled

```ruby
def foo(bar, baz, boo)
  local_var = some_other_method(bar, baz)
  if local_var == boo
    puts 'This is some event that happened in some class'
    return true
  else
    another_method(boo, bar)
  end
end
```

^ While this is a way to debug things in development it really doesn't help you while the app is running in production. It requires you to search for the full string in your code base and also requires looking at the code to figure out what happened. It doesn't even give you any information about the method

---
# Structured inter mingled

```ruby
def foo(bar, baz, boo)
  local_var = some_other_method(bar, baz)
  if local_var == boo
    puts "{'event': 'MyClass.foo.success',
           'params': '[#{bar},#{baz},#{boo}]',
           'message': 'success call of some_other_method'}"
    return true
  else
    another_method(boo, bar)
  end
end
```

---
# Structured inter mingled - custom method

```ruby
class Instrumentor
 def success(event, params, message)

 end
end

class MyClass
  def foo(bar, baz, boo)
    local_var = some_other_method(bar, baz)
    if local_var == boo
      puts "{'event': 'MyClass.foo.success',
             'params': '[#{bar},#{baz},#{boo}]'}"
      return true
    else
      another_method(boo, bar)
    end
  end
end
```

---
# Separate log file

---
# Wrap up
