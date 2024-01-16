
# Building Graphs - Part 1

Imagine we've been hired to work on an application that keeps track of friendships between people. This kind of application has many use cases - after all, this is the foundation of many social networks. Remember that a graph is just another way of depicting a network. So when we think about portraying a social network, we could also call it a social graph. So let's get to work! In our application, we should be able to do the following:

    Add people (nodes) to the social network. To keep it simple, they will only have a name property.
    Add friendships between people (edges between nodes). These will be unweighted and undirected.
    Remove nodes from the social network. Some people may want to revoke their accounts.
    Remove edges from the social network. Sadly, some people may want to end their friendships.

This graph will not be a connected graph. A connected graph is one where a potential path exists between every single node in the graph. However, in an actual social network, we can't assume that everyone knows each other. Our social network could look like this:

In the image above, Ada, Grace and Eve are friends. However, no one in this group is friends (yet) with Mary and Lina, who have their own friend group.

    An algorithm for checking the reachability between two nodes. For instance, we can reach Ada via Eve or Grace but not via Mary or Lina. In a social network trying to bring different groups together, knowing the reachability between two nodes could be very useful. For instance, in a social network working to mobilize voters, it might be useful to see if various people in a city are connected or not. If not, introducing them could allow them to work together and better mobilize. We will walk through the process of writing this algorithm.
    An algorithm checking the shortest path between two nodes. In the chart above, the shortest path between all reachable nodes is just one edge but a more complex social network would have paths of varying lengths. In this way, we can check the degrees of separation between two people in the network. This could have many uses in a real-world application as well. For instance, in a professional network like LinkedIn, it might be useful to see your closest connection at a place you hope to work - and how you are connected. Or you might want to design a friend finder that automatically recommends connecting with friends of friends - which, in the parlance of graphs, would be all adjacent nodes of a node. We won't walk through this method - instead, you will have the opportunity to use one of the algorithms we discuss in order to solve this problem on your own.

You can probably imagine all kinds of ways to build this application out further. Imagine, for instance, that we decided to weight edges based on the number of social interactions between two nodes. We could then determine their overall connectivity (and perhaps even make assumptions about how close they are as friends). And you can likely also imagine all kinds of other use cases for graphs besides social networks ranging from transportation networks to HR applications to business applications tracking international trade.

In the next lesson, we'll actually start building our application.



# Building Graphs - Part 2

We are going to keep this application very simple. In fact, our nodes will just be strings. That way, we can focus entirely on building a Graph class without worrying about a Node class.

So let's think about the simplest possible behavior our Graph class should have. We should be able to instantiate a graph with an empty data structure where we will store information about nodes and edges. So which data structure should we use?

Remember, we have three options:

    Edge list
    Adjacency list
    Adjacency matrix

We aren't going to do an edge list because honestly, it's not that great for searching. We'd have to look through the entire collection of pairs just to see a list of someone's friends. That's not very efficient.

That leaves an adjacency list or an adjacency matrix. We are going to use an adjacency list for a number of reasons. First, we can store additional data in an adjacency list (such as information about nodes and edges). We won't do that here, but it's an advantage that an adjacency list has over an adjacency matrix.

An adjacency list is also more efficient than an adjacency matrix for checking a node's adjacent nodes - and in the context of this application, that means it's more efficient for looking at a list of someone's friends. That is a pretty important operation. Why is it more efficient? Well, finding adjacent nodes is an O(N) search for both adjacency lists and adjacency matrices. However, with an adjacency list, we just need to iterate over a list of adjacent nodes (in this case, a list of someone's friends). In the case of an adjacency matrix, we'd need to iterate over all nodes - that is, everyone in the application, and check to see if each person is a friend or not. So imagine if our application has a million users. We want to see a list of Jasmine's friends. (She has fifty friends in the application in all.) It makes much more sense to iterate through a list of those fifty friends to see them than to iterate through all one million users, checking to see if each is a friend or not.

Now that we've decided to use an adjacency list, we know that when we instantiate a Graph, it should come with an empty adjacency list.

We are going to use a Map to store this list. While we could keep things very simple and just use a JavaScript object, there are performance advantages to using a Map - and we can easily use a Map's built-in methods to add nodes and edges to our adjacency list. What are the performance advantages? Well, for one thing, a Map uses a hash table lookup algorithm under the hood. The ES6 specs guarantee at least an average of sub-linear complexity for Map implementations. But what does that mean? Well, sub-linear complexity means that the Big O needs to be better than linear time (O(N)), so that could be something like O(1) constant time or O(log n) logarithmic time.

Meanwhile, if we used an object instead, we'd need to iterate through the object - and object iteration isn't optimized already, which means we'd have a Big O of O(N). So iterating through objects is slower.



## Instantiating a Graph

Our first check will make sure that we can correctly instantiate a new graph with an empty adjacency list:

__tests__/graph.test.js

import Graph from '../src/graph.js';

