---
title: "AIDL : IPC, The Android way"
datePublished: Sun Jun 04 2023 13:51:34 GMT+0000 (Coordinated Universal Time)
cuid: clihheqmd00010al1gvfrbrz5
slug: aidl-ipc-the-android-way
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685887032744/adbfd0dc-1df0-4716-960a-552915105d11.avif
tags: android, rpc, rpc-framework, aidl

---

> The Android Interface Definition Language (AIDL) is similar to other [IDLs](https://en.wikipedia.org/wiki/Interface_description_language): it lets you define the programming interface that both the client and service agree upon in order to communicate with each other using interprocess communication (IPC).

Aidl consists of 3 parts:

1. AIDL file
    
2. Client application
    
3. Server application
    

### Creating AIDL file

1. Enable build feature in your `build.gradle`.
    
    ```java
    buildFeatures {
        aidl = true
    }
    ```
    
2. Create a new AIDL file using `File` -&gt; `New` -&gt; `AIDL` -&gt; `AIDL File`
    
3. Let's name it `ICalculator.aidl`
    
4. Declare aidl methods:
    
    ```kotlin
    interface IExampleAidlInterface {
        int add(int a, int b);
    }
    ```
    

### Create an AIDL server

An AIDL server is an app that does the actual work and sends back the result to requesting clients. One service can connects with multiple clients via Binders.

Steps:

1. Create a service, e.g. `CalculatorService extends Service`
    
2. Create an Implementation of `ICalculator.Stub`, lets name it `CalculatorImpl`
    
3. Implement methods:
    
    ```kotlin
    object CalculatorImpl : ICalculator.Stub() {
        override fun add(a: Int, b: Int): Int {
            return a + b
        }
    }
    ```
    
4. Expose your stub implementation with a service binder:
    
    ```kotlin
    class CalculatorService : Service() {
        override fun onBind(intent: Intent): IBinder {
            return CalculatorImpl
        }
    }
    ```
    

Now your service is ready to be consumed by your clients, now let's create a client application.

### Create an AIDL client

An AIDL client is an app that binds to an AIDL service, calls API, shows response inside its own views.

Steps:

1. Create a service connection:
    
    ```kotlin
    private var aidlService: ICalculator? = null
    private val serviceConnection = object : ServiceConnection {
        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
            aidlService = ICalculator.Stub.asInterface(service)
            Toast.makeText(applicationContext, "Service Connected", Toast.LENGTH_LONG).show()
        }
    
        override fun onServiceDisconnected(name: ComponentName?) {
            aidlService = null
            Toast.makeText(applicationContext, "Service Disconnected", Toast.LENGTH_LONG).show()
        }
    }
    ```
    
2. Connect to AIDL service via Binders.
    
    ```kotlin
    val serviceIntent = Intent()
    serviceIntent.component = ComponentName(BuildConfig.AIDL_PACKAGE, BuildConfig.AIDL_SERVICE)
    bindService(serviceIntent, serviceConnection, BIND_AUTO_CREATE)
    ```
    
3. Call methods of AIDL after successful connection:
    
    ```kotlin
    aidlService?.add(2, 2)?.let {
        Toast.makeText(
            applicationContext,
            String.format(Locale.getDefault(), "Sum is: %d", it),
            Toast.LENGTH_LONG
        ).show()
    }
    ```
    

## Limitations

There is one BIG gotcha with AIDL services, which is when the system is in doze mode, your service app might not start to respond to requests.

---

Here is an example application using AIDL in real life 😎

%[https://github.com/tejpratap46/AIDL-Example]