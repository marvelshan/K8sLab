# 一

1. Which command would you use to create a tar archive called test.tar out of 3 files, called test1, test2 and test3?

   A) tar -fc test.tar test1 test2 test3

   B) tar -cf test.tar test1 test2 test3

   C) tar -xvf test.tar test1 test2 test3

   D) tar -f test.tar test1 test2 test3

2. You have connected a new HDD to a Linux machine but the device might not have been properly detected on boot. which command would be helpful in checking if the operating system has detected the HDD

   A) dmseg

   B) dd

   C) du

   D) dc

3. Which set contains only tables from iptables?

   A) filter, nat, mangle, raw, security

   B) Input, Output, prerouting, postrouting, forward

   C) accept, reject, drop, redirect

   D) Input, Output, nat, filter

4. One of the employees is leaving your company. Which command would you issue (as root) to lock local access for this user?

   A: passwd -l jdoe

   B: usermod -l jdoe

   C: login -l jdoe

5. You have received information that there has been a potential break-in attempt to one of the servers (Server1). What command chain can be used to get a sorted list of all IP addresses with failed authentication attempts from the SSH authentication logs, along with the usernames that were tried? Small caveat: due to a bug, one of your applications might occasionally try to log in to Server1 as user loki. Ignore any such attempts.

   Logfile (Ubuntu 16): `/var/log/auth.log`

   Sample matching logfile line: Jun 01 10:05:16 server1 sshd[12345]: Invalid user admin from 192.168.134.245

   Example result:

   ```
   192.168.134.201 admin
   192.168.134.201 administrator
   192.168.134.201 administrator
   192.168.134.201 superuser
   192.168.134.202 root
   192.168.134.202 www
   192.168.134.245 admin
   192.168.134.245 admin1
   192.168.134.245 root
   ```

   A: awk '/Invalid user/ { print $10, $8 }' /var/log/auth.log | grep --invert-match loki | uniq -c | sort

   B: awk '/Invalid user/ /var/log/auth.log | awk '{ print $10, $8 }' | sort -a | uniq | grep -v loki

   C: cat /var/log/auth.log | grep -v loki | cut -d'-' -f R9,8 | grep 'Invalid user' | sort | uniq

   D: grep -v loki /var/log/auth.log | cut --delimiters=, --fields=10,0 --only-delimited | sort | uniq | grep 'Invalid user'

6. You run the attached command, which is followed by the shown output. What's the most probable cause? Command: ping 8.8.8.8 -c 3 -i 3 Output:

   ```
   PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
   From 10.0.2.2 icmp_seq=1 Time to live exceeded
   From 10.0.2.2 icmp_seq=2 Time to live exceeded
   From 10.0.2.2 icmp_seq=3 Time to live exceeded
   --- 8.8.8.8 ping statistics ---
   3 packets transmitted, 0 received, +3 errors, 100% packet loss, time 5998ms
   ```

   A: Host with IP 8.8.8.8 is down and you should inform its owner about this.

   B: Something is wrong with your networking card.

   C: Option "-i 3" sets the time to live value too low, you should try to increase it and see what happens.

   D: This could be a network issue and additional tests need to be ran.

7. Given the attached commands, what would the output be? Assume you are working in a basic terminal.

   ```text

   A=( " bash arrays are cool" )

   echo ${A[0]}
   ```

   Options:

   A: a double quote mark ''

   B: 1 space character

   C: word bash

   D: phrase bash arrays are cool

   E: letter B

8. Which command can be used to disable the network interface eth0?

   Options:

   A: ip link set eth0 down

   B: ethtool -s eth0 mdix off

   C: ifconfig eth0 --down

   D: if down eth0

9. The attached script is placed in the user's home directory, and is then run from there with the shown command. What would its outcome be?

   Command: ./script www.google.com 5

   Script:

   ```bash

   #!/bin/bashPATH="./"LOGNAME="logfile"if [{ -n $1 }] && [{ -n $2 }]; then

   PING_ADDRESS=$1

   REPETITIONS=$2

   ping $PING_ADDRESS -c $REPETITIONS >> ${PATH}$(LOGNAME) 2>/dev/nullelse

   echo "Missing arguments!"}
   ```

   Options:

   A: www.google.com address would get pinged 5 times, and outcome of the command would get saved in file called 'logfile' located in users current working directory.

   B: ping: command not found message would get saved in file called 'logfile' located in users current working directory.

   C: No output would get generated (neither to standard output nor to the logfile), but www.google.com address would still be pinged.

   D: No output would get generated (neither to standard output nor to the logfile) and www.google.com address would not get pinged.

