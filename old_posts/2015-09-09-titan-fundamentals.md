---
id: 215
title: Titan Fundamentals
author: rahular
layout: post
excerpt: A basic introduction to the Titan Graph DB with starter code in Java. This post is more of a aggregation of useful pointers I found scattered in the web.
guid: http://rahular.com/?p=215
permalink: /titan-fundamentals/
categories:
  - Database
  - Graph
  - Titan
---
I have been trying to learn the [Titan](https://github.com/thinkaurelius/titan/) graph DB for a few weeks now. The documentation, though comprehensive, is scattered in GitHub and on their site. Since it is in active development, the versions keep changing and the documentation for the older versions sometimes become obsolete.

You can find the documentation of their latest release [here](http://s3.thinkaurelius.com/docs/titan/0.9.0-M2/)

After a few nights of sleepless research, I finally figured out how to use Titan in Java. So here is how you can quickly start writing programs with Titan as your DB.

> **Note**: I am using Eclipse Luna on Ubuntu 14.04.

 1. Download Titan. [Link](https://github.com/thinkaurelius/titan/wiki/Downloads)
 2. Extract it anywhere. You will find a folder named `lib` which will contain all the dependency JARs. Make note of it's path.
 3. Create a project in Eclipse and go to `properties -> Java Build Path`
 4. Click on `Add library -> User library -> User libraries`
 5. Click on `New` and give the library a name. I gave `titan-java`
 6. An empty `User library` should have been created if everything went well
 7. Click on `Add external JARs` and navigate to the `lib` folder (we made note of this in Step 2)
 8.  Add all the JARs and finish.

Now everything is set and we are good to go. The API is pretty straight-forward and since Titan adheres to [Tinkerpop's Blueprints API](https://github.com/tinkerpop/blueprints/wiki), the documentation is available readily all over the web.

Below you can see the starter code I wrote which will:

 - open the graph
 - add some data if it is empty
 - print the names of the vertices
 - close the graph

> **Note**: You will have to create a folder named `db` in the project's root folder. This is where Titan creates the database.
> 
> The code uses Berkeley DB as the index backend. You can change it with anything you want. Titan supports Cassandra, HBase, etc.

```
import com.thinkaurelius.titan.core.TitanFactory;
import com.thinkaurelius.titan.core.TitanGraph;
import com.tinkerpop.blueprints.Edge;
import com.tinkerpop.blueprints.Vertex;

public class Titan {

	private TitanGraph graph;
	private boolean isGraphOpen;
	
	public Titan() {
	}

	public void openGraph() {
		if(isGraphOpen) {
			System.out.println("Graph is already open.");
			return;
		}
		graph = TitanFactory.build()
				.set("storage.backend", "berkeleyje")
				.set("storage.directory", "./db").open();
		isGraphOpen = true;
	}
	
	public void addData() {
		Vertex a = graph.addVertex(null);
		Vertex b = graph.addVertex(null);
		a.setProperty("name","John");
		b.setProperty("name","Jane");
		
		Edge e = graph.addEdge(null, a, b, "knows");
		e.setProperty("since", 2006);
		
		graph.commit();
	}
	
	public void closeGraph() {
		if(!isGraphOpen) {
			System.out.println("Graph is already closed.");
			return;
		}
		graph.shutdown();
		isGraphOpen = false;
	}

	public static void main(String[] args) {
		Titan titan = new Titan();
		titan.openGraph();
		
		int count = 0;
		for(Vertex vertex: titan.graph.getVertices()) {
			System.out.println(vertex.getProperty("name"));
			count++;
		}
		
		if(count == 0) {
			System.out.println("The graph is empty.");
			System.out.println("Adding some data..");
			titan.addData();
			System.out.println("Data added. Run again to see changes.");
		}
		
		titan.closeGraph();
	}
}
```
Hope you found this helpful. That's all folks!
