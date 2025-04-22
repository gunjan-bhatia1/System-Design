# Page Rank algorithm using analogy of Party of influencers

## Story Time
* Image being at a party of 5 influencers: Tia, Sia, Kia, Pia, and Jia. Everyone is talking about each other, more you get mention the more imp you become. 
* But every mention doesn't have equal weightage, if popular person mentions it worth more than less popular does.

### **Question**
* We have to find the person who is most popular.

### Starting Point
* At beginning everyone is equal so we give pointer as 1.

### The Mentions (Links)
Here’s how the [![Mentions Connectivity Graph](Images/Mentions%20Connectivity%20Graph.png)](Images/Mentions%20Connectivity%20Graph.png)  
* Tia mentions Sia and Kia.
* Sia mentions Pia.
* Kia mentions Jia.
* Pia mentions Tia and Jia.
* Jia mentions Sia.

Mentions are link on internet when someone mention you that means they are giving you importance.

### Calculating importance
* Every mention doesn't have equal weightage, if popular person mentions it worth more than less popular does.
* We introduce **damping factor(D)** to make it more realistic.
  * 85% of importance come from people who mention 

* **Randomness factor** ((1- 0.85)/5) is calculated by (1 - D)/ N where N is total number of influencers.
  *  Randomness factor is nothing but when someone accidentally mention or accidentally click the link.

* Tia's score -> Pia(1/2) * 0.85 + ((1- 0.85)/5) = 0.455
* Sia's score -> Jia(1) * 0.85  + Tia(1/2) * 0.85 + ((1- 0.85)/5) = 1.305
* Kia's score -> Tia(1/2) * 0.85 + ((1- 0.85)/5) = 0.455
* Pia's score -> Sia(1) * 0.85 + ((1- 0.85)/5) = 0.88
* Jia's score -> Kia(1) * 0.85 + Pia(1/2) * 0.85 + ((1- 0.85)/5) = 1.305

### Formula of Page rank algorithm

**PR(A)** = (1 - D)/ N + D * Summation i = (M(A))[ PR(i)/L(i)]

* i = M(Tia) i.e. mentions of Tia -> {PIA}
* Pia mentions two people {Tia , Jia} -> L{Pia} = 2
* That's how the calculation came : Tia's score -> ((1- 0.85)/5) + 0.85 * Pia(1/2)   = 0.455

Thus, Sia and Jia are most powerful

This method is exactly how browser ranks websites. If popular websites link to yours, it becomes more important in search results. 
But not all links are the same—links from important websites give you more ranking power.