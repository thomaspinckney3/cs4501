
##Cause

### Speed
* people's demand is elastic
* user's 'perception: slow-- > use less  fast---> efficient 
* network, database, generate dynamic page, traffic, calculation on the back end

### Database
* wait for locks for consistency 
1. shared resource queue single thread 
2. cons of locking per column, per row, per field 
* size




### Network
* drop packets, speed of light, the slow start of TCP, mobile is slower 
* getting base page --> getting images, css
1.small number of concurrent connections each time 
2.browser has limited number of connections set up
* HTTP: TCP connections set up  vs HTTPS: SSL security set up 
HTTP is easy to set up, less objects(?)



--------------------------------------------

##Solution 


### Database
* indexing(sorted in several different ways)
1. read from disk is million times slower 
2. small data is faster than bigger data 
3. big updates breaks up to small updates.  not hold several highly contested pieces at same time



### How to make pages load faster
* image size
* lazy loading (only loads the thing is visible) 
* Design for speed (UI)
* HTTPs generally used everywhere for good practice, only used in pages have form 
* gzip (the content sent to browser has been zipped)
* future: HTTP 2.0
* send the unimportant requests to the the local data center, important requests to the master data center 
* content-distribution network: they don't save HTML, but other static stuff only copy the really popular stuff
* cached the unsaved content for the certain amount of time 

### How the cache data get updated 
* new image is created, not update the old one, it is still there, change the HTML page reference. 
* purge the old data in cache some time later
* let the cache lived for a short amount of time 
* validation: when DB is changed, shut down the cache ( delete the file in cache has been changed)
* backwards of cache: once the machine is reboot, cache is gone
