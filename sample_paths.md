# Sample Paths


*Source: Stochastic Modeling: Analysis and Simulation. Barry L. Nelson
1995.*

</br>

***Motivating Example:** A copy shop currently has one copy machine and
one clerk. Customers arrive and form a single queue, then are provide
full service in the order of arrival. The copy shop would like to expand
services by providing a second copy machine and a second clerk. The shop
would like to know whether it would be more effective to continue
providing full service to all customers, or to allow the new copy
machine to be available for self-service.*

*The single employee at the copy shop collects data for two hours one
morning. Customers are counted as they arrive. Arrival times are
indicated by* `clock = <hour>.<minute>`. `action` refers to customer
`a = arrival`, `f = finish`. `comment` includes whether the customer
required special handling services. A total of 20 customers were
observed.

       clock customer action           comment
    1   9.00        0   <NA>        shop opens
    2   9.12        1      a                 0
    3   9.14        2      a                 0
    4   9.17        3      a                 0
    5   9.19        1      f   collate, staple
    6   9.21        2      f                 0
    7   9.22        3      f                 0
    8   9.38        4      a                 0
    9   9.39        5      a                 0
    10  9.41        4      f                 0
    11  9.43        6      a                 0
    12  9.45        7      a                 0
    13  9.52        8      a                 0
    14  9.57        5      f     special paper
    15  9.58        9      a                 0
    16 10.00        6      f                 0
    17 10.01        7      f                 0
    18 10.08        8      f                 0
    19 10.11        9      f   collate, covers
    20 10.13       10      a                 0
    21 10.14       11      a                 0
    22 10.16       12      a                 0
    23 10.19       13      a                 0
    24 10.24       10      f two-sided, staple
    25 10.26       11      f                 0
    26 10.28       14      a                 0
    27 10.36       12      f                 0
    28 10.36       15      a                 0
    29 10.38       13      f                 0
    30 10.39       14      f                 0
    31 10.42       15      f   collate, staple
    32 10.45       16      a                 0
    33 10.47       16      f                 0
    34 10.48       17      a                 0
    35 10.50       17      f                 0
    36 10.53       18      a                 0
    37 10.58       19      a                 0
    38 11.00       20      a                 0
    39 11.01       18      f     special paper
    40 11.07       19      f two-sided, staple
    41 11.09       20      f   collate, staple

</br>

## Sample Path Decomposition

**Stochastic processes** are those which involve randomness. In the
example above, we can imagine several random processes, including
customer arrival times, finish times, service times, and queue times.

To calculate service times from the data above, we will need to first
perform several pre-processing (i.e., data handling) steps. First, we
must convert clock time into a count of minutes since *time = 0*, the
time when the shop opens. To prepare the data for visualizations and
simulations, we will convert the `action` column into arrival and finish
columns. We will also convert the `comments` column into an indicator
variable of `self-service = 0 / full-service = 1`. Finally, we will
reformat the dataset to customer number as row identifier (rather than
time).

``` r
## Calculate service_length, which is assumed to be independent of proposed changes
   # 1) Convert clock times to minutes past "start" time = 0
   # 2) Convert action column into arrive and finish columns
   # 3) Convert comment into dichotomous self-service (0) / full-service (1)
   # 4) Reformat dataset to customer number as row identifier (rather than time)

   #- Copy previous dataset
data1 <- raw_data

   #- 1) Convert all clock times to minutes past store opening
data1$time <- data1$clock - 9   # set store open to 0
      # time = transformed time

data1$time <- ifelse(data1$time < 1,   # if arrival in 9 o'clock hour
                     data1$time*100,       # transform by making decimal into minutes
                     data1$time)           # else do not transform

data1$time <- ifelse(data1$time < 2,   # if arrival in 10 o'clock hour
                     (data1$time - 1)*100 + 60, # remove 10 o'clock indicator
                     data1$time)                # transform decimal to minutes
                                                    # add 60 minutes for 10 o'clock hour

data1$time <- ifelse(data1$time < 3,   # if arrival in 11 o'clock hour
                     (data1$time - 2)*100 + 60*2,   # remove 11 o'clock indicator
                     data1$time)                    # transform decimal to minutes
                                                        # add 120 minutes for 11 o'clock hour

data1$time[1] <- 0  # fix time = 0
data1  #-- check Step 1 was performed correctly. 
```

       clock customer action           comment time
    1   9.00        0   <NA>        shop opens    0
    2   9.12        1      a                 0   12
    3   9.14        2      a                 0   14
    4   9.17        3      a                 0   17
    5   9.19        1      f   collate, staple   19
    6   9.21        2      f                 0   21
    7   9.22        3      f                 0   22
    8   9.38        4      a                 0   38
    9   9.39        5      a                 0   39
    10  9.41        4      f                 0   41
    11  9.43        6      a                 0   43
    12  9.45        7      a                 0   45
    13  9.52        8      a                 0   52
    14  9.57        5      f     special paper   57
    15  9.58        9      a                 0   58
    16 10.00        6      f                 0   60
    17 10.01        7      f                 0   61
    18 10.08        8      f                 0   68
    19 10.11        9      f   collate, covers   71
    20 10.13       10      a                 0   73
    21 10.14       11      a                 0   74
    22 10.16       12      a                 0   76
    23 10.19       13      a                 0   79
    24 10.24       10      f two-sided, staple   84
    25 10.26       11      f                 0   86
    26 10.28       14      a                 0   88
    27 10.36       12      f                 0   96
    28 10.36       15      a                 0   96
    29 10.38       13      f                 0   98
    30 10.39       14      f                 0   99
    31 10.42       15      f   collate, staple  102
    32 10.45       16      a                 0  105
    33 10.47       16      f                 0  107
    34 10.48       17      a                 0  108
    35 10.50       17      f                 0  110
    36 10.53       18      a                 0  113
    37 10.58       19      a                 0  118
    38 11.00       20      a                 0  120
    39 11.01       18      f     special paper  121
    40 11.07       19      f two-sided, staple  127
    41 11.09       20      f   collate, staple  129

