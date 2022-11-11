# CscriptParallelization

- OpenMP & MPI :::

    - File number 1# 92254702.c : 
    - CONFIGURATION : (How to configure, compile and run the file) 
        - SET THE THREAD NUMBER : 
           "export OMP_NUM_THREAS=" and entrer the wished number of thread
        - COMPILE :
           "mpicc -fopenmp -o your_wished_file_name 92254702.c"
        - RUN :
           "mpirun -np your_wished_rank_number ./your_wished_file_name"

  - OBSERVATIONS :  
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


- GPU parallelization :::
    - File number 2# 92254702.cu : 
    - CONFIGURATION : (How to configure, compile and run the file) 
        - CONFIG : 
           to run properly the code you will need a virtual machine able to host a linux OS, upon which the CUDA toolkit could be loaded and codes compiled that way
        - COMPILE :
           "nvcc 92254702.cu -o your_wished_file_name"
        - RUN :
           "./your_wished_file_name"
    - Observations : 
        - 1- The first problem was about converting the array f in 3D to the array d_f which was already define to be an array of 1D. 
        Fortunately, in fact f was only a 2D array: f[0] and f[1] don’t really matter, we only need I and j to move in f and same to move in d_f. 
        To use d_f as a 1D we only have to move according to i*512+j if we are in f[0], and move according to 512*512+i*512+j if we are in f[1], where 512*512 represents the f[0] part of the f array (to be sure to be in f[1] part).
        
        - 2- After setting this, I decided to show the number z equal to blockDim.x * blockIdx.x + threadIdx.x for each thread, to know how I would have to split the computation. 
        I observed such interesting things: the first thread had the number 0 and the last the number 511, so there were 512 threads! 512 threads a perfect number as N is equal to 512. 
        
        - 3 - So, to parallelize the array d_f (which corresponds to f in the gpu), we can split the computations among the I index of F. For each value of st (from 0 to NSTEP-1) we will give one value of I to one thread and he will compute all the 512 j values for this j: I mean, thread 0 will compute i=1, j from 0 to N-1; thread 1 computes i=2, with j from 0 to N-1; …; thread 511 computes i=511=N-1 for j from 0 to N-1.
        
        - 4 - Then I will stop all threads to wait that they all finish their task thanks to “__syncthreads();”. So, to do so I needed a share variable, different from *d_f so I created a shared double d_ff with “__shared__ double d_ff;” and for each I and j we have d_ff = d_f[i*512+j] +… followed by a line where I send the computed value d_ff in the correct index of d_f.

    - Results :
        ![image](https://user-images.githubusercontent.com/79518374/201366731-6609aec9-8ad1-4c65-9365-bfad0c1aa87f.png)
        ![image](https://user-images.githubusercontent.com/79518374/201366744-5521478d-0d47-439c-9be2-d160bbb9f4ef.png)
        - I didn’t get the exact result so I computed two times in order to frame the result in an interval of close values: on the left I launched the code normally, on the right I add +100 to NSTEP. So, I’m sure that to find 46419 I would have taken between 156.66 ms and 137.69 ms. And this shows the power of gpu computing. By sharing the computation between blocks and threads we can lower the computing time by almost 20!
        - The z line was just to check if I succeeded to enter in the gpu_kernel function. And we can see the number 287 because I printed the number of each thread. So maybe it added execution time to the gpu part. 
        - So, we can see the CUDA purpose: CUDA enables developers to speed up compute-intensive applications by harnessing the power of GPUs for the parallelizable part of the computation. And it’s exactly what we saw.
        




