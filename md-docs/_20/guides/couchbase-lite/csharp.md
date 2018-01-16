## Getting Started

- Add `http://mobile.nuget.couchbase.com/nuget/Developer/` to your Nuget package sources and expect a new build approximately every 2 weeks!

Your app must call the relevant `Activate()` function inside of the class that is included in the support assembly (there is only one public class in each support assembly, and the support assembly itself is a nuget dependency).  For example, UWP looks like `Couchbase.Lite.Support.UWP.Activate()`.  Currently the support assemblies provide dependency injected mechanisms for default directory logic, and platform specific logging (i.e. Android will log to logcat with correct log levels and tags.  No more "mono-stdout" at always info level).

### Resources

The API references for the .NET SDK are available [here](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db021).