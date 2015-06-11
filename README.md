# opal-record
active record data sync to browser

in browser can use scopes

Jobs.active
or
ProductionCenter.by_country_code("US")

or

Jobs.find
Jobs.find_by_friendly_id

etc

That's it... no other features

when you do a scope you will get back an array of models 
when you do a find you will get be an individual model

has_xxx relationships result in array of (or a single) id.


simpliest implementation:

scopes are just sent as messages to server which sends back a bunch of json, that gets converted back to a set of Opal
"models"...
{scope: [id1, id2 ... idn], data: {jobs => [{id => 1, name => "foo" ... }, ], 

find returns the same thing except {found: id1, data: ...}

optimizations

browser has a cache of these objects.  so Jobs.find(123) will just return that job if it already exists in browser.  
else it asks the server

lets say we ask server for today's orders.  `Orders.scoped_by_date(from: today.start_of_day, to: today.end_of_day)`

brute force we expect and array of orders, each order has an array of jobs (has_many), each job has things like paper,
finishing_option etc.  

So lets keep track of what we already sent to this browser, and don't send it again.

If we are wrong, browser will ask for it.

So in the data: portion we may return nothing i.e. data: {} meaning server thinks browser has everything.

lets say server is wrong... fine.  as browser reconstructs the data into objects it will ask (using find) for some
record (say Job.find(123).)   Okay great if server thinks browser allready has 123, then it knows something has happened and
clears its (servers) cache of stuff sent to this browser.  

Server uses the cache to record {table_name: ..., record_id:  ..., synced_with: ... synced_at ... }

def find(table_name, record_id, sync_with, synced_at)
  # returns nil if cache we aready did this (i.e. use your own copy) i.e. all params match something in db
  # and synced_at >= last_update of that id
  # else returns the record as a hash, and updates the cache, using current time as synced_with
end

Need a way to clear the cache?  

  
  



