---
layout: post
title:  "Locating Xcode6 instruments templates"
date:   2014-08-16 02:08:34
author: de_poon
author_link: //github.com/depoon
---

With the help of my colleagues, I am trying to request the author of calabash to fix (via Pull Request) an issue of run_loop repo with XCode6.

Apparently the issue lies with the output of

{% highlight bash %}
xcrun instruments -s templates
{% endhighlight %}

when XCode6SDK is set at `/Applications/Xcode.app/Contents/Developer` the output is:

{% highlight bash %}
Known Templates:
"Activity Monitor"
"Allocations"
"Automation"
"Blank"
"Cocoa Layout"
"Core Animation"
"Core Data"
"Counters"
"Dispatch"
"Energy Diagnostics"
"File Activity"
"GPU Driver"
"Leaks"
"Multicore"
"Network"
"OpenGL ES Analysis"
"Sudden Termination"
"System Trace"
"System Usage"
"Time Profiler"
"UI Recorder"
"Zombies"
{% endhighlight %}

However if your SDK is at a different location, eg `/Applications/Xcode6.app`, the output becomes

{% highlight bash %}
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/PlugIns/AutomationInstrument.xrplugin/Contents/Resources/Automation.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/PlugIns/OpenGLESAnalyzerInstrument.xrplugin/Contents/Resources/OpenGL ES Analysis.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/PlugIns/XRMobileDeviceDiscoveryPlugIn.xrplugin/Contents/Resources/Energy Diagnostics.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/PlugIns/XRMobileDeviceDiscoveryPlugIn.xrplugin/Contents/Resources/GPU Driver.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/PlugIns/XRMobileDeviceDiscoveryPlugIn.xrplugin/Contents/Resources/templates/Core Animation.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/PlugIns/XRMobileDeviceDiscoveryPlugIn.xrplugin/Contents/Resources/templates/System Usage.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/Resources/templates/Activity Monitor.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/Resources/templates/Allocations.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/Resources/templates/Blank.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/Resources/templates/Counters.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/Resources/templates/Leaks.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/Resources/templates/Network.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/Resources/templates/System Trace.tracetemplate"
"/Applications/Xcode6.app/Contents/Applications/Instruments.app/Contents/Resources/templates/Time Profiler.tracetemplate"
"/Applications/Xcode6.app/Contents/Developer/Platforms/MacOSX.platform/Developer/Library/Instruments/PlugIns/CoreData/Core Data.tracetemplate"
"/Applications/Xcode6.app/Contents/Developer/Platforms/MacOSX.platform/Developer/Library/Instruments/PlugIns/templates/Cocoa Layout.tracetemplate"
"/Applications/Xcode6.app/Contents/Developer/Platforms/MacOSX.platform/Developer/Library/Instruments/PlugIns/templates/Dispatch.tracetemplate"
"/Applications/Xcode6.app/Contents/Developer/Platforms/MacOSX.platform/Developer/Library/Instruments/PlugIns/templates/File Activity.tracetemplate"
"/Applications/Xcode6.app/Contents/Developer/Platforms/MacOSX.platform/Developer/Library/Instruments/PlugIns/templates/Multicore.tracetemplate"
"/Applications/Xcode6.app/Contents/Developer/Platforms/MacOSX.platform/Developer/Library/Instruments/PlugIns/templates/Sudden Termination.tracetemplate"
"/Applications/Xcode6.app/Contents/Developer/Platforms/MacOSX.platform/Developer/Library/Instruments/PlugIns/templates/UI Recorder.tracetemplate"
"/Applications/Xcode6.app/Contents/Developer/Platforms/MacOSX.platform/Developer/Library/Instruments/PlugIns/templates/Zombies.tracetemplate"
{% endhighlight %}

The difference in output affects how run_loop interprets and parses it.

If you need a quick fix for v.1.0.2

{% highlight bash %}
/Users/xxxXXXXxxx/.rvm/gems/ruby-2.0.0-p353/gems/run_loop-1.0.2/lib/run_loop
{% endhighlight %}

{% highlight bash %}
lib/run_loop/core.rb 560: not name == 'Automation'
{% endhighlight %}

change to
{% highlight bash %}
lib/run_loop/core.rb 560: not name =~ /\/Automation/
{% endhighlight %}