``` r
  #- 2) Split times into arrivals and finish to prepare for reformating dataset
     #- Initiate a new "arrive" column
data1$arrive <- rep(0, nrow(data1))
    #- Transfer times to arrive if arrival times
data1$arrive <- ifelse(data1$action == "a", data1$customer, 0)
 
    #- Initiate a new "finish" column
data1$finish <- rep(0, nrow(data1))
    #- Transfer times to finish if finish times
data1$finish <- ifelse(data1$action == "f", data1$customer, 0)

data1 #-- Check Step 2 was performed correctly
```

       clock customer action           comment time arrive finish
    1   9.00        0   <NA>        shop opens    0     NA     NA
    2   9.12        1      a                 0   12      1      0
    3   9.14        2      a                 0   14      2      0
    4   9.17        3      a                 0   17      3      0
    5   9.19        1      f   collate, staple   19      0      1
    6   9.21        2      f                 0   21      0      2
    7   9.22        3      f                 0   22      0      3
    8   9.38        4      a                 0   38      4      0
    9   9.39        5      a                 0   39      5      0
    10  9.41        4      f                 0   41      0      4
    11  9.43        6      a                 0   43      6      0
    12  9.45        7      a                 0   45      7      0
    13  9.52        8      a                 0   52      8      0
    14  9.57        5      f     special paper   57      0      5
    15  9.58        9      a                 0   58      9      0
    16 10.00        6      f                 0   60      0      6
    17 10.01        7      f                 0   61      0      7
    18 10.08        8      f                 0   68      0      8
    19 10.11        9      f   collate, covers   71      0      9
    20 10.13       10      a                 0   73     10      0
    21 10.14       11      a                 0   74     11      0
    22 10.16       12      a                 0   76     12      0
    23 10.19       13      a                 0   79     13      0
    24 10.24       10      f two-sided, staple   84      0     10
    25 10.26       11      f                 0   86      0     11
    26 10.28       14      a                 0   88     14      0
    27 10.36       12      f                 0   96      0     12
    28 10.36       15      a                 0   96     15      0
    29 10.38       13      f                 0   98      0     13
    30 10.39       14      f                 0   99      0     14
    31 10.42       15      f   collate, staple  102      0     15
    32 10.45       16      a                 0  105     16      0
    33 10.47       16      f                 0  107      0     16
    34 10.48       17      a                 0  108     17      0
    35 10.50       17      f                 0  110      0     17
    36 10.53       18      a                 0  113     18      0
    37 10.58       19      a                 0  118     19      0
    38 11.00       20      a                 0  120     20      0
    39 11.01       18      f     special paper  121      0     18
    40 11.07       19      f two-sided, staple  127      0     19
    41 11.09       20      f   collate, staple  129      0     20

