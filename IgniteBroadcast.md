###Ignite Broadcast Java example   
I will show scala example sometime later..

* Download and install ignite   
* This example will run on Java 7 and above
* start the ignite nodes
* broadcast a message to ignite nodes   

Download apache-ignite-hadoop-1.5.0.final-bin
Start ignite node 1 with following command
>./bin/ignite.sh ../apache-ignite-1.5.0.final-src/examples/config/example-ignite.xml

Start ignite node 2 with same command
>./bin/ignite.sh ../apache-ignite-1.5.0.final-src/examples/config/example-ignite.xml

```java
package org.apache.ignite.zeppelin;

import java.util.Collection;

import org.apache.ignite.Ignite;
import org.apache.ignite.IgniteCache;
import org.apache.ignite.Ignition;
import org.apache.ignite.lang.IgniteCallable;
import org.apache.ignite.resources.IgniteInstanceResource;

public class IgniteBroadcastTest {

	public static void main(String[] args){
		
	    try (Ignite ignite = Ignition.start("example-ignite.xml")) {
	        // Put values in cache.
	       final  IgniteCache<Integer, String> cache = ignite.getOrCreateCache("myCache");
	         
	        cache.put(1, "Hello");
	        cache.put(2, "World!");
	   

	        // Print out hello message on all nodes.
	        ignite.compute().broadcast(
	            new IgniteRunnable() {
	                @Override public void run() {
	                    System.out.println();
	                    System.out.println(">>> Hello Node! :) - " + cache.get(1));
	                }
	            }
	        );
	        
	        Collection<String> res = ignite.compute().broadcast(

	        		new IgniteCallable<String>() {
	                    // Automatically inject ignite instance.
	                    @IgniteInstanceResource
	                    private Ignite ignite;

	                    @Override public String call() {
	                        System.out.println(cache.get(1));
	                        System.out.println("Executing task on node: " + ignite.cluster().localNode().id());

	                        return "Node ID: " + ignite.cluster().localNode().id() + "\n" +
	                            "OS: " + System.getProperty("os.name") + " " + System.getProperty("os.version") + " " +
	                            System.getProperty("os.arch") + "\n" +
	                            "User: " + System.getProperty("user.name") + "\n" +
	                            "JRE: " + System.getProperty("java.runtime.name") + " " +
	                            System.getProperty("java.runtime.version");
	                    }
	            });

	        // Print result.
	        System.out.println();
	        System.out.println("Nodes system information:");
	        System.out.println();

	        for (String r : res) {
	            System.out.println(r);
	            System.out.println();
	        }
	        
	}
}

}


```


