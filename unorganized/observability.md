# Principles

## USE

For keeping track of the performance of machines / servers
What does USE stand for? : Utilization, Saturation, Errors 🧠 #observability

### *U*tilization && *S*aturation

* CPU Utilization, Network saturation
* Queue Length for jobs

### *E*rrors

Errors happening / time

## RED

For keeping track of the satisfaction of users
What does RED stand for? 🧠 #observability

Rate, Errors, Duration

### *R*ate

The flow of requests entering the system. So like the spigot of traffic

### *D*uration

The time it takes to handle each of those requests

### *E*rrors


RED is best for monitoring systems that have a lot of inputs. Messaging systems like ours basically are built around the idea of queues, and processing the queues faster than the requests can come in.
- - -

## The 4 Golden Signals

* Latency - How long it takes to serve a request
* Traffic - How many requests are flowing into the system
* Errors - Rate of requests that are failing
* Saturation - How "full" the system is

# Ideas

* Dashboards can be by context. (like a row for each)
* Then show the length of the queues for each process. Or perhaps if we have a process for each symbol, we just sum them up.

```
Policies:

Queue Lengths max over the last minute, show the last 3 hrs: Queues named: Policies*
DB queries: It's a different logical DB, so it should be possible to get separate data for just this db, even if it's just 1 server
  * Queries Per Second
  * Latency of queries

Trading:

Queue Lengths max over the last minute, show the last 3 hrs: Queues named: sum(TradingLiabilities*)
DB queries: It's a different logical DB, so it should be possible to get separate data for just this db, even if it's just 1 server
  * Queries Per Second
  * Latency of queries

Insurance:
Queue Lengths max over the last minute, show the last 3 hrs: Queues named: Insurance*
DB queries: It's a different logical DB, so it should be possible to get separate data for just this db, even if it's just 1 server
  * Queries Per Second
  * Latency of queries
```

## Hierarchical

If you start with broad data, you can then show more detailed info below it.



# Unorganized

## Variables

* There are template variables that can be used to templatize inputs (node 1, node 2), but also datasources completely.
* My idea above - could this be a standard template, that coudl be parameterized with just the prefix of the queue names?
* https://grafana.com/docs/grafana/latest/dashboards/variables/#examples-of-templates-and-variables

## Gerneral Best Practices

* Compare like to like
  * If the magnitude of the numbers is too different, the data won't be relatable.
  * Normalize (%'s), or separate
* Use Meaningful Colors. Blue for good, red for bad.
* "Browsing" dashboards is bad. You want to be able to give users a navigation flow that will match their goals somehow. see (Dashboard Links)[#dashboard-links]

# Dashboard Links

You can link from 1 dashboard to another in several ways:

## Dashboard Links

You can use template variables and time ranges in links between dashboards. So if you're using consistent template variables, they can carry over: ex: https://play.grafana.org/d/rUpVRdamz/dashboard-links-with-variables?orgId=1

tags are used to auto-select the dashboards you want to link to.

## Panel Links

If the link is relevant to the specific panel you're looking at, you can add panel links. These are not quite as flexible. You seem to have to specify the url more directly. But you can still include template variables.


## Panel Links

## Data Links