``` r
 #- 3) Convert comment column into self-serve/full-serve
    #- Initiate column
data1$full_service <- rep(0, nrow(data1))
    #- replace any characters with 1
data1$full_service <- ifelse(data1$comment == 0, 0, 1)

data1 #-- Check Step 3 was performed correctly
```

       clock customer action           comment time arrive finish full_service
    1   9.00        0   <NA>        shop opens    0     NA     NA            1
    2   9.12        1      a                 0   12      1      0            0
    3   9.14        2      a                 0   14      2      0            0
    4   9.17        3      a                 0   17      3      0            0
    5   9.19        1      f   collate, staple   19      0      1            1
    6   9.21        2      f                 0   21      0      2            0
    7   9.22        3      f                 0   22      0      3            0
    8   9.38        4      a                 0   38      4      0            0
    9   9.39        5      a                 0   39      5      0            0
    10  9.41        4      f                 0   41      0      4            0
    11  9.43        6      a                 0   43      6      0            0
    12  9.45        7      a                 0   45      7      0            0
    13  9.52        8      a                 0   52      8      0            0
    14  9.57        5      f     special paper   57      0      5            1
    15  9.58        9      a                 0   58      9      0            0
    16 10.00        6      f                 0   60      0      6            0
    17 10.01        7      f                 0   61      0      7            0
    18 10.08        8      f                 0   68      0      8            0
    19 10.11        9      f   collate, covers   71      0      9            1
    20 10.13       10      a                 0   73     10      0            0
    21 10.14       11      a                 0   74     11      0            0
    22 10.16       12      a                 0   76     12      0            0
    23 10.19       13      a                 0   79     13      0            0
    24 10.24       10      f two-sided, staple   84      0     10            1
    25 10.26       11      f                 0   86      0     11            0
    26 10.28       14      a                 0   88     14      0            0
    27 10.36       12      f                 0   96      0     12            0
    28 10.36       15      a                 0   96     15      0            0
    29 10.38       13      f                 0   98      0     13            0
    30 10.39       14      f                 0   99      0     14            0
    31 10.42       15      f   collate, staple  102      0     15            1
    32 10.45       16      a                 0  105     16      0            0
    33 10.47       16      f                 0  107      0     16            0
    34 10.48       17      a                 0  108     17      0            0
    35 10.50       17      f                 0  110      0     17            0
    36 10.53       18      a                 0  113     18      0            0
    37 10.58       19      a                 0  118     19      0            0
    38 11.00       20      a                 0  120     20      0            0
    39 11.01       18      f     special paper  121      0     18            1
    40 11.07       19      f two-sided, staple  127      0     19            1
    41 11.09       20      f   collate, staple  129      0     20            1

``` r
 #- 4) Recast dataset to customer ID as row identifier
data2 <- data1[-1,5:8]  # Reorder columns and drop unnecessary ones
   
   #- initiate two new columns, time1 (= arrive time) and time2 (= finish time)
time1 <- rep(0, 20)
time2 <- rep(0, 20)
for(i in 1:20)  {     #for each of 1:20 customers
  time1[i] <- data2$time[which(data2$arrive == i)]  # for arrival == CustomerID, collect time
  time2[i] <- data2$time[which(data2$finish == i)]  # for finish == CustomerID, collect time
}
 
   #- Service time for customer 1 is time2[1] - time1[1]
   #- For all others, service_length begins at max(previous customer's finish, current customer's start)
   #- Also capture wait_time_factual for testing simulations later
service_length <- rep(0, 20)
wait_time_factual <- rep(0, 20)
service_length[1] <- time2[1] - time1[1]
for(i in 2:20) {
  start_time <- max(time2[i-1], time1[i])
  wait_time_factual[i] <- ifelse(time2[i-1] > time1[i], diff(c(time1[i], time2[i-1])), 0)
  service_length[i] <- time2[i] - start_time
}

   #- Flag full-service by customer ID instead of time
   #- Also capture arrival times for simulations later
full_service <- rep(0, 20)
arrive_time <- rep(0, 20)
for(i in 1:20) {
  full_service[i] <- ifelse(data2$full_service[which(data2$finish == i)] == 0, 0, 1)
  arrive_time[i] <- data2$time[data2$arrive == i]
}


table2 <- data.frame(customer = 1:20,       # Table 2.2
                     service_length = service_length, 
                     full_service = full_service, 
                     arrive_time = arrive_time, 
                     wait_time_factual = wait_time_factual
                     )
table2
```

       customer service_length full_service arrive_time wait_time_factual
    1         1              7            1          12                 0
    2         2              2            0          14                 5
    3         3              1            0          17                 4
    4         4              3            0          38                 0
    5         5             16            1          39                 2
    6         6              3            0          43                14
    7         7              1            0          45                15
    8         8              7            0          52                 9
    9         9              3            1          58                10
    10       10             11            1          73                 0
    11       11              2            0          74                10
    12       12             10            0          76                10
    13       13              2            0          79                17
    14       14              1            0          88                10
    15       15              3            1          96                 3
    16       16              2            0         105                 0
    17       17              2            0         108                 0
    18       18              8            1         113                 0
    19       19              6            1         118                 3
    20       20              2            1         120                 7

