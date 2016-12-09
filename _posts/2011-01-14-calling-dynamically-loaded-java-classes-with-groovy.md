---
layout: post
title: Calling dynamically loaded Java classes with Groovy
---
So I've got this ridiculous task ahead of me... I somehow need to call a Spring Bean data access object (DAO) from InstallAnywhere using groovy scripting. The idea is that the webapp being installed has all the code in order to set up the database, but there are certain things that need to be done by the installer. Part of getting this working is dynamically loading jars from a directory, and using reflection to grab the class and method I need to call...
Instead of duplicating code and having to maintain it in multiple places, I asked the dev team to provide me a singular class member function that I can call externally. Needless to say, they failed to deliver, and blindly pointed me towards the spring DAOs they were using to access the database. This requires a whole bunch of additional setup in order to get working, since it was written to work in the webapp context.

I wrote a simple class to wrap all that stuff away for me (it's not working quite yet, but I'm hopeful that we can get the problems worked out,) and now I need to integrate it with the installer. There are two basic approaches here. One is to build an IA plugin that will link in the code directly. The problem here is that I don't want to be building installing plugins on the fly during the build, especially when other installer builds need to use that IA instance. So that idea is out.

The alternative is to package the jar and it's dependencies in the installer, and extract them when needed. IA comes with a nifty (and until now, unknown to me) plugin that allows you to extract files from the installer during the pre-installation phase. So how do I run it?

You can always add a main class, of course, but I'm far more masochistic than that. We have a simple plugin that runs a groovy script provided as a variable, so I figured that's the best way to go. If you don't have such a plugin, I highly recommend writing something like it. I can't provide ours, but it shouldn't be too difficult to quickly throw together.

{% highlight groovy }
def classLoader;

def loadJars(jarPath) {
	def topClassLoader = ClassLoader.systemClassLoader;
	while(topClassLoader.parent){
		topClassLoader = topClassLoader.parent;
	}

	//this is a hack in order to get the system classloader updated
	//in the event that your java code tries to use it for it's own reflection
	//routines. Might not work for all JREs.
	def addURL = URLClassLoader.class.getDeclaredMethod("addURL", [URL.class] as Class[]);
	addURL.setAccessible(true);
	ClassLoader cl = ClassLoader.getSystemClassLoader();

	def jarList = [];
	def libDir = new File(jarPath);
	libDir.eachFileMatch(~".*jar"){f ->
		jarList.add(f.toURL());
		addURL.invoke(cl, [f.toURL()] as URL[]);
	}
	classLoader = new URLClassLoader(jarList as URL[], topClassLoader);
}

def getMethod() {
	def api = Class.forName("com.corp.ClassOfInterest", true, classLoader);
	def updateMethod = api.getMethod("methodOfInterest", [java.lang.String.class, java.lang.String.class, java.lang.String.class] as Class[]);
	return updateMethod;
}

def executeMethod(arg1, arg2, arg3) {
	def methodOfInterest = getMethod();
	methodOfInterest.invoke(null,
		[	new String(arg1),
			new String(arg2),
			new String(arg3)
		] as Object[]);
}

loadJars(this.args[0]);
executeMethod(this.args[2], this.args[3], this.args[1]);
{% endhighlight %}

In my case, the script above was written to be executed either from the command line, or with groovy's `run()` function. It creates a classloader with all the jar files in the directory given in the first argument, and uses reflection to locate a static function in a class. You could get a member function of course, but I didn't have a need for any object instances.

Provided the function you are trying to call actually works, there should be no problem... hopefully. XD