describe('Graph', () => {

  let graph = new Graph();

  afterEach(() => {
    graph = new Graph();
  });

  test('should correctly instantiate a graph', () => {
    expect(graph.adjacencyList.size).toEqual(0);
  });
});

Note that we start by instantiating a new graph and also add an afterEach() block to reset the graph after each test. It's overkill for just one test but it will DRY up future tests.

In the test itself, we expect(graph.adjacencyList.size).toEqual(0). Maps have a size property, which means we can just check to see if our graph's adjacencyList has a size property that's equal to zero (an empty map).

Here's the code to get this passing:

src/graph.js

export default class Graph {
  constructor() {
    this.adjacencyList = new Map();
  }
}

## Creating Nodes

What's the next thing we need to do? Well, we need to be able to add people to our social network. People are the equivalent of nodes in a graph. We'll keep our methods sufficiently abstract so they pertain more to creating graphs in general than building social networks more specifically. For that reason, we'll add a Graph.prototype.addNode() method, not a Graph.prototype.addFriend() method.

Here's our new test:

__tests__/graph.test.js

...
  test('should add a new node', () => {
    graph.addNode("Jasmine");
    expect(graph.adjacencyList.get("Jasmine").size).toEqual(0);
  });
...

We should be able to add a node via a Graph.prototype.addNode() method. We don't have a Graph.prototype.getNode() method yet so we can't use that for our expectations. For that reason, we'll just access the node via the graph's adjacencyList. Since the adjacencyList is a Map, we can use Map.prototype.get() to get the value associated with the key. Why are we looking for a size property again once we get the value related to Jasmine? Well, each key in our adjacency list will be a Set.

A Set is a collection of entirely unique values. Jasmine and Ada can be friends only once - not three times - so we want to make sure that a node can occur in the collection only once. That's where using a JavaScript Set comes in handy.

Once again, the ES6 specifications guarantee that Set implementations generally have sub-linear complexity - so Sets are also pretty efficient and much better than using a collection like an array.

Here's the code we need to pass our test:

src/graph.js

export default class Graph {
  ...
  addNode(name) {
    this.adjacencyList.set(name, new Set());
  }
}

As we can see, the implementation is simple. We use Map.prototype.set() to add a key-value pair to our adjacencyList. The key will be the name of the node we are adding and the value will be an empty Set.

## Checking to See if Nodes Exist

What's the next behavior we should implement? Well, we need to know whether a node exists in our graph or not. That means the next step is to add a Graph.prototype.hasNode() method. The simplest behavior for this method will be to return false if the node doesn't exist in the adjacency list.

Here's a test:

__tests__/graph.test.js

...
  test('should return false if the node does not exist in the adjacency list', () => {
    expect(graph.hasNode("Ada")).toEqual(false);
  });
...

And here's the minimum code in a Graph.prototype.getNode() method to get this test passing:

src/graph.js

...
  hasNode(name) {
    return false;
  }
...

Now we need to write a test for when a node does exist in a graph:

__tests__/graph.test.js

...
  test('check to see if node exists in graph', () => {
    graph.addNode("Jasmine");
    expect(graph.hasNode("Jasmine")).toEqual(true);
  });
...

To get this test passing, we can use a method that a Map's prototype offers. Here's the update to our method:

src/graph.js

...
  hasNode(name) {
    if (this.adjacencyList.get(name)) {
      return true;
    }
    return false;
  }
...

We take advantage of Map.prototype.get(), which either returns a value of a key in a map or returns undefined if the key doesn't exist. Here's a situation where our code is already more efficient because we are using maps instead of just iterating through a basic object. To check if our graph has a node, we need to do a search. Because we are using Map.prototype.get() instead of looping through an array or object, calling this method has sub-linear complexity instead of linear complexity (O(N)).

Now that we've added all the basic functionality for nodes other than removing nodes, which we'll get to later, we're ready to start adding functionality for edges.

## Adding Edges

At this point, we have methods to add nodes to a graph and to see whether a node exists in our graph. The next step is to add functionality to add relationships between nodes.

In other words, we need to be able to add edges, which means we need a Graph.prototype.createEdge() method.

As always, we'll start with a test:

__tests__/graph.test.js

...
  test('add an edge between two nodes', () => {
    graph.addNode("Jasmine");
    graph.addNode("Ada");
    graph.createEdge("Jasmine", "Ada");
    expect(graph.adjacencyList.get("Jasmine").has("Ada")).toEqual(true);
    expect(graph.adjacencyList.get("Ada").has("Jasmine")).toEqual(true);
  });
...

First, we need to add two nodes to our test. Then we need to create an edge between these two nodes with a Graph.prototype.createEdge() method. Note that this test has two expectations. Remember that this is an undirected graph - a friendship goes two ways, not one. That means we need to add each node to the other's adjacency node list. If this were a directed graph, we'd just update one adjacency list - and the order of arguments in our Graph.prototype.createEdge() method would matter.

