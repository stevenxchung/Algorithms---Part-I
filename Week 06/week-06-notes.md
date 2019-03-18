## Week 6: Hash Tables

> We begin by describing the desirable properties of hash function and how to implement them in Java, including a fundamental tenet known as the *uniform hashing assumption* that underlies the potential success of a hashing application. Then, we consider two strategies for implementing hash tables—*separate chaining* and *linear probing*. Both strategies yield constant-time performance for search and insert under the uniform hashing assumption. We conclude with applications of symbol tables including sets, dictionary clients, indexing clients, and sparse vectors.

### Hash Tables
* Another approach to implementing symbol tables that can also be very effective in a practical application is hashing
* Hashing involves using a **hash function** - a method for computing array index from a key
* When using a hash function we want to scramble the keys uniformly to produce a table index:
  * Efficiently computable
  * Each table index equally likely for each key
* Some examples of bad hashing:
  * Using the first three digits of a phone number
  * Using the first three digits of SSN
* Some examples of good hashing:
  * Using the last digits of a phone number or SSN
* We also need a different approach for each key type
* All Java classes inherit a method `hashCode()` which returns a 32-bit `int`
* In general if `x.equals(y)`, then `(x.hashCode() == y.hashCode())`
* It is highly desirable to have `!x.equals(y)`, then `(x.hashCode() != y.hashCode())`
* The default implementation for `hashCode()` uses the memory address of x

* Java implementation is different for `Integer`, `Boolean`, `Double`, etc. below is an implementation of `hashCode()` for the `Integer` type:
```java
public final class Integer {
  private final int value {
    // ...
    public int hashCode() {
      return value;
    }
  }
}
```

* The standard recipe for hash code design is as follows:
  * Combine each significant field using the *31 * x + y* rule
  * If field is a primitive type, use wrapper type `hashCode()`
  * If field is null, return `0`
  * If field is a reference type, use `hashCode()`
  * If field is an array, apply to each entry

### Separate Chaining
* Separate chaining is a collision red solution strategy that makes use of elementary linked list
* During a collision, two distinct keys hashing to same index:
  * Birthday problem - can't avoid collisions unless you have a ridiculous (quadratic) amount of memory
  * Coupon collector + load balancing - collisions will be evenly distributed
* How do we deal with collisions efficiently?
  * Use an array of *M < N* linked lists:
    * Hash: map key to integer *i* between *0* and *M - 1*
    * Insert: put at front of *ith* chain (if not already there)
    * Search: need to search only *ith* chain

* Separate chaining implementation in Java is as follows:
```java
public class SeparateChainHashST<Key, Value> {
  // NOTE: Array doubling and halving code omitted
  private int M = 97; // Number of chains
  private Node[] st = new Node[M]; // Array of chains

  private static class Node {
    private Object key; // No generic array creation
    private Object val; // Declare key and value of type Object
    private Node next;
    // ...
  }

  public Value get(Key key) {
    int i = hash(key);
    for (Node x = st[i]; x != null; x = x.next) {
      if (key.equals(x.key)) {
        return (Value) x.val;
      }
    }
    return null;
  }
}
```

* For insertion the code becomes:
```java
public class SeparateChainHashST<Key, Value> {
  // NOTE: Array doubling and halving code omitted
  private int M = 97; // Number of chains
  private Node[] st = new Node[M]; // Array of chains

  private static class Node {
    private Object key; // No generic array creation
    private Object val; // Declare key and value of type Object
    private Node next;
    // ...
  }

  public void put(Key key, Value val) {
    int i = hash(key);
    for (Node x = st[i]; x != null; x = x.next) {
      if (key.equals(x.key)) {
        x.val = val;
        return;
      }
    }
    st[i] = new Node(key, val, st[i]);
  }
}
```

* The number of probes (`equals()` and `hashCode()`) for search/insert is proportional to *N / M*:
  * *M* too large -> too many empty chains
  * *M* too small -> chains too long
  * Typical choice: *M ~ N / 5* -> constant-time operation

### Linear Probing
* Another closure resolution method is known as linear probing
* Linear probing is also known as *open addressing* - when a new key collides, find next empty slot, and put it there
* The methods for linear probing for hash tables is as follows:
  * **Hash** - map key to integer *i* between *0* and *M - 1*
  * **Insert** - put a table at index *i* if free; if not try *i + 1*, *i + 2*, etc.
  * **Search** - search table index *i*; if occupied but no match, try *i + 1*, *i + 2*, etc.
  * Note: array size *M* **must be** greater than number of key-value pairs *N*

* The Java implementation for linear probing:
```java
public class LinearProbingHashST<Key, Value> {
  // NOTE: Array doubling and halving code omitted
  private int M = 30001;
  private Value[] vals = (Value[]) new Object[M];
  private Key[] keys = (Key[]) new Object[M];

  private int hash(Key key) {
    /* as before */
  }

  private void put (Key key, Value val) {
    int i;
    for (i = hash(key); keys[i] != null; i = (i + 1) % M) {
      if (keys[i].equals(key)) {
        break;
      }
    }
    keys[i] = key;
    vals[i] = val;
  }

  private Value get(Key key) {
    for (int i = hash(key); keys[i] != null; i = (i + 1) % M) {
      if (key.equals(keys[i])) {
        return vals[i];
      }
    }
    return null;
  }
}
```

* Clustering became a problem:
  * **Cluster** - a contiguous block of items
  * **Observation** - new keys likely to hash into middle of big clusters
* Under uniform hashing assumption, the average of number of probes in a linear probing hash table of size *M* that contains *N = a * M* keys is:
  * Search hit ~ *(1 / 2) * (1 + (1 / (1 - a)))*
  * Search miss/insert ~ *(1 / 2) * (1 + (1 / (1 - a)^2))*
* In summary, for linear probing:
  * *M* too large -> too many empty array entries
  * *M* too small -> search time blows up
  * Typical choice: *a ~ N / M ~ 1/2* -> constant-time operation