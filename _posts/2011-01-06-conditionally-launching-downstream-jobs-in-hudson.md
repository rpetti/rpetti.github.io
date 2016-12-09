---
layout: post
title: Conditionally launching downstream jobs in Hudson
---
This recently came up in the hudson users mailing list, so I figured I'd post it here.

Lets say you wanted to setup a downstream job that would only run when certain conditions are met (such as certain build parameters, or anything else for that matter). Well, this is what I typically use the [Groovy Postbuild Plugin](http://wiki.hudson-ci.org/display/HUDSON/Groovy+Postbuild+Plugin) for!

Here's an example of the code I'm making use of:

{% highlight groovy %}
if("true".equals(manager.build.buildVariables.get("buildInstaller")) &&
      manager.build.getResult().equals(hudson.model.Result.SUCCESS)) {
   def nextJob = manager.hudson.getItem("Installer");
   nextJob.scheduleBuild(5,
      new hudson.model.Cause.UpstreamCause(manager.build),
      new hudson.model.ParametersAction(
           new hudson.model.StringParameterValue("branchDir",
                   manager.build.buildVariables.get("branchDir"))
           )
      );
}
{% endhighlight %}

This will launch the job called "Installer" when the current job succeeds, and the "buildInstaller" job parameter equals true. It will also pass the current value of the "branchDir" parameter and pass it into the next build. If you want to add more parameters, just add another StringParameterValue object to the ParametersAction constructor. If you want to add a static parameter value, just replace the `manager.build.buildVariables.get("branchDir")` with the string literal you want.

It's important to know that while this does indeed launch the job, the dependencyGraph and other hudson plugins will not see it as a downstream job. Unfortunately, I don't think there's not much we can do about that right now without writing a whole new plugin, or enhancing an existing one like the parameterized trigger plugin.

