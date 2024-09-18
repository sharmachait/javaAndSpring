Concurrency is achieved with Time Slicing algorithm to give each task a fair chance to run
![[Pasted image 20240911092725.png]]
![[Pasted image 20240908124523.png]]
this examples starts and stops the other thread immediately
## to run some code in another thread we have 3 options
#### extending Thread class
![[Pasted image 20240908124708.png]]
#### implementing runnable interface
better way
![[Pasted image 20240908130121.png]]
#### lambda expression best way

![[Pasted image 20240908130228.png]]
==do this for master and slave threads when starting as slave==

# the .start() is non blocking

# if we want the calling thread to be blocked at some point by a thread do thread.join()

## accessing the current thread
![[Pasted image 20240908130348.png]]
this gets us the current thread that is running the runnable
if we do it out in the main method we get the main thread
we can give names to thread at the time of creation in the constructor

## simple thread pool with 3 threads and queue capacity of 10 better to just use executor service
![[Pasted image 20240908154609.png]]