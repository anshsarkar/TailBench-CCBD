# tailbench-ccbd

This repository has 5 out of 8 tailbench applications building and running successfully. These applications are `img-dnn`, `masstree`, `silo`, `specjbb` and `sphinx`.

# Applications

TailBench includes eight latency-critical applications, each with its own
subdirectory:

 - `img-dnn`      : Image recognition
 - `masstree`     : Key-value store
 - `moses`        : Statistical machine translation
 - `shore`        : OLTP database (optimized for disk/ssd)
 - `silo`         : OLTP database (in-memory)
 - `specjbb`      : Java middleware
 - `sphinx`       : Speech recognition
 - `xapian`       : Online search

Please see the paper, [TailBench: A Benchmark Suite and Evaluation Methodology for Latency-Critical Applications, Kasture and Sanchez, IISWC-2016](http://people.csail.mit.edu/sanchez/papers/2016.tailbench.iiswc.pdf) for further details on these applications.

# Harness

The TailBench harness controls application execution (e.g., implementing warmup
periods, generating request traffic during measurement periods), and measures
request latencies (both service and queuing components). The harness can be set
up in one of three configurations:

 - **Networked**    : Client and application run on different machines, communicate
                  over TCP/IP
 - **Loopback**     : Client and application run on the same machine, communicate
                  over TCP/IP
 - **Integrated**   : Client and applicaion are integrated into a single process and
                  communicate over shared memory

See the TailBench paper for more details on these configurations.

Each TailBench application has two components: a server comoponent that
processes requests, and a client component that generates requests. Both
components interact with the harness via a simple C API. Further information on
the API can be found in the header files `tailbench/harness/tbench_server.h` (for the server
component) and `tailbench/harness/tbench_client.h` (for the client component). 

Application and client execution is controlled via environment variables. Some
of these are common for all three configurations, while others are specific to
some configurations. We describe the environment variables in each of these
categories below. The quantities in parentheses indicate whehter the environment
variable is used for the application or the client, and if it is only used for
some configurations (e.g. networked + loopback).  Additionally, some application
clients use additional environment variables for configuration. These are
described in the README files within application directories where applicable.

# Environment Variables

```
TBENCH_WARMUPREQS (application): Length of the warmup period in # requests. No
latency measurements are performed during this period.

TBENCH_MAXREQS (application): The total number of requests to be executed during
the measurement period (the region of interest). This count *does not* include
warmup requests.

TBENCH_MINSLEEPNS (client): The mininum length of time, in ns, for which the
client sleeps in the kernel upon encountering an idle period (i.e., when no
requests are submitted).

TBENCH_QPS (client): The average request rate (queries per second) during the
measurement period. The harness generates interarrival times using an
exponential distribution.

TBENCH_RANDSEED (client): Seed for the random number generator that generates
interarrival times.

TBENCH_CLIENT_THREADS (client, networked + loopback): The number of client
threads generating requests. The total request rate is still controlled by
TBENCH_QPS; this parameter is useful if a single client thread is overwhelmed
and is not able to meet the desired QPS.

TBENCH_SERVER (client, networked + loopback): The URL or IP address of the
server. Defaults to localhost.

TBENCH_SERVER_PORT (client, networked + loopback): The TCP/IP port used by the
server. Defaults to 8080.
```

# Running TailBench
To execute the 5 applications these steps can be followed.

## Setting up Directory Structure

Create an empty directory named `scratch` using the following command.

```bash
mkdir scratch
```

Make sure that your directory structure looks like this:

    .
    ├── tailbench                           # Directory that contains all the tailbench applications
        ├── img-dnn
        ├── masstree
        └── ...                             # etc 
    ├── tailbench.inputs                    # Dataset for tailbench (can be downloaded)
    └── scratch                             # Initially an empty directory

## Downloading TailBench Inputs

`tailbench.inputs` is the dataset that is required for testing. This dataset can be downloaded [here](http://tailbench.csail.mit.edu/tailbench.inputs.tgz). It can be downloaded using the following commands

```bash
wget http://tailbench.csail.mit.edu/tailbench.inputs.tgz
tar -xvzf tailbench.inputs.tgz
```

## Installing Dependencies

```bash
sudo apt-get install libboost-all-dev
```

```bash
sudo apt-get install openjdk-8-jdk libopencv-dev autoconf ant libtcmalloc-minimal4 swig google-perftools bzip2 libnuma-dev libjemalloc-dev libgoogle-perftools-dev libdb5.3++-dev libmysqld-dev libaio-dev uuid-dev libbz2-dev python-numpy python-scipy libgtop2-dev
```

## Updating configs

Now we go ahead and update our config file.

```bash
cd tailbench
nano configs.sh
```

Set the following environment variables as described below.

```bash
# Set this to point to the top level of the TailBench data directory
#Set to the path of the directory for tailbench datasets called tailbench.inputs
DATA_ROOT= /directoryfortailbenchinputs

# Set this to point to the top level installation directory of the Java
# Development Kit. Only needed for Specjbb
# No need to change this
JDK_PATH=/usr/lib/jvm/java-8-openjdk-amd64/

# This location is used by applications to store scratch data during execution.
# Copy the path to the scratch directory that was created earlier.
SCRATCH_DIR= /pathtoscratchdirectory
```

## Building the applications

The preferred operating system would be Ubuntu 18.04.
The applications can be run either by using a global script present in the `tailbench` directory or each application can be built and run individually by running the scripts in each application directory.  It is preferred that we run each application individually for better error handling.

Before executing any application we need to install some dependencies and make some changes to our config file.

### Building each Application

First execute the following commands to give the scripts executable access.

```bash
cd applicationname
sudo chmod +x build.sh
sudo chmod +x clean.sh
sudo chmod +x run.sh
```

Then run these commands to build the application.

```bash
sudo ./clean.sh
sudo ./build.sh
```

Once built successfully without any errors (warnings can be ignored) run the following command to run the application.

```bash
sudo ./run.sh
```

### Building all Applications

To build and run all applications

```bash
cd tailbench
sudo chmod +x build.sh
sudo chmod +x clean.sh
sudo ./clean.sh
sudo ./build.sh
```

## Parsing Results

At the end of the run, each liblat client publishes a lats.bin file, which
includes a <service time, end-to-end time> tuple for each request submitted by
the client. Note that the tuples are not guaranteed to be in the order the
requests were submitted, and therefore cannot be used to generate a time series
for request latencies. The lats.bin file contains binary data, and can be parsed
using the `parseLatencies.py` script in `tailbench/utilities`.

# Building and running

Please see BUILD-INSTRUCTIONS for instructions on how to build and execute TailBench applications.

# Note on SPECjbb

Since SPECjbb is not freely available, we do not include it in the TailBench
distribution. Instead, the specjbb directory contains a patch file
(tailbench.patch) that can be applied to a pristine copy of SPECjbb2005 to
obtain the version used in the TailBench paper.

# Acknowledgement

This project was carried out under the guidance of Dr K V Subramaniam, Professor of CSE at PES University as a part of the Center for Cloud Computing and Big Data Summer Internship Programme 2020. 