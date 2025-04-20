## Garbage Collection

* It is basically reclaiming the memory of the objects which are no longer in use. To prevent memory leaks GC is done.
  * When GC is done application stops working in traditional GC(serial GC).
  * There are also GC which work in parallel with application.
* **Heap Size** is also responsible for long pauses in GC.
  * Heap is responsible for storing objects.
  * Heap size is large then GC has more memory to traverse and analyze when determining which objects to collect.
  * GC tuning can be done using ```java -Xms512m -Xmx2g MyApp``` **-Xms (initial heap size) and -Xmx (maximum heap size).**
    * Large heap size prevent frequent gc but longer gc pause.
* **System Activity** : If the system is under heavy load. GC will take time.
  * In that case GC have to compete with other operation that need resource.
* **Memory Fragmentation** : In case memory is non-contiguous blocks or divided into small area.
  * GC will need time for memory compaction(merging together free memory) (G1 GC or serial GC).

### GC Algorithms

* **Minor GC** : cleans the young small object and pauses are short.
* **Major GC/Full GC**: Reclaim both old and young object. These can be longer if heap size is large.
* **G1 GC**: It takes care of the pause and high throughput(cleaning objects). 
  * In this we divide into region. 
  * And region with live objects are cleaned up first.
  * In case there is no memory G1 GC also do full gc and stop everything.

### Minimize GC Pause

* **Choose the Right GC** : For low latency applications we might prefer G1 GC, ZGC, or Shenandoah GC. Reduced pause time.
  * ZGC and Shenandoah GC can perform GC concurrently with application threads, aiming for very low pause times.
* **Tune heap size.**
* **Use parallel GC :** In multi core system parallel GC use multiple thread for GC reducing impact on resources
* **Optimise object creation :** Prevent frequent object creation in tight loops this will lead to frequent GC and frequent pause. Increased cpu usage
 
  | **Do**                                                        | **Example**                                               | **Don't**                                | **Example**                                        |
  |---------------------------------------------------------------|-----------------------------------------------------------|------------------------------------------|----------------------------------------------------|
  | **Reuse objects, buffers, builders**                          | `StringBuilder sb = new StringBuilder(); sb.append(val);` | **Create new objects repeatedly**        | `String result = ""; result += val;` inside a loop |
  | **Prefer primitives over wrappers**                           | `int count = 0;`                                          | **Box primitives unnecessarily**         | `Integer count = new Integer(0);`                  |
  | **Use StringBuilder / StringBuffer for string concatenation** | `sb.append("Value: ").append(i);`                         | **Use + in loops**                       | `"Value: " + i`                                    |
  | **Keep object scope narrow**                                  | **Declare variables inside the loop or method**           | **Hold onto objects longer than needed** | **Declare objects outside and reuse accidentally** |
  | **Clear temp collections when done**                          | `list.clear(); list = null;`                              | **Leave them populated for GC**          | **Never clear or null out after use**              |
  | **Use Collections.emptyList() instead of null**               | `return Collections.emptyList();`                         | **Return null for empty lists**          | `return null;`                                     |

  * **Reduce long-lived objects**

  | **Do**                                                       | **Example**                                               | **Don't**                                  | **Example**                                  |
  |--------------------------------------------------------------|-----------------------------------------------------------|--------------------------------------------|----------------------------------------------|
  | **Use object pooling for costly objects**                    | `Use ThreadPoolExecutor or a DB connection pool`          | **Create new objects every time**          | `new Connection() for every query`           |
  | **Use WeakReference / SoftReference for cache-like objects** | `WeakReference<MyObject> ref = new WeakReference<>(obj);` | **Retain strong references unnecessarily** | `Keep everything in a static List`           |
  | **Share immutable empty instances**                          | `Collections.emptyList()`                                 | **Create new empty lists repeatedly**      | `new ArrayList<>() each time`                |
  | **Release when no longer needed**                            | `cache = null;` after use                                 | **Hold onto objects forever**              | `Keep cache alive unnecessarily`             |
  | **Avoid unnecessary static/global references**               | `Use Dependency Injection or managed singletons`          | **Make everything static**                 | `static final List cache = new ArrayList();` |



