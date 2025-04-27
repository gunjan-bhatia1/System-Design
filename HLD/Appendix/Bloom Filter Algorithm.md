# Bloom Filters

* We start with array of bits all set to 0
* We have 3 hash functions.
* To add an item we hash the key with all the hash function and set bit to 1.
* To check if item exist we rehash the key and if bit is set yes key exist if it's 0 it don't exist.
  * Suppose you add "apple", and hash functions say indexes 2, 4, 6 → set those bits.
  * And to check if apple exist we rehash we see bits are set.
  * And if we check for banana bit will be 0. 

## Real life eg:

* **Web browsers(Safe browsing)** : 
  * **Google use bloom filter** to check if url is bad before hitting.
    * It's fast because it doesn't look into entire db

* **Database Query Optimisation:**
  * Cassandra , HBase, BigDataTable use bloom filter to check if key might be present rather than reading disk.

* Distributed System -> Caches
  * Akamai, cloudflare -> Uses bloom filter to check if file exist or not in cache.

* **Email Spamming system:**  
  * The email server uses bloom filter to check if an email signature matches known spam without scanning huge spam lists.

* Network - Routing : Router can use bloom filter to check if route exist or not.

* **Spell Checker** uses bloom filter to check if spelling is correct or not.


