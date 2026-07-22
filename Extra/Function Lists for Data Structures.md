
Stack : 
push(Element e) - pushes an element 
pop() - return an element from stack and throws exception if stack is empty
peek() - returns the top of the stack and rthrows exception is stack is empty;
size() 
isEmpty()
clear()
how to create :
```
Stack<t> st = new Stack<>();
```




Queue :
add() - adds and element but throws an exception is capacity is exceeded.
remove() - removes an element but throws an exception if queue is empty
element() - retreives the head (without removing it)of the queue and throws and exception if queue is empty

alternative methods :
offer() - true if added false if not
poll() - return the value or null if queue is empty
peek() - returns the head if queue is not empty and returns null if it is 

clear()

how to make : 
Queue <t> queue = new LinkedList<>();
Queue<t> pq = new PriorityQueue<>();// default is min element at the top
Queue<t> reversePq = new PriorityQueue<>(Collections.reverseOrder()); //max pq

Queue<t> ad = new ArrayDequeu<>();


so queue is an interface which is extended by Deque interface which is them implemented by two classes LinkedList and ArrayDeque, also Queue is direcly implemented by PriorityQueue as well.



Linked List :

add() - adds an item to the list
addFirst() - add at the start
addLast() - adds at the last same as add
size()
element() - same as getFirst()
getFirst()
getLast()
isEmpty()
offerFirst()
offerLast()
peekFirst()
peekLast()
poll() - retreives and removes the first item in the list
pollFirst() - same as poll();
pollLast() - 
pop() - remove and return first item 
push() - adds an item to the beggining 

as we can see that Linked list contains all the methods present in the dequeue as well since it implements that interface as well.





