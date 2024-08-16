# INDIS-2024

I. To reproduce our experiment it's necessary to have at least one source node and one or more destination hosts and follow the following steps:
1) Build iperf3 with the patches used in the article on both source and destination nodes
 ```
sudo apt install -y build-essential git libtool
git clone -b indis https://github.com/marcosfsch/iperf
./bootstrap.sh; ./configure; make
```
2) Install the testing harness tool on the source node only
```
sudo apt install -y net-tools python3-pip jq python3-ping3 python3-pika python3-requests python3-yaml python3-tabulate
git clone -b bbr3-testing https://github.com/esnet/testing-harness
cd testing-harness
sudo pip install .
cd testing-harness
```
3) Create a hostlist.csv with the list of destination hosts/IPs that will be used in each test, i.e.:
```
hostname,rtt,NIC_speed
10.3.54.2,0.2ms,100Gbps
10.3.56.2,25ms,100Gbps
10.3.60.2,54ms,100Gbps
10.3.58.9,104ms,100Gbps
```
4) Configure passwordless ssh access from the source node to the destination nodes using ssh keys, so that the harness tool can run remote commands
5) Choose a .ini configuration file used in the paper and do the proper customization for your environment, i.e. https://github.com/marcosfsch/INDIS-2024/blob/main/AmLight/ubuntu22-6.8/iperf3-final-set-bigtcp.ini
Including, iperf3 path and CPU cores used for affinity (-A) and to be monitored by mpstat
6) Run the test harnesstool
```
nohup ./collect.py -j iperf3-final-set-bigtcp.ini -H hostlist.csv -i enp177s0f0np0 -l tuning-test.log -o ubuntu22-6.8
```
Where -j is the configuration file, -H the destination hostlist -i the NIC that will be used in the tests, -l the logfile and -o the path where the results will be saved
7) Create a summary of the results
```
cd ubuntu22-6.8; ../utils/summarize_all.py | tee summary.txt
```
PS: To get consistent results use the same tuning recommendations as in Section III of the paper

II. The suggested configuration file will take around 10h. (60 seconds per test, times 10 runs, times 14 different setups, times 4 destination IPs)

III. The summary.txt file provides a high level overview of the results including two sessions, the first show detailed information for each test in the same order they were executed, including Test name, destination Host IP, number of runs, mean/min/max/stdev throughput considering all runs, mean retransmission, sender and receiver CPU utilization. I.e.:
```
Test iperf3_1stream_bigtcp to Host: 10.3.54.2   (num tests: 10)
       Throughput:   Mean: 60.0 Gbps   Min: 57.2 Gbps  Max: 61.3 Gbps   STDEV: 1.1   Retr: 344
     Sender CPU 1:   sys :  0.0   idle: 100.0   Total:0.0
     Sender CPU 2:   usr :  0.9   sys : 73.8   soft:  3.8   steal:  0.0   idle: 21.4   Total:78.5
   Receiver CPU 1:   sys :  0.2   soft:  1.4   steal:  0.0   idle: 98.4   Total:1.6
   Receiver CPU 2:   usr :  1.1   sys : 97.2   soft:  1.7   steal:  0.0   idle:  0.0   Total:100.0
```

The second section presents a summary of all the results per destination sorted by average throughput in descending order. I.e:
```
IP Address: 10.3.54.2
    Test Name: iperf3_4streams_zerocopy, Ave Throughput: 99.0 Gbps (std: 0.0, min: 99.0, max: 99.0),  Retr: 24
    Test Name: iperf3_8streams_zerocopy, Ave Throughput: 99.0 Gbps (std: 0.0, min: 99.0, max: 99.0),  Retr: 54
…
    Test Name: iperf3_1stream_bigtcp_pacing50, Ave Throughput: 50.0 Gbps (std: 0.0, min: 50.0, max: 50.0),  Retr: 0
IP Address: 10.3.56.2
    Test Name: iperf3_4streams_zerocopy_fq20, Ave Throughput: 71.7 Gbps (std: 16.1, min: 26.9, max: 80.0),  Retr: 1168
…
```

IV. The steps to graph the results were manually done for the submitted  paper. The authors acknowledge the importance of automating this steps but they would need to be done for a revised version of the paper
