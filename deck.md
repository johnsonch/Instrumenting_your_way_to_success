footer: Instrumenting your way to success :: Chris Johnson :: @johnsonch
build-lists: true
autoscale: true

# Instrumenting your way to success


![full](images/pexels-photo-713126.jpeg)

^ Photo by Eneida Nieves from Pexels

---
## About Me

![fit right](http://www.johnsonch.com/images/me.jpg)

* Husband and Father
* Operations Manager @ healthfinch
* Owner at JohnsonCH, LLC
* @johnsonch => most places on the internet

---
# healthfinch is hiring!

Come talk to me!

![fit right](images/Charlie_Flag-01-01.png)


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

![full](images/pexels-photo-356065.jpeg)

---
# The TL;DR;

### intentional structured logging

![full](https://images.pexels.com/photos/261909/pexels-photo-261909.jpeg)

^ Spoiler!

---
# Why is it important?

* You will look smarter when:
  * you can quickly diagnose issues
  * validate features
* Being intentional helps stick to providing business value

![full](https://images.pexels.com/photos/356079/pexels-photo-356079.jpeg)

---
# After today we'll

* Have some ideas and some patterns to take back home with you
* A passion for tracking metrics about your applications

^ How are we going to be better after we learn this?
    - We're going to have an idea going forward about how to not only
    think about error logging but success logging, metric logging and
    organizing those logs into a consistant parsable form
  What should we take home from this session?
    - A passion for tracking your applications metrics

![full](https://images.pexels.com/photos/39644/robonaut-machines-dexterous-humanoid-39644.jpeg)

---
# What you're not going to learn
* How to analyze your collected data
* Specifics of instrumenting for your language/framework of choice

^  What aren't we going to learn about?
    - How to analyze logs and aggregate data, there just isn't enough time

![full](https://images.pexels.com/photos/259027/pexels-photo-259027.jpeg)

---
# Why I keep saying "Instrumenting"
![full](https://images.pexels.com/photos/695571/pexels-photo-695571.jpeg)

---
# Words are hard

![full](https://images.pexels.com/photos/695571/pexels-photo-695571.jpeg)

^ The air quotes are because I hate so many terms in software development,
but lack the passion to come up with better ways to describe them. And If
I would have called this a structured logging session, would you have come?

---
# What is instrumenting

> instrumentation refers to an ability to monitor or measure the level of a product's performance, to diagnose errors and to write trace information
-- wikipedia

^ Boil this down to it's an intentional act to make sure you know what your app is doing both good and bad

![full](https://images.pexels.com/photos/951241/pexels-photo-951241.jpeg)

---
# What should we instrument?
 * Legal info
 * Benchmarks
 * User metrics
 * Errors
 * Anything that would be useful

^ Really there is no limit to the amount of data to capture from your application
the only caution is if your app is spending too much time processing event data
it may slow things down

![full](https://images.pexels.com/photos/241544/pexels-photo-241544.jpeg)


---
# How do we get from print statements to structured logs?

![full](https://images.pexels.com/photos/297755/pexels-photo-297755.jpeg)

---
# Slowly and Deliberately

![full](https://images.pexels.com/photos/320956/pexels-photo-320956.jpeg)

---
# Raw print statements intermingled

```ruby
def self.import(file)
  CSV.foreach(file.path, headers: true) do |row|
    parcel_hash = row.to_hash
    ...
    save_time = Benchmark.measure { parcel.save }
    puts "Save Time: #{save_time}"
  end
end
```

---
# Raw print statements intermingled

```bash
   (0.0ms)  begin transaction
  SQL (0.6ms)  INSERT INTO "parcels" ("address", "current_year_value", ...
   (2.6ms)  commit transaction
Save Time:   0.010000   0.000000   0.010000 (  0.316892)
```

^ While this is a way to debug things in development it really doesn't help you while the app is running in production. It requires you to search for the full string in your code base and also requires looking at the code to figure out what happened. It doesn't even give you any information about the method

^ Code: https://github.com/johnsonchmatc/madison_properties/tree/tccc22_a

---
# Structured intermingled

```ruby
def self.import(file)
  CSV.foreach(file.path, headers: true) do |row|
    parcel_hash = row.to_hash
    ...
    save_time = Benchmark.measure { parcel.save }
    log = { event: 'parcel.import.save_time', message: save_time }
    puts log.to_json
  end
end
```

^ Code: https://github.com/johnsonchmatc/madison_properties/tree/tccc22_b

---
# Structured intermingled

```bash
   (0.1ms)  begin transaction
  SQL (0.6ms)  INSERT INTO "parcels" ("address", "current_year_value", "previous_year_value", ....
   (3.0ms)  commit transaction
{"event":"parcel.import.save_time","message":{"label":"",
"real":0.40027463901787996,"cstime":0.0,"cutime":0.0,"stime":0.0,
"utime":0.009999999999999787,"total":0.009999999999999787}}
```

---
# Structured - separate file

```ruby
class BenchmarkLogger < Logger
  def format_message(_severity, _timestamp, _progname, msg)
    "#{msg}\n"
  end
end

logfile = File.open("#{Rails.root}/log/benchmark.log", 'a') # create log file
logfile.sync = true # automatically flushes data to file
BENCHMARKLOGGER = BenchmarkLogger.new(logfile) # constant accessible anywhere
```

^ Code: https://github.com/johnsonchmatc/madison_properties/tree/tccc22_C

---
# Structured - separate file

```ruby
  def self.import(file)
    CSV.foreach(file.path, headers: true) do |row|
      parcel_hash = row.to_hash
      ...
      save_time = Benchmark.measure { parcel.save }
      message = { event: 'parcel.import.save_time', message: save_time }
      BENCHMARKLOGGER.info message.to_json
    end
  end
```

^ Code: https://github.com/johnsonchmatc/madison_properties/tree/tccc22_c

---
# Structured - separate file

```bash
{"event":"parcel.import.save_time",
"message":{"label":"",
"real":0.0930731890257448,
"cstime":0.0,
"cutime":0.0,
"stime":0.010000000000000009,
"utime":0.010000000000000231,
"total":0.02000000000000024}}
```

---
# Pretty cool!

---

![full](http://randyscarpetcare.com/communities/6/000/002/006/766/images/15086235.jpg)

---
# Case Study

![fit inline](http://homevestors.com/wp-content/uploads/ForScreen_RGB_ZillowLogo_White-on-Blue.png)

^ https://www.zillow.com/engineering/debugging-production-event-logging/

---
### Case Study :: Zillow
- Inconsistent search results
- Deemed to be a race condition that they couldn't reproduce locally

^ https://www.zillow.com/engineering/debugging-production-event-logging/

![full](images/IMG_6005.jpg)

---
### Case Study :: Zillow

```ruby
if self.class == Apartment
…
::NewRelic::Agent.record_custom_event(
'ApartmentIndex',
event_params
)
end
```

> [We] begin registering searchable event data every time an apartment record was indexed to ElasticSearch.

^ https://www.zillow.com/engineering/debugging-production-event-logging/

---
### Case Study :: Zillow

> lean into your logging services to diagnose difficult production problems, like race conditions

^ https://www.zillow.com/engineering/debugging-production-event-logging/

![full](images/20170806-IMG_8213.jpg)

---
# What next?

![fit inline](https://i.imgflip.com/28t0vs.jpg)

---
# Thank You

Chris Johnson
email: chris@johnsonch.com / cj@healthfinch.com
twitter: @johnsonch
slides: https://github.com/johnsonch/Instrumenting_your_way_to_success/blob/master/deck.md

---
# healthfinch is hiring!

Come talk to me!

![fit right](images/Charlie_Flag-01-01.png)