Note also that we have to do some chaining in our expectations to reach the values we want. We start by using Map.prototype.get() to get the value associated with a node, which happens to be a set. Then we use Set.prototype.has() to determine whether the set actually has the node we are looking for.

If the worst-case scenario in terms of Big O is O(N) for both of these operations, that means with chaining we have O(2N) - so still O(N). However, the worst-case will be rare and the average-case will be sub-linear complexity.

Here's the method to get this test to pass:

src/graph.js

...
  createEdge(node1, node2) {
    let set1 = this.adjacencyList.get(node1);
    let set2 = this.adjacencyList.get(node2);
    set1.add(node2);
    set2.add(node1);
  }
...

To create an edge, we need information about the nodes we want to create an edge between. That means our method takes two arguments. We need to grab the set associated with each node. Then we need to add each node to the other node's list of adjacent nodes. Remember that because these are sets, we don't need to worry about duplicates - JavaScript will take care of that for us.

We can also make our method more concise with additional chaining:

src/graph.js

createEdge(node1, node2) {
  this.adjacencyList.get(node1).add(node2);
  this.adjacencyList.get(node2).add(node1);
}


## Checking to See if an Edge Exists

We could just do something like the following whenever we want to see if an edge (in this case, a friendship) exists:

graph.adjacencyList.get("Ada").has("Jasmine")

This would check to see if Ada and Jasmine are friends.

However, this is pretty important functionality for our application to have. That means we should add it.

Let's start with the easiest behavior - just returning false.

__tests__/graph.test.js

...
  test('check to see if edge exists in graph', () => {
    graph.addNode("Jasmine");
    graph.addNode("Ada");
    expect(graph.hasEdge("Jasmine", "Ada")).toEqual(false);
  });
...

Here's the implementation:

src/graph.js

...
  hasEdge(node1, node2) {
    return false;
  }
...

Now that we have that test passing, we're ready to move on. Typically, we won't update tests we've already written but we are going to make an exception here.

We will update the following test to use our new method:

__tests__/graph.test.js

...
  test('add an edge between two nodes', () => {
    graph.addNode("Jasmine");
    graph.addNode("Ada");
    graph.createEdge("Jasmine", "Ada");
    expect(graph.adjacencyList.get("Jasmine").has("Ada")).toEqual(true);
    expect(graph.adjacencyList.get("Ada").has("Jasmine")).toEqual(true);
  });
...

Instead of expecting graph.adjacencyList.get("Ada").has("Jasmine") to equal true, why not just apply our new method instead?

Our updated test looks like this:

__tests__/graph.test.js

...
  test('add an edge between two nodes', () => {
    graph.addNode("Jasmine");
    graph.addNode("Ada");
    graph.createEdge("Jasmine", "Ada");
    expect(graph.hasEdge("Ada", "Jasmine")).toEqual(true);
  });
...

As we can see here, we now expect graph.hasEdge("Ada", "Jasmine")).toEqual(true);.

This test will fail - let's update our Graph.prototype.hasEdge() method now:

src/graph.js

...
  hasEdge(node1, node2) {
    if (this.adjacencyList.get(node1).has(node2)) {
      return true
    }
    return false;
  }
...

Because this is an undirected graph, it doesn't matter which way we check the relationship - each node is adjacent to the other node. So we just check that the set associated with the node1 key includes node2. If it does, our method will return true.

That's all we need to get all our tests passing.


## Removing Edges

We should also be able to remove an edge as well - no more friendship, sadly…

Here's the test:

__tests__/graph.test.js

...
  test('remove an edge between two nodes', () => {
    graph.addNode("Jasmine");
    graph.addNode("Ada");
    graph.createEdge("Jasmine", "Ada");
    graph.removeEdge("Jasmine", "Ada");
    expect(graph.hasEdge("Ada", "Jasmine")).toEqual(false);
  });

In this test, we first create two nodes and add an edge. Next, we'll call graph.removeEdge("Jasmine", "Ada");. Based on removing that edge, our expectation should show that there's no longer an edge between the Ada and Jasmine nodes.

Now let's get this test passing:

src/graph.js

...
  removeEdge(node1, node2) {
    this.adjacencyList.get(node1).delete(node2);
    this.adjacencyList.get(node2).delete(node1);
  }
...

Because this is an undirected graph, both nodes include the other node in their set of adjacent nodes. That means we need to remove node1 from node2's adjacent nodes and vice versa. Here's another situation where having sub-linear complexity benefits us. If we were doing our own implementation with objects and arrays, we'd need to do an O(N) search to get a node and then another O(N) search to find the node to delete. Removing the node from the array would then shift all the remaining indexes of the array - yet another O(N) operation. Then we'd do that all over again going in the other direction. While O(6N) still boils down to O(N), it should be clear that having more efficient code can really help us in the long run, especially if our datasets get especially large.