---
title: "Creating or modifying a JANA factory in order to implement a reconstruction algorithm"
teaching: 15
exercises: 20
questions:
- "How to write a reconstruction algorithm in EICrecon?"
objectives:
- "Learn how to create a new factory in EICrecon that supplies a reconstruction algorithm for all to use."
- "Understand the directory structure for where the factory should be placed in the source tree."
- "Understand how to use a generic algorithm in a JANA factory."
keypoints:
- "Create a factory for reconstructing single subdetector data or for global reconstruction."
---

## Introduction


Now that you've learned about JANA plugins and JEventProcessors, let's talk about JFactories. JFactories are another essential JANA component just like JEventProcessors. While JEventProcessors are used for _aggregating_ results from each event into a structured output such as a histogram or a file, JFactories are used for computing those results in an organized way.

In theory, you could calculate whatever results you need right in the body of your JEventProcessor's Process() method. However, JFactories are definitely the right way to go if your code is going to stick around for awhile, be used by multiple people, and/or be integrated into the EICrecon codebase itself. Here's why:


1. They make your code reusable. Different people can use your results later without having to understand the specifics of what you did.

2. If you are consuming some data which doesn't look right to you, JFactories make it extremely easy to pinpoint exactly which code produced this data.

3. EICrecon needs to run multithreaded, and using JFactories can help steer you away from introducing thorny parallelism bugs.

4. You can simply ask for the results you need and the JFactory will provide it. If nobody needs the results from the JFactory, it won't be run. If the results were already in the input file, it won't be run. If there are multiple consumers, the results are only computed once and then cached. If the JFactory relies on results from other JFactories, it will call them transparently and recursively. 



## Algorithms vs Factories 

In general, a Factory is a programming pattern for constructing objects in an abstract way. Oftentimes, the Factory is calling an algorithm under the hood. 
This algorithm may be very generic. For instance, we may have a Factory that produces Cluster objects for a barrel calorimeter, and it calls a clustering algorithm 
that doesn't care at all about barrel calorimeters, just the position and energy of energy of each CalorimeterHit object. Perhaps multiple factories for creating clusters 
for completely different detectors are all using the same algorithm. 

Note that Gaudi provides an abstraction called "Algorithm" which is essentially its own version of a JFactory. In EICrecon, we have been
separating out _generic algorithms_ from the old Gaudi and new JANA code so that these can be developed and tested independently. In JANA, factories are uniquely identified by their 
object type and their tag, which is just a string. The tag plays a similar role as the collection name in Gaudi. 


## Parallelism considerations


JEventProcessors observe the entire _event stream_, and require a _critical section_ where only one thread is allowed to modify a shared resource (such as a histogram) at any time. 
JFactories, on the other hand, only observe a single event at a time, and work on each event independently. Each worker thread is given an independent event with its own set of factories. 
This means that for a given JFactory instance, there will be only one thread working on one event at any time. You get the benefits of multithreading _without_ having to make each JFactory thread-safe.


You can write JFactories in an almost-functional style, but you can also cache some data on the JFactory that will stick around from event-to-event. This is useful for things like conditions and geometry data, where for performance reasons you don't want to be doing a deep lookup on every event. Instead, you can write callbacks such as `BeginRun()`, where you can update your cached values when the run number changes. 


Note that just because the JFactory _can_ be called in parallel doesn't mean it always will. If you call event->Get() from inside `JEventProcessor::ProcessSequential`, in particular, the factory will run
single-threaded and slow everything down. However, if you call it using `Prefetch` instead, it will run in parallel and you may get a speed boost.


** How do I use an existing JFactory? **

Using an existing JFactory is extremely easy! Any time you are someplace where you have access to a `JEvent` object, do this:


~~~ c++

auto clusters = event->Get<edm4eic::Cluster>("EcalEndcapNIslandClusters");

for (auto c : clusters) {
  // ... do something with a cluster
}

~~~

