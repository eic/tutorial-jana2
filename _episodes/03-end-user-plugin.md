---
title: "Creating a plugin to make custom histograms/trees"
teaching: 15
exercises: 20
questions:
- "Why should a I make a custom plugin?"
- "How do I create a custom plugin?"
objectives:
- "Understand when one should make a plugin and when they should just use a ROOT macro."
- "Understand how to create a new, stand-alone plugin with your own custom histograms."
keypoints:
- "Plugins can be used to generate custom histograms by attaching directly to the reconstruction process."
- "Plugins can be used for monitoring or custom analysis."
---
Plugins are the basic building blocks when it comes to analyzing data.  They request objects and perform actions, such as making histograms, or writing out certain objects to files.  When your plugin requests objects (e.g. clusters) the factory responsible for the requested object is loaded and run (We will dive into factories in the next exciting episode of how to use JANA).  When running EICrecon you will configure it to use some number of plugins (each potentially with their own set of configuration parameters). Now, let us begin constructing a new plugin.

To do this we will use the eicmkplugin.py script that comes with EICrecon.  This utility should be your "go-to" for jumpstarting your work with EICrecon/JANA. 

The eickmkplugin script can be called simply by typing: "eicmkplugin.py" followed by the name of the plugin. Let's begin by calling: "eicmkplugin.py  myPlugin".  After a moment there should now exist a new folder labeled "myPlugin". That directory contains 2 files: a CMakelists.txt file (needed for compiling our new plugin) and the source code for the plugin itself. 

Inside the source code for your plugin is a fairly simple class.  The private data members should contain the necessary variables to successfully run your plugin;  this will likely include any histograms, canvases, fits or other accoutrement. The public section contains the required Constructor, Init, Process, and Finish functions.  In the constructor we need to supply JANA the name of our plugin.  In init we get the application, as well as initialize any variables or histograms (etc etc).  The Process function typically gets objects from the event and does something with them (e.g. fill the histogram of cluster energy). And finally Finish is called where we clean up and do final things like writing the resulting root files.

To begin, start by setting your EICrecon_MY environment variable to a directory where you have write permission. The build instructinos will install the plugin to that directory. When eicrecon is run, it will also look for plugins in the $EICrecon_MY directory and the EICrecon build you are using.

~~~
mkdir EICrecon_MY
export EICrecon_MY=${PWD}/EICrecon_MY
~~~

To generate a plugin, do the following:
~~~
eicmkplugin.py myFirstPlugin
cmake -S myFirstPlugin -B myFirstPlugin/build
cmake --build myFirstPlugin/build --target install
~~~

You can test plugin installed and can load correctly by runnign eicrecon with it: 
~~~
eicrecon -Pplugins=myFirstPlugin,JTest -Pjana:nevents=10
~~~

The second plugin, JTest, just supplies dummy events. To generate your first histograms, edit the myFirstPluginProcessor.cc and myFirstPluginProcessor.h files (located in the myFirstPlugin directory). It should look similar to the one below: 

```c++
#include <JANA/JEventProcessorSequentialRoot.h>
#include <TH2D.h>
#include <TFile.h>

#include <edm4hep/SimCalorimeterHit.h>

class myFirstPluginProcessor: public JEventProcessorSequentialRoot {
private:

    // Data objects we will need from JANA e.g.
    PrefetchT<edm4hep::SimCalorimeterHit> rawhits   = {this, "EcalBarrelHits"};

    // Declare histogram and tree pointers here. e.g.
    TH1D* hEraw = nullptr;

public:
    myFirstPluginProcessor() { SetTypeName(NAME_OF_THIS); }
    
    void InitWithGlobalRootLock() override;
    void ProcessSequential(const std::shared_ptr<const JEvent>& event) override;
    void FinishWithGlobalRootLock() override;
};
```
Next, edit the myFirstPluginProcessor.cc file to the following:
```c++
#include "myFirstPluginProcessor.h"
#include <services/rootfile/RootFile_service.h>

// The following just makes this a JANA plugin
extern "C" {
    void InitPlugin(JApplication *app) {
        InitJANAPlugin(app);
        app->Add(new DaveTestProcessor);
    }
}

//-------------------------------------------
// InitWithGlobalRootLock
//-------------------------------------------
void myFirstPluginProcessor::InitWithGlobalRootLock(){

    // This ensures the histograms created here show up in the same root file as
    // other plugins attached to the process. Place them in dedicated directory
    // to avoid name conflicts.
    auto rootfile_svc = GetApplication()->GetService<RootFile_service>();
    auto rootfile = rootfile_svc->GetHistFile();
    rootfile->mkdir("myFirstPlugin")->cd();

    hEraw  = new TH1D("Eraw",  "BEMC hit energy (raw)",  100, 0, 0.075);
}

//-------------------------------------------
// ProcessSequential
//-------------------------------------------
void myFirstPluginProcessor::ProcessSequential(const std::shared_ptr<const JEvent>& event) {

     for( auto hit : rawhits() ) hEraw->Fill(  hit->getEnergy() );
}

//-------------------------------------------
// FinishWithGlobalRootLock
//-------------------------------------------
void myFirstPluginProcessor::FinishWithGlobalRootLock() {

    // Do any final calculations here.
}
```
You can test the plugin using the following simulated data file:

~~~bash
wget https://eicaidata.s3.amazonaws.com/2022-09-04_pgun_e-_podio-0.15_edm4hep-0.6_0-30GeV_alldir_1k.edm4hep.root
eicrecon -Pplugins=myFirstPlugin 2022-09-04_pgun_e-_podio-0.15_edm4hep-0.6_0-30GeV_alldir_1k.edm4hep.root
~~~

You should now have a root file, eicrecon.root, with the resulting hEraw histogram inside of it.




{% include links.md %}