10. One of the servers is having problems: almost all of the disk space in the root partition is being used up and you have no idea why! what command would help you to analyze the disk contents and identify the cause of the lost space ?

    a) du -ahd1 /

    b) df -ah /

    c) dc -e size /

    d) ls -al /

# 二

1. Select a Dockerfile command required to execute a build:

   A: MAINTAINER

   B: FROM

   C: ARG

   D: FROM but only when using Docker multistage

2. The command for listing all containers in Docker is:

   A: docker container ls -a

   B: docker ls --all

   C: docker show --all

   D: docker ps -aux

3. How to connect from one container to another within the same Docker network?

   A: You can use Docker API to ask the hosting system for IPs of the other containers in order to reach them via both UDP and TCP

   B: You can use container's name to reach it via TCP

   C: You can use container's name to reach it via UDP

   D: You can use container's name to reach it via TCP or UDP

4. The command for transferring files from a host to a Docker container is:

   A: docker cp foo.txt mycontainer:/foo.txt

   B: docker mv foo.txt mycontainer:/foo.txt

   C: rsync foo.txt mycontainer:/foo.txt

   D: docker sync -R foo.txt mycontainer:/foo.txt

5. What is the purpose of using EXPOSE command in Dockerfile?

   A: Exposing an image on another machine

   B: Exposing volumes from one container to another

   C: Exposing a specified port on which application is listening in the runtime

   D: Exposing an image for specified user groups

6. What is a Docker hub?

   A: It's a service for connecting multiple containers before deployment

   B: It's a registry service which allows users to download, share and manage Docker images

   C: It's a tool for managing volumes among containers

   D: It's a registry of all built and deployed images

7. How to start a terminal session of an already running container?

   A: It's impossible to enter a running container if it wasn't started in the interactive mode (-it).

   B: docker container exec <container_id> /bin/bash

   C: docker run -it <container_id> /bin/bash

   D: docker exec -it <container_id> /bin/bash

8. How to disable a specific container? Choose one wrong answer:

   A: docker stop <container_id>

   B: docker kill <container_id>

   C: docker rm <container_id>

   D: docker container pause <container_id>

9. What is the correct way to pass an argument with a value during Docker build phase?

   A: Adding --build-arg [KEY=VALUE,] flag to the docker build command.

   B: Docker does not use any build arguments.

   C: Setting it as an environment variable available on the machine executing a given build.

   D: Defining it in a .env file in the root directory.

10. What is the difference between EXPOSE and PUBLISH commands?

    A: When not specified neither EXPOSE nor -p, the service in the container will only be accessible from outside of the container itself.

    B: If you PUBLISH a port, the service in the container is not accessible from outside Docker, but from within other Docker containers.

    C: If you EXPOSE and -p <port>, the service in the container is accessible from anywhere, even outside Docker.

    D: When EXPOSE command is defined, ports can be accessible for the host.

# 三

1. What is a state in Terraform?

   A: State is a JSON file which keeps track of metadata and mappings between resources and remote objects.

   B: State is an attribute of a Terraform resource describing its current lifecycle; for example, 'destroyed'.

   C: State is an immutable type of Terraform resource.

   D: State is an extension to Terraform which allows external providers to be used.

2. What is the 'taint' operation used for in Terraform?

   A: It marks resources to be destroyed and recreated on the next execution of apply.

   B: It allows properties of the resource to be changed directly in the Terraform state without using the apply command.

   C: It rewrites all Terraform configuration files to a canonical format.

   D: It is used to convert HCL configuration files into JSON.

3. When is State Locking performed in Terraform?

   A: Locking happens automatically on any request to edit or read the state, and it blocks 'write' operations only.

   B: Locking happens automatically on any request to edit the state file, and it blocks all other I/O operations.

   C: Locking happens automatically on any request to edit or read the state file, and it blocks all other I/O operations.

   D: Locking happens automatically on any request to edit the state file, and it blocks 'write' operations only.

4. Select one false statement about Terraform imports:

   A: Importing may result in a "complex import" where multiple resources are imported at once and extra changes in the Terraform code are required to prevent their destruction.

   B: Importing a resource does not require a state to be locked.

   C: Only some types of resources can be imported, and this depends on the specific implementation in the provider.

   D: It is possible to import resources with the "count" or "for_each" attribute defined in the configuration file.

5. Select one false statement about Terraform's communication with providers:

   A: Terraform can use client pull network communication where the server provides the configuration from Terraform.

   B: Communication between Terraform and its providers is JSON based.

   C: The plugin used for communication with a specific provider must implement a golang abstraction layer which allows Terraform to communicate with its resources.

   D: Terraform can use server push network communication where Terraform pushes the configuration to the provider.

