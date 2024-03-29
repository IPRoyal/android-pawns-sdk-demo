![GitHub Latest Release)](https://img.shields.io/github/v/release/IPRoyal/android-pawns-sdk-demo?logo=github&style=flat&color=5324CE)

![alt text](https://pawns.app/wp-content/uploads/2022/12/pawns-app-dark.svg)

# Pawns SDK  #
-------------
## An internet sharing library for Android  ##

Contact our representative to get terms and conditions and collect all information needed.

### Content ###
- [Information needed](#information-needed)
- [Installation](#installation)
- [Setup](#setup)
- [How to use](#how-to-use)
- [Service types](#service-types)
- [Main Functionality](#main-functionality)
- [Foreground service](#foreground-service)
- [Increase SDK service run time](#increase-sdk-service-run-time)
  - [Battery optimisation settings](#battery-optimisation-settings)
  - [Launch on boot complete](#launch-on-boot-complete)
- [License](#license)

### Summary ###

* Min Sdk 21

### Information needed ###

* API KEY

## Installation ##

Add jitpack repository to your project level build.gradle
````
allprojects {
    maven { url 'https://jitpack.io' }
}
````

Within build.gradle (:app) dependencies section add library
````
implementation'com.github.IPRoyal:android-pawns-sdk:x.y.z'
````

## Setup ##

Make a few updates in the onCreate method of your app's Application class

Use Pawns.Builder to build and setup SDK for later use in your application. It is a mandatory to provide your API key as it is used to identify your developer account when Internet Sharing service is running.
By default SDK will start service as **FOREGROUND** service. Optionally, you can provide service configuration to modify what exactly foreground service notification will display to user, when sharing service is launched.
**TIP** *It is recommended to setup service configuration, because it acts as an explanation to the user why your application is running foreground service.*

Setup code example:

**Kotlin**

````
    override fun onCreate() {
        super.onCreate()

        Pawns.Builder(context)
            .apiKey("Your api key here")
            .serviceConfig(
                ServiceConfig(
                    title = R.string.service_name,
                    body = R.string.service_body,
                    smallIcon = R.drawable.ic_demo_icon
                )
            )
            .serviceType(ServiceType.FOREGROUND)
            .build()
    }  
````

**Java**

````
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        new Pawns.Builder(context)
                .apiKey("Your api key here")
                .serviceConfig(
                        new ServiceConfig(
                                R.string.service_name,
                                R.string.service_body,
                                R.drawable.ic_stat_name,
                                ServiceNotificationPriority.DEFAULT,
                                null
                        )
                )
                .serviceType(ServiceType.FOREGROUND)
                .build();
    }  
````

### How to use ####

#### Service types ####

There are currently two types of service you can run:
* Background service does not require to display notification, so it is less intrusive to the user. Downside is that application must be 
active and running, otherwise it will get terminated. Putting phone to sleep will stop service as well.
* Foreground service must display a notification as this service can run independently from application. Meaning that it will continue to run even when
application is not active and not running. If application is closed there are still some specific cases when service will get terminated after a while, but 
we will provide tips how to extend time of running later on.

#### Main Functionality ####

Our SDK provides 3 main functionalities

* Starting service
* Stopping service
* Exposing state of service

#### Starting service ####

**Kotlin**

````
Pawns.getInstance().startSharing(context)
````
**Java**

````
Pawns.Companion.getInstance().startSharing(context);
````

#### Stopping service ####

**Kotlin**

````
Pawns.getInstance().stopSharing(context)
````

**Java**

````
Pawns.Companion.getInstance().stopSharing(context);
````

#### Observing state of service ####

Depending on your technology stack this may vary, but we covered two main use cases, which hopefully will help you out.

#### Coroutines and Composable ####

Collecting service state for composable components is very similar to collecting state from viewModel. Pawns instance exposes coroutines StateFlow serviceState, which can be observed easily.
````
val state = Pawns.getInstance().getServiceState().collectAsState(Dispatchers.Main.immediate)
````
#### Xml and Listeners ####

Within your activity or fragment implement our PawnsServiceListener, which will get triggered every time our service state changes. Updating UI within onStateChange method might require using **runOnUiThread**, because the state change happens while running on background thread.
````
public interface PawnsServiceListener {
    public fun onStateChange(state: ServiceState)
}
````
We provide with listener register/unregister methods

**Kotlin**

````
    override fun onCreate() {
        super.onCreate()
        Pawns.getInstance().registerListener(pawnsServiceListener)
    }

    override fun onDestroy() {
        super.onDestroy()
        Pawns.getInstance().unregisterListener()
    }  
````

**Java**

````
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        Pawns.Companion.getInstance().registerListener(pawnsServiceListener);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        Pawns.Companion.getInstance().unregisterListener();
    } 
````

The current service state can also be read through its value property

**Kotlin**

````
    val lastKnownState = Pawns.getInstance().getServiceStateSnapshot()
````

**Java**

````
    ServiceState lastKnownState = Pawns.Companion.getInstance().getServiceStateSnapshot();
````

## Foreground service ##

Setup internet sharing service notification channel name (displayed to users in application settings under notification section). Default value is "Sharing service".

````
   <application
   ...

        <meta-data
            android:name="com.iproyal.sdk.pawns_service_channel_name"
            android:value="My Internet Sharing" />

    </application>
````

### Increase SDK service run time ###

#### Battery optimisation settings ####

The Android system stops a service only when memory is low and it must recover system resources for the activity that has user focus. If the service is bound to an activity that has user focus, it's less likely to be killed; if the service is declared to run in the foreground, it's rarely killed.

Our library is using foreground services. By default android system does not kill foreground services, but a fair number of manufacturers such as “Xiaomi”, “Honor” and others have their own battery optimisation layer and kills foreground services after awhile.

One of the solutions is to request users to go to your application settings and remove battery optimisation or in other words remove battery restrictions.
````
Intent(Settings.ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS).apply {
    data = Uri.parse("package:" + context.packageName)
}
````
#### Launch on boot complete ####

Android applications can declare a BroadcastReceiver, which gets triggered when android device restarts and finishes booting. When receiving this event, simply Pawns.instance.startSharing(context).

If you have not done initialisation during onCreate() of your application class, then you will need to initialise Pawns when receiving boot complete action.

Some android devices have a special application setting option to allow AutoStart. If this option is not enabled, then action for boot complete will not be received.


# License #
~~~~
 Copyright 2022 Pawns.app.

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
~~~~