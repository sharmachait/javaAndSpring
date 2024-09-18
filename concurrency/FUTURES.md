when we create threads we are actually creating platform threads and they are an expensive resource

solution to this memory and time overhead is using a thread pool
create a set number of threads before hand and use them over and over again

so instead of creating a new thread we submit a task to the thread pool

## Task
1. Runnable
2. Callable

we submit these tasks to the executor service that handle these tasks and return the result when completed 

![[Pasted image 20240908131220.png]]

Runnable doesnt return anything and doesnt throw any exception

Callable has a return type and can also throw exception

![[Pasted image 20240908131318.png]]
 we can use the returned future object to inspect the state of the task, the result or the exception thrown
 ![[Pasted image 20240908143429.png]]

# Executor Service

```java
public static void main(String[] args){
	ExecutorService executorService = Executors.newFixedThreadPool(10);
	executorService.execute(newRunnable("task1"));
	executorService.shutdown();
}

public static Runnable newRunnable(String msg){
	return new Runnable(){
		public void run(){
			Sout(msg);
		}	
	};
}
```

two built in implementations of the executorSerivce
1. ThreadPoolExecutor
![[Pasted image 20240909094707.png]]

the extra idle threads are allowed to stay alive only for keepalivetime
2. ScheduledThreadPoolExecutor
this allows us to schedule tasks to run at some point

![[Pasted image 20240909094505.png]]

when working with a thread pool we can execute Runnables
and submit runnable and callables
the Submit method returns a future

```java
public static void main(String[] args){
	ExecutorService executorService = Executors.newFixedThreadPool(10);
	Future f = executorService.submit(newRunnable("task1"));
	while(!f.isDone()){
		sout(false);
	}

	try{
		//future . get is blocking
		f.get();//runnable will return null, callable can return a value
	}
	catch(Exception e){
		sout(e.message);
	}

	executorService.shutdown();
}

public static Runnable newRunnable(String msg){
	return new Runnable(){
		public void run(){
			Sout(msg);
		}	
	};
}
```

```java
new Callable<Integer>(){  
	@Override
    public Integer call(){  
        return 1;  
    }  
};
```

in case of the callable the f.get() will return what ever the call method is returning in the Callable

# running or calling multiple callables/runnables concurrently 

invokeAny and invokeAll

```java

ExecutorService executorService = Executors.new FixedThreadPool(3);

List<Callable<String>> callables  =new ArrayList<>();

callables.add(new Callable<String>(){
	@override
	public string call(){
		return "hi";
	}
})

callables.add(new Callable<String>(){
	@override
	public string call(){
		return "bye";
	}
})

try{
	String res = (String) executorService.invokeAny((Collection) callables);
	
	List<Future<String>> ress = 
	executorService.invokeAll((Collection<Callable<String>>) callables);
	for(Future future : ress){
		sout(future.get());
	}
}
catch(Exception e){
	sout(e.message);
}
finally{
	executorService.shutdown();
	executorService.shutdownNow();
}
```

invokeAny returns the result from the task that finishes first, rest other tasks are cancelled

invokeAll returns a list of futures

the shutdown method waits for all the tasks that have been submitted to be completed

to shutdown immediately and not wait call the shutdownNow() method instead

invokeAny would cancel other tasks what if i dont want them to be cancelled i just want to process one by one as they are completed
# best way to process multiple results 
![[Pasted image 20240909122135.png]]the take method return the first future that was completed