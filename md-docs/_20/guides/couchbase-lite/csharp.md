## Getting Started

- Add `http://mobile.nuget.couchbase.com/nuget/Developer/` to your Nuget package sources and expect a new build approximately every 2 weeks!

Your app must call the relevant `Activate()` function inside of the class that is included in the support assembly (there is only one public class in each support assembly, and the support assembly itself is a nuget dependency).  For example, UWP looks like `Couchbase.Lite.Support.UWP.Activate()`.  Currently the support assemblies provide dependency injected mechanisms for default directory logic, and platform specific logging (i.e. Android will log to logcat with correct log levels and tags.  No more "mono-stdout" at always info level).

## Supported Versions

This is an attempt to show versions of things that .NET is supposed to support.  This page may fall out of date, and the authoritative answer for this will come from the Couchbase support and/or management team.  Couchbase Lite .NET will be a .NET Standard 2.0 library as of the Couchbase Lite .NET 2.0 GA release (and hopefully, its beta release of 2.0).  

| .NET Runtime     | Minimum Runtime Version | Minimum OS version |
| ---------------- | ----------------------- | ------------------ |
| .NET Core Win    | 2.0                     | 10 (any supported) |
| .NET Core Mac    | 2.0                     | 10.12              |
| .NET Core Linux  | 2.0                     | n/a*               |
| .NET Framework   | 4.6.1                   | 10 (any supported) |
| UWP              | 6.0.1                   | 10.0.16299         |
| Xamarin iOS      | 10.14                   | 9.0                |
| Xamarin Android  | 8.0                     | 4.1 (API 16)       |

* There are many different variants of Linux, and we don't have the resources to test all of them.  They are tested on Ubuntu 16.04, but have been shown to work on CentOS, and in theory work on any distro supported by .NET Core.

Comparing this to 1.x you can see we've traded some lower obsolete versions for new platform support:

| .NET Runtime     | Minimum Runtime Version | Minimum OS version |
| ---------------- | ----------------------- | ------------------ |
| .NET Framework   | 3.5                     | XP                 |
| Mono Mac         | 5.2*                    | 10.9               |
| Mono Linux       | 5.2*                    | n/a**              |
| Xamarin iOS      | 8.0*                    | 8.0                |
| Xamarin Android  | 4.6*                    | 2.3 (API 9)        |

* These runtime versions are approximate.  Couchbase Lite 1.x is built using relatively new versions of all of these and an absolute minimum is unclear, and cannot actually be determined anymore due to lack of vendor support (i.e. Xamarin iOS 8 uses Xcode 6, etc).  So basically "any version" is an approximately good metric here.

** See above note about Linux

### Resources

The API references for the .NET SDK are available [here](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db021).