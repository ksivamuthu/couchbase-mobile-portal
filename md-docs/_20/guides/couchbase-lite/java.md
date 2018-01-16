<br />
⚠ Support in the current Developer Build is for Android only. The SDK cannot be used in Java applications.

## Getting Started

- In the top-level **build.gradle** file, add the following Maven repository in the `allprojects` section.

	```groovy
	allprojects {
		repositories {
			jcenter()
			maven {
				url "http://mobile.maven.couchbase.com/maven2/dev/"
			}
		}
	}
	```

- Next, add the following in the `dependencies` section of the application's **build.gradle** (the one in the **app** folder).

	```groovy
	dependencies {
		compile 'com.couchbase.lite:couchbase-lite-android:2.0.0-DB021'
	}
	```

### Resources

The API references for the Java SDK are available [here](http://docs.couchbase.com/mobile/2.0/couchbase-lite-java/db021).
