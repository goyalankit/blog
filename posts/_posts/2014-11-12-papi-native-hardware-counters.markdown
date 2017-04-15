---
layout: post
title: "Papi native hardware counters"
date: 2014-11-12 16:11:50 -0600
comments: true
categories: papi hpc
---

## Check for the available native hardware counters:

{% highlight sh %}
$ papi_native_avail
{% endhighlight %}

## Sample code to get the values for native hardware counters

{% highlight c %}
#include <stdio.h>
#include <papi.h>

#define MAX_COUNTERS 256
int rozmer = 1;

static char * events[] = {
  "PERF_COUNT_HW_CACHE_L1D:WRITE",
  "PERF_COUNT_HW_CACHE_L1D:ACCESS",
  "PERF_COUNT_HW_CACHE_L1D:READ",
  "PERF_COUNT_HW_CACHE_L1D:PREFETCH",
  "PERF_COUNT_HW_CACHE_L1D:MISS",
  "HW_PRE_REQ:L1D_MISS",
  //"L1-DCACHE-LOADS"
};

#define NUMCOUNTERS 6
void compute() {
  int EventCode, retval;
  int EventSet = PAPI_NULL;
  long long PAPI_Counters[MAX_COUNTERS];

  /* Initialize the library */
  retval = PAPI_library_init(PAPI_VER_CURRENT);

  if  (retval != PAPI_VER_CURRENT) {
    fprintf(stderr, "PAPI library init error!\n");
    exit(1);
  }

  /* PAPI create event */
  if (PAPI_create_eventset(&EventSet) != PAPI_OK) {
    fprintf(stderr, "create event set: %s", PAPI_strerror(retval));
  }

  /* Add events to Event Set */
  for (int i = 0; i < NUMCOUNTERS; i++) {
    if ((retval = PAPI_add_named_event(EventSet, events[i])) != PAPI_OK) {
        fprintf(stderr, "add named event: %s", PAPI_strerror(retval));
    }
  }

  /* Start the counters */
  retval = PAPI_start(EventSet);

  float matice1[rozmer][rozmer];
  float matice2[rozmer][rozmer];
  float matice3[rozmer][rozmer];

  // Code for which we want hardware counters.

  // Start
  int j,k,m;
  for(j = 0; j < rozmer; j++)
  {
    for (k = 0; k < rozmer; k++)
    {
      float temp = 0;
      for (m = 0; m < rozmer; m++)
      {
        temp = temp + matice1[j][m] * matice2[m][k];
      }
      matice3[j][k] = temp;
    }
  }
  // End

  /* Stop the counters */
  retval = PAPI_stop(EventSet, PAPI_Counters);

  if (retval != PAPI_OK) exit(1);

  /* Print the counters */
  for (int j = 0; j < NUMCOUNTERS; j++) {
    printf("%20lld %s\n", PAPI_Counters[j], events[j]);
  }
}

void main(){
  compute();
}

{% endhighlight %}
