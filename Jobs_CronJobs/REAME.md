A Job creates one or more Pods and ensures that a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created.

To run the job: kubectl apply -f pi-job.yaml

job.batch/pi-job created

Clean up and delete the Job:
When a Job completes, the Job stops creating Pods. The Job API object is not removed when it completes, which allows you to view its status. Pods created by the Job are not deleted, but they are terminated. Retention of the Pods allows you to view their logs and to interact with them.


To Check the status of the job : kubectl describe job pi-job

To get the jobs running in the cluster: kubectl get jobs
O/P
NAME     COMPLETIONS   DURATION   AGE
pi-job   1/1           34s        6m8s


To retrive the pods created by the job: kubectl get pods
O/P
NAME           READY   STATUS      RESTARTS   AGE
pi-job-bj4bs   0/1     Completed   0          6m15s 

To retrive the log file of the pod that was executed by the job: kubectl logs pi-job-bj4bs
O/P
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901

that is 1999 decimal points 

To delete the Job, execute the following command: kubectl delete job pi-job

CRON JOBS:

This CronJob deploys a new container every minute that prints the time, date and "Hello, World!".
Note:
CronJobs use the required schedule field, which accepts a time in the Unix standard crontab format. All CronJob times are in UTC:

The first value indicates the minute (between 0 and 59).
The second value indicates the hour (between 0 and 23).
The third value indicates the day of the month (between 1 and 31).
The fourth value indicates the month (between 1 and 12).
The fifth value indicates the day of the week (between 0 and 6).
The schedule field also accepts * and ? as wildcard values. Combining / with ranges specifies that the task should repeat at a regular interval. In the example, */1 * * * * indicates that the task should repeat every minute of every day of every month.


To create coron job: kubectl apply -f hello-cronjob.yaml
O/P
cronjob.batch/hello created

Get the job name: kubectl get jobs
O/P
NAME               COMPLETIONS   DURATION   AGE
hello-1570454400   1/1           2s         45s

Describe job: kubectl describe job hello-1570454400 

Get the created pod name from the above job. In my case the pod name was hello-1570454460-445lw

View the output of the Job by querying the logs for the Pod. kubectl logs hello-1570454520-b6gsm

To view all job resources in your cluster, including all of the Pods created by the CronJob which have completed, execute the following command:
kubectl get jobs
O/P
NAME               COMPLETIONS   DURATION   AGE
hello-1570454580   1/1           2s         2m41s
hello-1570454640   1/1           2s         101s
hello-1570454700   1/1           1s         40s


Clean up and delete the Cron Job:
In order to stop the CronJob and clean up the Jobs associated with it you must delete the CronJob.

To delete all these jobs, execute the following command:
kubectl delete cronjob hello
O/P
cronjob.batch "hello" deleted

To verify that the jobs were deleted, execute the following command:
kubectl get jobs
O/P
No resources found.



