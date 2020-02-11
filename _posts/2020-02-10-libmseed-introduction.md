---
layout: post
title: "Brief Introduction of miniSEED and libmseed"
category: [seismology, seismology data format]
tags: [miniSEED, libmseed]
---

I am working on a project about seismic wave analysis these days.
For reading data containing seismic waves, it comes up to several
format, and the one I get in touch frequently is miniSEED. 

miniSEED is famous of its simplexity compared to its superset SEED,
which stands for Standard for the Exchange of Earthquake Data. The
biggest difference between them is that miniSEED only contains a
header -- to carry necessary information about the data -- and 
the values representing the amplitude of the seismic wave. [^1]
As usual, numeruous libraries support to read and write miniSEED
data in ease. For instance, `libmseed` is provided by IRIS team
to let people to maniplulate miniSEED data and create their own
binding on other programming languages.

## Why I Write This Post
1. Seismology is a fresh new area to me, and I think getting
familiar with the data maniplulating tool is a good start for
realizing what seismologists are doing.
2. Compared to the version before 3, `libmseed` made a great
number of changes in order to support the newer version standard
of miniSEED (or miniSEED 3.x). [^2] And writing this post makes 
not only some records what I have done for porting this library but also 
a brief introduction for others to use this work.

## Installation
Installation of `libmseed` will be quite easy, and it has almost
no dependencies (though strictly, `glibc`).
### Unix-like System
Type these commands will be sufficient:
```
$ make
$ sudo make install # to install libmseed so that you can link against to other programs
```
After you install the necessary files, you can choose one of the
following ways to let the runtime linker where the `libmseed` is:
1. Add `libmseed` into `LD_LIBRARY_PATH` environment variable (e.g., .bashrc file).
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/libmseed/shared/lib
```
2. Create a new `*.conf` file in `/etc/ld.so.conf.d/`: [^3]
For instance:
```
/etc/ld.so.conf.d/99local.conf
```
with just:
```
/usr/local/lib
```
After the modification you have to run:
```
$ sudo ldconfig
```

### Windows System
Type the comand instead:
```
$ nmake -f Makefile.win
```
Then add the library path to compiler setting may be work.

## Linking against to Other Programs
Assume you put `libmseed.h` in `/usr/local/include` and the shared libraries
in `/usr/loca/lib`, you can link `libmseed` in the following way for instance:
```
$ gcc -I/usr/local/include -o main main.c -L/usr/local/lib -lmseed
```

## Demo
In this part, I am going to demostrate how to read the infomation of header
and the values of each record. The demo procedures are shown as follows:
1. Clone the [repo](https://github.com/Cuda-Chen/libmseed-hello-world) written by me.
2. Compile it.
3. Run with the following command:
```
$ make
$ ./libmseed_hello_world <your miniSEED file> 
```
After running the above commands, you should get the following output (take test.mseed
provided by `libmseed` as example):
```
...
XFDSN:IU_COLA_00_L_H_1, 4, 512, 135 samples, 1 Hz, 2010,058,06:50:00.069539
XFDSN:IU_COLA_00_L_H_1, 4, 512, 188 samples, 1 Hz, 2010,058,06:52:15.069539
XFDSN:IU_COLA_00_L_H_1, 4, 512, 126 samples, 1 Hz, 2010,058,06:55:23.069541
XFDSN:IU_COLA_00_L_H_1, 4, 512, 156 samples, 1 Hz, 2010,058,06:57:29.069539
XFDSN:IU_COLA_00_L_H_1, 4, 512, 138 samples, 1 Hz, 2010,058,07:00:05.069539
XFDSN:IU_COLA_00_L_H_1, 4, 512, 132 samples, 1 Hz, 2010,058,07:02:23.069539
XFDSN:IU_COLA_00_L_H_1, 4, 512, 110 samples, 1 Hz, 2010,058,07:04:35.069539
XFDSN:IU_COLA_00_L_H_1, 4, 512, 131 samples, 1 Hz, 2010,058,07:06:25.069539
XFDSN:IU_COLA_00_L_H_1, 4, 512, 135 samples, 1 Hz, 2010,058,07:08:36.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 117 samples, 1 Hz, 2010,058,07:10:51.069539
XFDSN:IU_COLA_00_L_H_1, 4, 512, 114 samples, 1 Hz, 2010,058,07:12:48.069539
XFDSN:IU_COLA_00_L_H_1, 4, 512, 117 samples, 1 Hz, 2010,058,07:14:42.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 123 samples, 1 Hz, 2010,058,07:16:39.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 123 samples, 1 Hz, 2010,058,07:18:42.069539
XFDSN:IU_COLA_00_L_H_1, 4, 512, 142 samples, 1 Hz, 2010,058,07:20:45.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 130 samples, 1 Hz, 2010,058,07:23:07.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 127 samples, 1 Hz, 2010,058,07:25:17.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 120 samples, 1 Hz, 2010,058,07:27:24.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 118 samples, 1 Hz, 2010,058,07:29:24.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 108 samples, 1 Hz, 2010,058,07:31:22.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 107 samples, 1 Hz, 2010,058,07:33:10.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 107 samples, 1 Hz, 2010,058,07:34:57.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 110 samples, 1 Hz, 2010,058,07:36:44.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 103 samples, 1 Hz, 2010,058,07:38:34.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 103 samples, 1 Hz, 2010,058,07:40:17.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 104 samples, 1 Hz, 2010,058,07:42:00.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 105 samples, 1 Hz, 2010,058,07:43:44.069536
XFDSN:IU_COLA_00_L_H_1, 4, 512, 103 samples, 1 Hz, 2010,058,07:45:29.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 103 samples, 1 Hz, 2010,058,07:47:12.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 106 samples, 1 Hz, 2010,058,07:48:55.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 103 samples, 1 Hz, 2010,058,07:50:41.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 103 samples, 1 Hz, 2010,058,07:52:24.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 110 samples, 1 Hz, 2010,058,07:54:07.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 103 samples, 1 Hz, 2010,058,07:55:57.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 108 samples, 1 Hz, 2010,058,07:57:40.069538
XFDSN:IU_COLA_00_L_H_1, 4, 512, 32 samples, 1 Hz, 2010,058,07:59:28.069538
...
```
Execute with the `-D` option will output the values of each records:
```
$ ./libmseed_hello_world -D test.mseed
XFDSN:IU_COLA_00_L_H_1, 4, 512, 135 samples, 1 Hz, 2010,058,06:50:00.069539
   -502676     -504105     -507491     -506991     -505093     -506634  
   -507449     -506132     -504335     -503581     -504801     -502676  
   -499117     -502521     -508058     -508719     -506640     -503213  
   -501518     -503297     -504019     -503973     -505239     -505618  
   -505363     -508445     -512396     -510357     -503344     -499292  
   -500644     -499573     -497070     -496368     -494254     -495751  
   -498163     -498512     -503053     -504862     -502569     -505073  
   -508702     -510540     -510995     -509509     -510162     -509896  
   -506668     -505275     -501772     -499662     -504604     -506787  
   -508899     -511028     -506260     -506356     -507500     -502717  
   -501599     -499809     -497380     -499415     -499962     -501449  
   -506331     -508451     -507561     -505771     -504482     -504974  
   -502925     -499716     -499168     -497727     -496606     -497632  
   -498311     -501691     -506977     -508209     -509408     -510606  
   -507239     -504164     -503331     -501325     -498815     -499063  
   -500754     -501606     -502901     -505046     -506651     -506710  
   -505258     -502673     -502796     -505352     -504242     -503463  
   -505552     -506242     -505771     -504268     -503942     -504831  
   -505228     -505811     -504930     -503956     -502634     -501069  
   -503523     -505187     -502566     -502208     -502309     -499495  
   -498321     -497806     -498092     -499768     -499282     -500757  
   -506604     -509557     -508944     -510178     -509697     -505212  
   -499533     -495590     -496168  