6. Choose one sentence which does not compare data source and resource correctly:

   A: Both resources and data sources can use count/for_each to be created multiple times.

   B: In order to access any of the data source objects, both data source and resource are to be accessed from a namespace called 'data' while resources may be reached directly via their types and names.

   C: The data source is an immutable object, whereas resources may be modified.

   D: Both resources and data sources can be imported from the Terraform CLI using import.

7. Select a false statement about configuring providers in Terraform:

   A: Each provider type can have only one configuration.

   B: A resource can explicitly define which provider will manage its configuration.

   C: Providers can use attributes of other resources for configuration.

   D: Provider configuration can be defined with variables or local values.

8. What is a target in Terraform?

   A: A target is a keyword for a module configuration which defines its behaviour.

   B: A target is a resource on which 'plan', 'apply' or 'destroy' commands will be executed.

   C: There isn't anything called a target in Terraform.

   D: It is an interface for a provider whose API is being used by Terraform.

9. Choose one false sentence about output in Terraform:

   A: Output values may be used to expose data outside of a Terraform module.

   B: It is possible to prevent sensitive data of outputs from being displayed in the CLI.

   C: Outputs are stored in the Terraform state.

   D: Only string values may be used as output.

10. What type of provisioning is not supported by Terraform?

    A: Executing commands on every planned application.

    B: Creating files.

    C: Defining external provisioners like puppet/chef.

    D: Executing commands over SSH.

# 四

1. The following questions concern MySQL. To create user andrew with password passwd123 on localhost one should write:

   A: CREATE USER 'andrew'@'localhost' IDENTIFIED BY 'passwd123';

   B: CREATE USER 'andrew'@'localhost' IDENTIFIED WITH 'passwd123';

   C: CREATE USER 'andrew'@'localhost' IDENTIFIED BY PASSWORD 'passwd123';

   D: CREATE USER 'andrew'@'localhost' SET PASSWORD 'passwd123';

2. To allow a particular user to remove a particular table in the current database, one should grant which privilege?

   A: DELETE

   B: TRUNCATE

   C: DROP

   D: It is not possible to grant a specific privilege only to remove a table in MySQL.

   E: It is not possible to specify privileges per table in MySQL.

3. To safely clear a table with foreign key constraints applied to it (together with all the records matched in other tables), one should:

   A: Create a trigger ON CASCADE, also removing the records bound by constraints.

   B: Disable the foreign key check, TRUNCATE the table and then re-enable foreign key check.

   C: Perform a DELETE query on that table without a WHERE clause.

   D: None of the answers above is correct.

4. If we have tables threads and posts (with foreign keys to the threads table), and we want to get a list of all threads containing more than five posts, we could:

   A: Use GROUP BY, JOIN, COUNT and WHERE.

   B: Use GROUP BY, JOIN, COUNT and HAVING.

   C: Use JOIN, SUM and WHERE.

   D: Use JOIN, ORDER BY and LIMIT.

5. What does CREATE TABLE table2 SELECT \* FROM table1 do?

   A: Creates a new table called table2 and copies data from table1.

   B: Creates a new, empty table called table2 with the same structure as table1.

   C: Creates a view called table2 based on table1.

   D: Produces a syntax error.

6. If one creates ENUM('abc', 'xyz', '123', 'aaa', 'def') and selects records in descending order from this enum, then:

   A: Records with abc will be first.

   B: Records with xyz will be first.

   C: Records with 123 will be first.

   D: Records with aaa will be first.

   E: Records with def will be first.

7. If we have a table that stores point coordinates in 2D with two columns, x and y, and we want to get a non-duplicated list of all points in the table, we should use:

   A: SELECT DISTINCT [x, y] FROM ...

   B: SELECT DISTINCT x, DISTINCT y FROM ...

   C: SELECT x, y FROM ... GROUP BY x, y

   D: SELECT DISTINCT (x, y) FROM ...

   E: More than one answer is correct.

8. If more than one column is returned by a subquery when using the IN operator in the main query, then what is used as the result of the subquery?

   A: There is a syntax error in such a statement; only one column should be returned by the subquery in this case.

   B: Values of the primary key(s).

   C: Values of the first returned column.

   D: Values of the last returned column.

   E: All columns are used as tuples, and one can provide a column names tuple as the left operand of IN, as long as it has the same arity as the returned tuples.

9. If one creates a multicolumn index (col1, col2, col3), then what types of queries will it use?

   A: SELECT ... WHERE col2 = ... AND col1 = ...

   B: SELECT ... WHERE col2 = ...

   C: None.

   D: Both.

10. If one wants to select all records with the value NULL from column col, one could use:

    A: WHERE col = NULL

    B: WHERE col <=> NULL

    C: WHERE col IFNULL(col)

    D: WHERE (NOT col) IS NOT NULL

    E: More than one answer is correct.
