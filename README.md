SGE Basic Usage

1. What is a batch system?

Any HPC (or HTC) system — usually a cluster of machines — needs a means of sharing computational resources fairly between users; without, there would be anarchy. Batch-queueing systems — usually abbreviated to simply batch systems — are intended to do this.

All batch systems have at least these features:

- a scheduler for allocating resources (CPUs!) to jobs and for prioritising jobs;  
- one or more queues to which jobs are submitted — each queue might be configured for a particular type of job, for example, serial or parallel jobs, long or short jobs, or those requiring particularly high memory.  

To use a batch system, each computational job must be defined: the program to run is specified, with any input and output files, arguments, and the required environment (e.g., working directory, PATH and perhaps LD_LIBRARY_PATH). The defined job is then submitted to the batch system, usually to a specified queue.

Traditionally, jobs must be run non-interactively, i.e., in batch mode, though the ability to run queued, interactive jobs may be provided. (Interactive jobs may be run on RCS clusters only where specified in the user documentation.)

On RCS-run HPC clusters, all computationally-intensive activity should be run under the documented batch-queueing system — any computationally-intensive process running outside of the batch system will be killed by the system administrator.

Many batch systems are available, including NQS, LSF, Sun Grid Engine — SGE — and PBS (and, for HTC, Condor). SGE is the batch system used on Mace01, Man2e and Redqueen.

2. SGE Overview

Sun Grid Engine is so-named because it is the engine of Sun Grid, an on-demand grid computing service operated by Sun Microsystems. However, SGE is otherwise (in the opinion of the author) a misnomer — SGE is a batch-queueing system; it is not grid middleware.

SGE is an open-source community effort sponsored by Sun Microsystems and hosted by CollabNet at gridengine.sunsource.net; it is free (as in beer) to download and use. Some features of SGE are introduced in this section, below.

- Scheduler, Queues and Slots  
SGE includes both a scheduler for allocating resources (CPUs!) to computational jobs and a queueing mechanism. Each queue is associated with a number of slots: one computational process runs in each slot; each compute node in the HPC cluster provides one or more slots.
- Parallel Environments  
For most parallel jobs, including those using OpenMP and MPI (e.g., OpenMPI or MPICH), and parallel programs such as Fluent and Star-CD, an SGE Parallel Environment must be specified. This PE acts as glue ensuring that SGE and the parallel, i.e., multi-process, program play nicely together. (More details are given in the dedicated section, below [Section ]).
More advanced features — if you are new to HPC clusters and batch systems, you can probably skip these:

Job Arrays  
Job Checkpointing  
Advance Reservation  

3. Setting Up Your Environment

To use SGE correctly, you will need to ensure your environment is set up correctly: your PATH, LD_LIBRARY_PATH and MANPATH all need values appending (or prepending) to them; in addition some SGE-specific environment variables need to be set. This simplest way to do this is to source the provided settings.sh script (assuming you use BASH as your shell). On Mace01

    source /usr/local/sge60u8/default/common/settings.sh  

or on Man2 and Redqueen  

    source /usr/local/sge_v6.2/default/common/settings.sh  

4. A First SGE Job

Submission of a job to SGE is done by using the qsub command, for example

    prompt> qsub hostname-date.sh  

where hostname-date.sh is a simple BASH script which defines the job:

!/bin/bash

-- SGE options :

    #$ -S /bin/bash  
    #$ -cwd  
    #$ -q serial.q

-- the commands to be executed (programs to be run) :

    /bin/hostname  
    /bin/date  
    /bin/sleep 60  

This script is explained in detail below.

1. The First Line: #!/bin/bash and #$ -S /bin/bash  

These two lines tell SGE to run the script using the BASH command-line interpreter/shell — see below for details [Section 7.]  

2. Output and Error Messages from the Job  

Output from the job, once finished, will appear in a file called hostname-date.sh.o<job-ID>, for example, hostname-date.sh.o2590, viz,  
  comp007  
  Mon Apr 20 16:03:11 BST 2009  
Any error messages will appear in hostname-date.sh.e<job-ID> — there should be none in this case.  

3. Comments  

Any line beginning with hash symbol, #, is a comment; it will not be executed, i.e., does not form part of the job run.  