XFDSN:IU_COLA_00_L_H_1, 4, 512, 188 samples, 1 Hz, 2010,058,06:52:15.069539
...
```
And execute with the `-s` will show the brief summary like this:
```
$ ./libmseed_hello_world -s test.mseed
...
Records: 107, Samples: 12600
```

## Explaination of Code
Let's deep dive into `libmseed`'s functions!

First, to read one record from a miniSEED file, we can use `ms3_readmsr()`.
The function will read a record into a struct of type `MS3Record **`. What's
more, you can set the flag using `|=` operator then pass the flag to this function
so that it performs certain actions. In here I pass `MSF_VALIDATECRC` and `MSF_UNPACKDATA`
to let the function performs both validate the data using CRC and unpack the
data into `MS3Record` type.

Then, as a computer engineer, I always concern about the length of data and 
its values. Here I will divide into two parts to tell you where the length and
values reside:
1. The length of data is the `numsamples` variable in `MS3Record`.
2. The values of data is the `datasamples1 variable in `MS3Record`.
    - The most interesting part is that the type of values is `void`. I guess
is for storing different types easily in C.

Next, we want to output each element of the values. To output the message, `libmseed`
provides a handy function called `ms_log()`. Besides, you can pass `int` to `level`
parameter for different message level.
> Note that when doing any manupulation of values, you should always get the type
> of them (`sampletype` if using `MS3Record`) then cast them to appropiate type.
> It is also a good practice in C that the programming must know what type of the
> variable is.
> 

Eventually, you should clean up what you have allocated in C. The library provides
the following approach to clean `MS3Record` struct: 
```
ms3_readmsr (&msr, NULL, NULL, NULL, 0, 0);
```

## Conclusion and Thoughts
In this post, I introduce what is miniSEED and `libmseed`. Also, I demonstrate how
to easily read the miniSEED file with explainations of necessary functions. I hope
this post will be a great introduction to those who are working on seismic wave and
people who want to port version 3 and above of `libmseed` to other programming 
language and platform.

# References
[^1] http://ds.iris.edu/ds/nodes/dmc/data/formats/miniseed/
[^2] https://github.com/iris-edu/libmseed/blob/master/ChangeLog#L62
[^3] https://stackoverflow.com/questions/17889799/libraries-in-usr-local-lib-not-found
