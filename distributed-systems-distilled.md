# Chapter 1

* the three most important considerations in software system:

## Reliability
* system should continue to work __corectly__ even when things go wrong (software, hardware and user faults)

### Hardware faults
* hard disk crashes, faulty RAM, unplugged wires, blackouts
* simple fix is to have multi machine redundancy
* lately as applications have reached such large scales, software fault tolerance is also used 

### Software errors
* these types of errors are harder to anticipate, as they are correlated across nodes
* usually these types of bugs make an assumption about its environment that at some point is no longer true
* no quick solution, lots of small things help:
  * think carefully about assumptions + interactions
  * thorough testing
  * process isolation

### Human Errors
* Humans, at scale, are faulty and unreliable
* things that help
  * make software easy to do the right thing, hard to do wrong thing
  * decouple people from places where they can cause damage
  * test a lot
  * allow quick recovery
  * set up monitoring
  * manage people well


## Scalability
* as the data and traffic volume grow, there should be reasonable ways to handle that growth

## Maintainability
* Over time, people will work on the system. They should be able to maintain and add to it productively