4. SGE Options: -cwd and -serial.q  

Any line beginning with hash-dollar, i.e., #$, is a special comment which is understood by SGE to specify something about how or where the job is run. In this case we specify the directory in which the job is to be run (# -cwd) and the queue which the job will join (# -q serial.q).  

5. Commands to be Executed

Any line which does NOT begin with a hash symbol, #, is executed just as if it were entered manually at the command line. In this case /bin/hostname, /bin/date and /bin/sleep are executed, one after another.

5. Cluster Queues, Queue Instances and Slots

When submitting a job, a SGE queue should be specified. This is a cluster-queue; it will be intended for a particular type of job (e.g., serial or parallel — OpenMP or MPI; short or long; or those requiring particularly large amounts of memory. To list the cluster-queues available on a system enter qconf -sql (show queue list) at the command line, for example

    prompt> qconf -sql
    
    openmp.q  
    parallel-16.q  
    parallel.q  
    serial-bf.q  
    serial-test.q  
    serial.q  
    serialfat.q  

Attributes of each cluster-queue can be found by using qconf, again, for example

    prompt> qconf -sq serial.q
    
    qname                 serial.q  
    hostlist              @serial  
    .                     .  
    slots                 4  
    tmpdir                /tmp  
    shell                 /bin/csh  
    .                     .  
    shell_start_mode      unix_behavior  
    .                     .  
    .                     .  

Each cluster-queue has instances on one or more compute-nodes in the cluster. Each instance "contains" one or more slots — usually the same number as there are CPUs (more accurately, CPU cores) on a compute node.

Example

An example should help make things clear. Suppose we have a cluster on which there are four compute nodes, cat1, cat2, cat3 and cat4. To get a complete listing of queue instances, enter qstat -f:

    prompt> qstat -f
    
    queuename              qtype resv/used/tot. load_avg arch          states
      
    serial-fat.q@cat3      BIP   0/0/8          0.01     lx24-amd64    
      
    serial.q@cat4          BIP   0/0/8          0.00     lx24-amd64    
      
    parallel.q@cat1        BIP   0/8/8          8.12     lx24-amd64    
      
    parallel.q@cat2        BIP   0/8/8          8.19     lx24-amd64    

The above listing shows that:

- on cat1 and cat2 there are instances of the cluster-queue parallel.q — eight slots each;
- on cat3 there is an instance of the cluster-queue serial-fat.q — again, eight slots;
- on cat4 there is an instace of serial.q — eight slots.

6. Commonly-Used SGE Qsub Options

There are many SGE options which can be specified in a qsub file (using the #$ syntax); these are all described in the manual page (man qsub). The most commonly used options are briefly described below.

    -cwd  
    Execute the job from the current (working) directory — the directory from which the qsub command is issued. If this option is not present, the job will be executed the user's home directory.  
    
    -j y  
    Merge the standard error stream into the standard output stream, i.e., job output and error messages are sent to the same file, rather than different files.
    -q
    Specify the queue to which a job is sent. (Use the command qconf -sql to list all available queues.)
    -pe
    Specify the SGE parallel-environment to which a job is sent — see the section on parallel jobs [Section ].
    -S bash
    See below [Section 7.]

7. Job Shell

Every job submitted to SGE runs under the specified commandline-interpreter aka shell (for example BASH or CSH). The syntax of the qsub script depends on this shell.

New users should read (only) the Short Version. More experienced Linux and HPC users may wish to read the Long Version.

Short Version  
There are multiple ways by which SGE determines the shell under which a job is run, i.e., the command interpreter for the script. The default shell on almost all current Linux systems is BASH, so a simple, safe policy is to ensure that #!/bin/bash appears as the first line in the script and that #$ -S bash appears below.

    #!/bin/bash
    
    #$ -S bash
    
    ...rest of script here...

This policy should ensure that all jobs run under the BASH shell on all HPC systems using SGE as the batch system. 

Long Version  
There are multiple ways by which SGE determines the shell under which a job is run:

- If the queue attribute shell_start_mode is set equal to unix_behavior, then the first line, e.g, #!/bin/bash determines the shell.   
- If shell_start_mode is set equal to posix_compliant then the shell is determined by:
the -S option, if specified, for example #$ -S /bin/bash;
otherwise, the queue attribute shell is used — this is set to /bin/csh default;
and if none of the above are defined, the SGE default shell is used, again /bin/csh.  
- All queues on Mace01, Man2e and Redqueen have shell_start_mode set equal to unix_behaviour.

8. Monitoring Jobs

When submitting jobs to the batch system, one may wish to know:

- Are my jobs running?  
- What other jobs are running?  
- Which queues are busy?  
- When will my job run?  

The answers to these questions (or, at least, an indication of the answers) is provided by output from the qstat command.  

With older versions of SGE (e.g., v6.0), qstat lists all jobs belonging to all users by default; on newer versions of SGE (e.g., v6.2) only a user's own jobs are listed — in this case, to list all jobs, use

    qstat -u "*"

N.B. The double-quotes around the asterisk.  

Example output:

    prompt> qstat -u "*"
    
    job-ID  prior  name        user     state submit/start at      queue              slots
      
    1807  0.50500  selfeats_a  mcaieas2   r   03/24/2009 22:28:53  serial.q@comp007       1
     1808  0.50500  selfeats_4  mcaieas2   r   03/24/2009 22:35:08  serial.q@comp011       1
    1873  0.50500  runEpi.sh   mcbicjb2   r   03/26/2009 14:52:30  serial.q@comp008       1
    1945  0.50500  STDIN       mbexfbt2   r   03/28/2009 18:31:00  serial.q@comp004       1
    2405  0.50500  new.txt     mjkieak3   r   04/12/2009 18:51:46  serialfat.q@comp002    1
    2441  0.52500  fluent      mbgbfjz2   r   04/14/2009 17:38:18  parallel.q@comp022     4
    2456  0.52500  fluent      mbgbfjz2   r   04/15/2009 08:28:03  parallel.q@comp014     4
    2460  0.50500  runmatn1r1  mchifja2   r   04/15/2009 13:07:25  serial.q@comp011       1
    2469  0.50500  STDIN       mbexfbt2   r   04/15/2009 17:46:10  serial.q@comp013       1
     2481  0.55167  fluent      mcjiych3   r   04/15/2009 22:50:55  parallel-16.q@punch    8
    2483  0.55167  fluent      mcjiych3   r   04/15/2009 22:56:40  parallel-16.q@punch    8
    2512  0.60500  ch_flow_le  mbgnfsr2   r   04/16/2009 17:06:10  parallel.q@comp020    16
    2513  0.60500  ch_flow_le  mbgnfsr2   r   04/16/2009 17:17:10  parallel.q@comp019    16
    2536  0.50500  ge.qsub     msysssp4   r   04/17/2009 11:15:08  serial.q@comp009       1
    2514  0.60500  ch_flow_le  mbgnfsr2   qw  04/16/2009 17:18:34                        16
    2515  0.60500  ch_flow_le  mbgnfsr2   qw  04/16/2009 17:21:49                        16

This shows:

- There are several jobs running — state is equal to r — and two queued and waiting their turn to run — state is equal to qw.  
- The (unique) job-ID of each job, together with its name and the owner (user) of the job; the date and time at which it was submitted (for waiting jobs) or started running, the queue in which the job is running; and the host on which the used slots are located plus the number of slots used.  
- N.B. For MPI-based jobs, the host listed in the queue column is that on which the Rank 0 process is running; other processes associated with this job may be running on other hosts — to get a complete listing of activity in each slot on each host, use qstat -u "*" -f.  

Example output:  

    prompt\> qstat -u \"*" -f
    
    queuename   qtype resv/used/tot. load_avg arch          states
    
    parallel-16.q@judy  BIP   0/8/16    6.92    lx24-amd64  2632 0.58278 fluent     mbgnfae2     r     04/22/2009   22:48:39     8        
     
    parallel-16.q@punch BIP   0/14/16   11.77    lx24-amd64    2585 0.6050  fluent     mbgnfae2     r     04/20/2009 12:52:56    10 2622 0.53833 fluent     mbgbfjz2     r     04/22/2009 09:46:46     4        
     
    parallel.q@comp014  BIP   0/4/4 2.99     lx24-amd64 2595 0.58278 pk9_PoF_ke mcji2ak3     r     04/21/2009 12:06:11     4        
                        .

9. Deleting Jobs

On occasion it may be necessary to remove a waiting job from the queue, or to stop a running job (and remove it from the queue). This is done using qdel <job-ID>. For example

    qdel 1807 

deletes the first job in the first "qstat" listing above.

10. A Typical Serial Job

The following qsub script is intended be a "real world" example. It includes commands which will give an estimate of the job length, and also the name of the host on which it ran. Further explanation is given in the script's comments.

    #!/bin/bash
    
    #$ -S /bin/bash
    #$ -cwd
    #$ -q serial.q

  

    echo " "
    echo "Job ran on and started at: "
    /bin/hostname
    /bin/date
    echo " "
    
    export PATH=/bin:/usr/bin:/software/matlab/bin
    matlab < myinput.m
    echo " "
    echo "Job finished at: "
    /bin/date
    
     ## Set SGE options:
    
      # -- ensure BASH is used, no matter what the queue config;
      # -- run the job in the current-working-directory;
      # -- submit the job to the "serial.q" queue;
    
      # ...ensure matlab is on our PATH...
      
      # ...run matlab with input file "myinput.m" --- N.B. "myinput.m" must be in the current-working-directory...

SGE  常见命令

1. qsub 提交任务

    -cwd	 #从当前工作路径运行作业
    -wd working_dir	#定义工作目录
    -o  path	定义标准输出文件路径、文件名
    -e  path	#定义标准错误输出文件路径、文件名
    -j y[es]|n[o]	#定义作业的标准错误输出是否写入到输出文件中
    -now y[es]|n[o]	#立即执行作业
    -a date_time	#作业开始运行时间
    -b y[es]|n[o]	#指定运行程序是二进制文件还是脚本文件，默认n
    -m b|e|a|s|n	#定义邮件发送规则。
     b：作业开始时发送。e：作业结束时发送。a：作业失败时发送 s：作业挂起时发送。n：不发送
    -M user[@host]	#定义邮件地址
    -l resource=value	#表明作业运行所需要的资源。-l vf=2g,p=1  #vf内存简写，p线程数简写；
    #资源可分开写，可写全称，单位可大小写，如-l virtual_free=2G -l num_proc=1。
    -N job_name	#重命名作业名
    -q queue_name	#定义作业运行队列
    -S shell_path	#指定运行Shell环境
    -P project_name	#定义项目名称，前提是存在该项目
    -p priority	#定义优先级，-1023 到 1024 , 默认值0
    -r y[es]|n[o]	#定义作业失败后是否重新运行 -v variable	#定义环境变量
    -dl date_time	#定义作业到期时间，在作业到期时间之前，作业的优先级会逐步提高，直到管理员指定的最高级别。

2. qstat查看任务状态

    qstat -u username  查看某个用户的任务
    
    qstat -u \* 查看所有用户的任务
    
    qstat -j jobID 查看某个任务的详细信息
    
    qstat -f 查看用户自己在每个节点的任务情况
    
    qstat -q all.q -u \* 查看某个队列下所有任务
    
    qstat -q all.q@node1 -u \* 查看某个队列的某一节点下所有任务
    
    qdel 删除任务
    qdel [ -f ]  [ -help ] [-u wc_user_list] [ wc_job_range_list ] [ -t task_id_range ]
    
    qdel job_id 删除job
    
    qdel -u usrname 删除用户的所有任务

3. 任务的状态

    qw #等待状态
    hqw #任务挂起等待中，待依赖的任务完成后执行
    Eqw #投递任务出错
    r #任务正在运行
    s #暂时挂起
    dr #节点挂了之后，删除任务就会出现这个状态，只有节点重启之后，任务才会消失

4. 挂起/恢复任务

qhold命令：挂起qw的任务

    qhold jobid
    qhold -u \*
    
    qrls jobid #恢复

qmod命令：挂起running中的任务

    qmod -sj jobid
    qmod -usj jobid #恢复

如果未提交到SGE系统，直接运行的命令用kill -STOP pid 挂起，用kill -CONT pid恢复。