As you can see, it doesn't matter whether the `Cluster` objects were calculated from some simpler objects, or were simply loaded from a file. This is a very powerful concept. 

One thing we might want to do is to swap one factory for another, possibly even at runtime. This is easy to do if you just make the factory tag be a parameter:


~~~ c++

std::string my_cluster_source = "EcalEndcapNIslandClusters";  // Make this be a parameter

auto clusters = event->Get<edm4eic::Cluster>(my_cluster_source);

for (auto c : clusters) {
  // ... do something with a cluster
}

~~~

## How do I create a new JFactory?


~~~ c++
class Cluster_factory_EcalEndcapNIslandClusters : public JFactoryT<edm4eic::Cluster> {
public:

    Cluster_factory_EcalEndcapNIslandClusters() {
        SetTag("EcalEndcapNIslandClusters");
    }


    void Init() override {
        auto app = GetApplication();

        // This is an example of how to declare a configuration parameter that
        // can be set at run time. e.g. with -PEEMC:EcalEndcapNIslandClusters:scaleFactor=0.97
        m_scaleFactor =0.98;
        app->SetDefaultParameter("EEMC:EcalEndcapNIslandClusters:scaleFactor", m_scaleFactor, "Energy scale factor");

        // This is how you access shared resources using the JService interface
        m_log = app->GetService<Log_service>()->logger("JEventProcessorPODIO");
    }


    void Process(const std::shared_ptr<const JEvent> &event) override {

    	log->info("Processing event {}", event->GetEventNumber());
        // Grab inputs
        auto protoclusters = event->Get<edm4eic::ProtoCluster>("EcalEndcapNIslandProtoClusters");

        // Loop over protoclusters and turn each into a cluster
        std::vector<edm4eic::Cluster*> outputClusters;
        for( auto proto : protoclusters ) {

        	// ======================
        	// Algorithm goes here!

        	// auto cluster = new edm4eic::Cluster( ... );
            // outputClusters.push_back( cluster );
        	// ======================
        }

        // Hand ownership of algorithm objects over to JANA
        Set(outputClusters);
    }

private:
    float m_scaleFactor;
    std::shared_ptr<spdlog::logger> m_log;
};

~~~

You can't pass JANA a JFactory directly (because it needs to create an arbitrary number of them on the fly). Instead you pass it a `JFactoryGenerator` object: 

~~~ c++

// In your plugin's init

#include <JANA/JFactoryGenerator.h>
// ...
#include "ProtoCluster_factory_EcalEndcapNIslandProtoClusters.h"

extern "C" {
    void InitPlugin(JApplication *app) {
        InitJANAPlugin(app);
        // ...

        app->Add(new JFactoryGeneratorT<Cluster_factory_EcalEndcapNIslandClusters>());
     }

~~~



## How can I generate a JFactory skeleton?

JANA comes with a script called `jana-generate.py` which lives in `$JANA_HOME/bin/jana-generate.py`. This script will generate skeletons for a number of different JANA components, including JFactories.
Note that this doesn't have any customizations specific to EICrecon. Eventually we may add an EICrecon-specific skeleton, analogous to `eicmkplugin.py`. To generate a skeleton JFactory, run `jana-generate.py` like this:

~~~ bash
$ jana-generate.py JFactory edm4eic::Cluster EcalEndcapNIslandClustersImproved
~~~

- `edm4eic::Cluster` is the object name, a.k.a. the type of objects this factory produces. 
- `EcalEndcapNIslandClustersImproved` is the "tag" or collection name. 


This generates `.cc` and `.h` files for your new factory. 
Note that it also generates a stub definition of "edm4eic::Cluster.h", which we definitely _don't_ want, so just delete that file.
Also note that in EICrecon, we've adopted the naming convention "ObjectName_factory_CollectionName", which this version of the script does _not_ do yet. So go ahead and rename it.

Anyway, you now have a stub factory! Feel free to play around with it.




{% include links.md %}