Now that we have processed the data, we can visualize service times for
all customers, as well as stratify by potential service types, with
potential self-service customers as those with no special handling
needs.

``` r
## Calculate means/sds for reference in figure
mean_f <- mean(table2$service_length[table2$full_service == 1])
sd_f <- sd(table2$service_length[table2$full_service == 1])

mean_s <- mean(table2$service_length[table2$full_service == 0])
sd_s <- sd(table2$service_length[table2$full_service == 0])

mean_a <- mean(table2$service_length)
sd_a <- sd(table2$service_length)


## Visualization
par(mfrow = c(3,1))
hist(table2$service_length[table2$full_service == 0], breaks = seq(0, 20, by = 2.5), 
     main = "Self Service")
text(x = 13, y = 7, label = paste("mean = ", round(mean_s, 4), ", sd = ", round(sd_s, 3)))

hist(table2$service_length[table2$full_service == 1], breaks = seq(0, 20, by = 2.5),
     main = "Full Service", xlim = c(0,20))
text(x = 13, y = 1.75, label = paste("mean = ", round(mean_f, 4), ", sd = ", round(sd_f, 3)))

hist(table2$service_length, breaks = seq(0, 20, by = 1.25), 
     main = "All Service")
text(x = 13, y = 5, label = paste("mean = ", round(mean_a, 4), ", sd = ", round(sd_a, 3)))
```

![](sample_paths.markdown_strict_files/figure-markdown_strict/unnamed-chunk-3-1.png)

This analysis provides support that potentially self-service customers
do indeed have shorter service times, and so providing them with a
self-service machine and a separate queue may reduce customer wait
times.

This is an example of a **dynamic system**, since what happens to one
customer affects the actions of other customers. So far we have examined
**sample paths**, which include both factors that are under control
(*system logic*) and factors that are not under control (*system
inputs*).

------------------------------------------------------------------------

*A **sample path** is a record of the time-dependent behavior of a
system.*

***Sample path decomposition** represents a sample path as inputs and
logic.*

***Simulation** generates new sample paths without building the new
system.*

***Sample-path analysis** extracts system performance measures from
sample paths.*

------------------------------------------------------------------------

</br>

## Simulating the Self-Service System

To generate a sample path for the self-service system, we need to define
its system logic.

-   There will be two queues for customers, who will self-select as
    self-service (those who require no special handling) and
    full-service (those who will require special handling performed by
    the employee).

-   Customers will always join the correct queue.

-   Service times previously observed will be representative of
    simulation service times.

To perform this simulation, we will simulate the two queues - one
self-service and one full-service. First, stratify the data by service
type, then re-calculate service times and wait times within the new
queue system.

``` r
  #- Split dataset into self-serve and full-serve
self_queue <- table2[table2$full_service == 0,]
full_queue <- table2[table2$full_service == 1,]
 

  #- To simulate the self-service option, repeat previous service times, wait times calculations within each stratified dataset 
n = nrow(self_queue)
self_queue$customer_sim1 <- 1:n
self_queue$finish_time_sim1 <- rep(0, n)
naive_finish <- self_queue$arrive_time + self_queue$service_length
self_queue$finish_time_sim1[1] <- naive_finish[1]
self_queue$wait_time_sim1 <- rep(0, n)

for(i in 2:nrow(self_queue)) {
  self_queue$finish_time_sim1[i] <- ifelse(self_queue$arrive_time[i] > naive_finish[i-1],   # if arrive time is after previous person finishes
                                           self_queue$arrive_time[i] + self_queue$service_length[i],         # finish_time = arrive + service
                                           self_queue$finish_time_sim1[i-1] + self_queue$service_length[i])  # finish_time = finish[i-1] + service 

  self_queue$wait_time_sim1[i] <- ifelse(self_queue$arrive_time[i] > naive_finish[i-1],     # if arrive time is after previous person finishes
                                   0,                                                            # wait time is zero
                                   self_queue$finish_time_sim[i-1] - self_queue$arrive_time[i])
}


n2 = nrow(full_queue)
full_queue$customer_sim1 <- 1:n2
full_queue$finish_time_sim1 <- rep(0, n2)
naive_finish <- full_queue$arrive_time + full_queue$service_length
full_queue$finish_time_sim1[1] <- naive_finish[1]
full_queue$wait_time_sim1 <- rep(0, n2)

for(i in 2:nrow(full_queue)) {
  full_queue$finish_time_sim1[i] <- ifelse(full_queue$arrive_time[i] > naive_finish[i-1],   # if arrive time is after previous person finishes
                                           full_queue$arrive_time[i] + full_queue$service_length[i],         # finish_time = arrive + service
                                           full_queue$finish_time_sim1[i-1] + full_queue$service_length[i])  # finish_time = finish[i-1] + service 
  
  full_queue$wait_time_sim1[i] <- ifelse(full_queue$arrive_time[i] > naive_finish[i-1],     # if arrive time is after previous person finishes
                                         0,                                                            # wait time is zero
                                         full_queue$finish_time_sim[i-1] - full_queue$arrive_time[i])
}

self_queue[,c(1, 4, 7:8)]  # view datasets, relevant columns only 
```

       customer arrive_time finish_time_sim1 wait_time_sim1
    2         2          14               16              0
    3         3          17               18              0
    4         4          38               41              0
    6         6          43               46              0
    7         7          45               47              1
    8         8          52               59              0
    11       11          74               76              0
    12       12          76               86              0
    13       13          79               88              7
    14       14          88               89              0
    16       16         105              107              0
    17       17         108              110              0

