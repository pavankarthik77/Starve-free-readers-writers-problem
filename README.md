# Starve-free-readers-writers-problem
## Starve Free Solution
The traditional answer to the dilemma results in either the reader or the writer becoming starved. I attempted to provide a starvation-free method in this solution. Only one assumption was made while suggesting the solution: Semaphore maintains the first in first out (FIFO) sequence when locking and releasing processes ( Semaphore uses a FIFO queue to maintain the list of blocked processes).
### Pseudocode
```cpp
void wait(Semaphore *S,int* process_id){
  S->value--;
  if(S->value < 0){
  S->Q->push(process_id);
  block(); 
  }
}
    
void signal(Semaphore *S){
  S->value++;
  if(S->value <= 0){
  int* PID = S->Q->pop();
  wakeup(PID); //this function will wakeup the process with the given pid using system calls
  }
}
```
### Semaphores Used
In this solution 3 semaphores are used along with noofrdr to keep track of number of readers entering. wrt is for writer's entry into critical section. rd_1 ensures that no 2 processes enter at a time into critical section and also ensures that noofrdr is incremented one by one as it is also a shared resource. rd_2 takes care of decrementing of noofrdr doesnt happen simultaneously.
```cpp
noofrdr=0;
wrt=1, rd_1=1, rd_2=1;
```
### Reader's Code
```cpp
do{
/* ENTRY SECTION */
       wait(rd_1,pid);            
       noofrdr++;                       //update the number of readers trying to access critical section 
       if(noofrdr==1)                   // if I am the first reader then request access to critical section
         wait(wrt);                      
       signal(rd_1);                    //release access to the read_count
/* CRITICAL SECTION */
       
/* EXIT SECTION */
       wait(rd_2,pid)            //requesting access to change read_count         
       noofrdr--;                       //a reader has finished executing critical section so read_count decrease by 1
       if(noofrdr==0)                   //if all the reader have finished executing their critical section
        signal(wrt);                       //releasing access to critical section for next reader or writer
       signal(rd_2);                    //release access to the read_count  
       
/* REMAINDER SECTION */
       
}while(true);
```
### Writer's code
```cpp
do{
/* ENTRY SECTION */
      wait(rd_1,process_id);              //waiting for its turn to get executed
      wait(wrt,process_id);               //requesting  access to the critical section
      signal(rd_1,process_id);            //releasing turn so that the next reader or writer can take the token
                                          //and can be serviced
/* CRITICAL SECTION */

/* EXIT SECTION */
      signal(wrt)                         //releasing access to critical section for next reader or writer

/* REMAINDER SECTION */

}while(true);
```
## Correctness of Solution
### Mutual Exclusion
The wrt semaphore ensures that only one writer can access the critical section at any given time, ensuring mutual exclusion between the writers. When the first reader attempts to access the critical section, it must first acquire the rwt mutex lock, ensuring mutual exclusion between the readers and writers.
### Bounded Waiting
Each reader or writer must first acquire the turn semaphore, which employs a FIFO queue for the stalled processes, before accessing the crucial region. As a result of the queue's FIFO policy, each process must wait for a fixed period of time before accessing the crucial section, satisfying the condition of bounded waiting..
### Progress Requirement
The code is written in such a way that there is no danger of deadlock, and the readers and writers take a finite amount of time to pass through the critical area, and at the conclusion of each reader writer code, they release the semaphore to allow additional processes to access the crucial region.
## References
- Operating System Concepts by Abraham Silberschatz, Peter B. Galvin, Greg Gagne 
-PSK sir's Slides
- [Wikipedia](https://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem)
