# CscriptParallelization

OpenMP & MPI :::
    File number 1# 92254702.c : 
-CONFIGURATION : (How to configure, compile and run the file) 
    -- SET THE THREAD NUMBER
       "export OMP_NUM_THREAS=" and entrer the wished number of thread
    -- COMPILE :
       "mpicc -fopenmp -o your_wished_file_name 92254702.c"
    -- RUN :
       "mpirun -np your_wished_rank_number ./your_wished_file_name"
   
-OBSERVATIONS :

  - Only OpenMP parallelization: 
    
    - 1-rank with 4-thread :
    
      - Even if you had pragma command for each loop it will not necessary lower the execution time. 
I mean, sometime when there is a loop in another, adding a pragma command for the 2 loops will give a longer execution time than with only 1 pragma on these 2 loops.
--> Let’s see that by adding or removing the « #pragma omp parallel line 81 the execution time changes.
Without we took less than 8.6 sec.
![image](https://user-images.githubusercontent.com/79518374/201192946-00a47e92-2884-42cc-ac84-386eb0d56313.png)
With we are always above 8.8 sec.
![image](https://user-images.githubusercontent.com/79518374/201193035-2572e653-6149-4d9b-885a-8043414066c2.png)
So I removed it.

      - To optimase the time it’s not enough to only add « #pragma omp parralel for » before all the for loops.

    - Comparisaon between 1 rank and 1 thread, and it with 1 rank and 4 threads :   
![image](https://user-images.githubusercontent.com/79518374/201193595-c4e28491-773e-4b28-ba8f-d0daef23fbe8.png) 
It’s surprising but 1 rank + 1 thread is faster than 4 threads.
==> I read on internet that it can be explain by the fact that as we haven’t a big enough code, the thread creation time, the walk between threads and their synchronization add time.
So with a to little programm seeing the difference is difficult. 
More, often small codes have bigger time with threads.




  - Only MPI parallelization:  4-ranks with 1-thread
    - EXPLANATIONS : 
I couldn’t handle the sharing of the array when all the process where running so I must wait the previous finish to calculate its part.
So in my code you will see that I cut the loop about the steps, it’s because when I was cuting the loop for the array F and launch the process at the same time I couldn’t receive properly each parts so I coudn’t find the same result as when we basically launch « fdm_sample.c ». 
But with my programm I find the good result, unfortunatelly even if I use MPI tp share the calcul to the process i don’t really use all its power as the processes run turn by turn. 

    - To really analyse we should compare the time of each process and imagine that they are running at the same time, so the global time decrease.
![image](https://user-images.githubusercontent.com/79518374/201195146-9ca1050e-db8a-4f0d-8603-a872e69fe061.png)
As expected the global time is bigger when I put 4 ranks in my program ! Because they don't run at the same time.
However, we still see the impact of the cores parallelization : 
The first core took 2.33 sec to compute it’s part. 
Then he send it to the second who took 5.01-2.33 = 2.68 sec. 
Then the third followed and took 7.66 – 5.01 = 2.65sec. 
And the last process who gives the final result took 10.33-7.66 = 2.67 sec.
If I had succeed in my parallelization the global time would be lower than 8.56 sec. 
By the way it’s not necessary the bigger of the last four times as there is some time taken to exchange the data, etc. 
I’m pretty sure that we would have obtain less than 5.5 sec, which is 3 sec less than with one single rank. 
It shows all the power and usefulness of the MPI parallelization.





  - Hybrid parallelization: 2-ranks with 2-thread
    - Here we could expect to have a bigger time with 2 ranks and 2 threads than with only one, like previously.
But it’s not the case.
![image](https://user-images.githubusercontent.com/79518374/201357477-f301d62d-e78c-41d7-a256-73a81f9c6dd5.png)
By removing one thread we get 0.2 more secondes. 
It means that the thread usefulness appears here ! 
I can’t explain why, but in this case 2 threads is better.
Maybe it’s always the case with my program if I enter a rank greater than 1. 
But it’s light, problably because it follow what i said previously : on small codes thread parallelization will add time as we need time to communicate between it, to create it, etc.