``` r
full_queue[,c(1, 4, 7:8)]
```

       customer arrive_time finish_time_sim1 wait_time_sim1
    1         1          12               19              0
    5         5          39               55              0
    9         9          58               61              0
    10       10          73               84              0
    15       15          96               99              0
    18       18         113              121              0
    19       19         118              127              3
    20       20         120              129              7

``` r
  #-- Under factual conditions, average wait time was:
round(mean(table2$wait_time_factual), 2)
```

    [1] 5.95

``` r
  #-- With a self-service queue, average wait-time for potentially self-service customers went from:
round(mean(self_queue$wait_time_factual), 2)
```

    [1] 7.83

``` r
  #-- to:
round(mean(self_queue$wait_time_sim1), 2)
```

    [1] 0.67

``` r
  #-- and average wait-time for full-service customers with special needs went from:
round(mean(full_queue$wait_time_factual), 2)
```

    [1] 3.13

``` r
  #-- to:
round(mean(full_queue$wait_time_sim1), 2)
```

    [1] 1.25

``` r
  #-- Overall: 
round(mean(c(self_queue$wait_time_sim1, full_queue$wait_time_sim1)), 2)
```

    [1] 0.9

## Simulating the Two-Machine Full-Service System

Again, we begin by defining the system logic of this experiment:

-   There will be one queue for all customers, with two copy machines
    each run by an employee who can provide all services.

We do not need to work with the stratified dataset, though the indicator
variable will allow us to calculate counterfactual wait times

``` r
#-- In this simulation, we can serve two customers simultaneously.
wait_time_sim2 = rep(0, 20)
finish_time_sim2 = rep(0, 20)

 #- Customers 1 & 2 will be served immediately
finish_time_sim2[1] <- table2$arrive_time[1] + table2$service_length[1]
finish_time_sim2[2] <- table2$arrive_time[2] + table2$service_length[2]

for(i in 3:20) {
  if(table2$arrive_time[i] > min(finish_time_sim2[i-1], finish_time_sim2[i-2])) {
    wait_time_sim2[i] = 0
    finish_time_sim2[i] = table2$arrive_time[i] + table2$service_length[i]
  }
  
  if(table2$arrive_time[i] < min(finish_time_sim2[i-1], finish_time_sim2[i-2])) {
    wait_time_sim2[i] = min(finish_time_sim2[i-1], finish_time_sim2[i-2]) - table2$arrive_time[i]
    finish_time_sim2[i] = min(finish_time_sim2[i-1], finish_time_sim2[i-2]) + table2$service_length[i]
  }
}


#-- Under factual conditions, average wait time was:
round(mean(table2$wait_time_factual), 2)
```

    [1] 5.95

``` r
#-- With two full-service queues, average wait time for all customers becomes: 
round(mean(wait_time_sim2), 2)
```

    [1] 0.1

``` r
round(mean(wait_time_sim2[table2$full_service == 0]), 2)
```

    [1] 0.08

``` r
round(mean(wait_time_sim2[table2$full_service == 1]), 2)
```

    [1] 0.12

### Discussion

From these simulations we can see that both systems are a significant
improvement to the one machine + one clerk full-service system, with the
two-clerk full-service system providing the shortest wait times. The
client would need to weigh this additional improvement against the costs
of retaining a second employee, however.
