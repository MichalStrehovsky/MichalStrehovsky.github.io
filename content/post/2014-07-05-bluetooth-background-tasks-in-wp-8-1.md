---
title: Bluetooth background tasks in WP 8.1
date: 2014-07-05T18:44:11+00:00
tags:
  - programming
---
This weekend I was working on a small Windows Phone project to scratch one of my itches. I wanted to create an app that would connect to a ELM327 Bluetooth adapter in my car whenever the adapter is within my phone&#8217;s reach and do something with the adapter in the background. The Bluetooth adapter exposes itself as a simple RFCOMM COM port.

I thought I should write down the steps to accomplish this (mostly for my own future reference). I chose to do this with a C# WinRT app. It might work with Silverlight, but I&#8217;m not sure about that. You need to target Windows Phone 8.1 since most of these APIs are new to Phone 8.1.

### Manifest

First things first &#8211; the Package.appxmanifest changes. The app needs to declare the RFCOMM serial port capability and the background task:

```xml
<Package ...>
  <Applications>
    <Application ...>
      ...
      <Extensions>
        <Extension Category="windows.backgroundTasks"
                   EntryPoint="ObdGasHelperBackground.ObdTask">
          <BackgroundTasks>
            <m3:Task Type="rfcommConnection"/>
          </BackgroundTasks>
        </Extension>
      </Extensions>
    </Application>
  </Applications>
  <Capabilities>
    <m2:DeviceCapability Name="bluetooth.rfcomm">
      <m2:Device Id="any">
        <m2:Function Type="name:serialPort" />
      </m2:Device>
    </m2:DeviceCapability>
  </Capabilities>
</Package>
```

The EntryPoint part specifies the full name of the WinRT class implementing the background tasks (more about it later).

### Enumerating RFCOMM devices

To enumerate the RFCOMM serial port services of all paired Bluetooth devices, use this snippet:

```csharp
foreach (var d in await DeviceInformation.FindAllAsync(
         RfcommDeviceService.GetDeviceSelector(RfcommServiceId.SerialPort)))
{
    RfcommDeviceService service = await RfcommDeviceService.FromIdAsync(d.Id);
}
```

Note I said paired &#8211; if the device is not paired to the phone yet, it will not show up even if it&#8217;s within reach. It might be a good idea to show the user a button that launches the Bluetooth settings system dialog to let them pair new devices. Also note another implication of this &#8211; a paired device will show up even if it&#8217;s not within reach. This is usually a good thing.

The service object has a few interesting properties that can be shown in the UI to determine what device does it represent. Intellisense is your friend.

### Registering a background task

Once we have found a device service to out liking (and maybe let the user pick it in the UI), we can register a background task to fire every time the phone sees the device service being advertised.

We need to start by requesting background access:

```csharp
if (await BackgroundExecutionManager.RequestAccessAsync() == BackgroundAccessStatus.Denied)
{
    // Uh oh, we can't create background tasks. Maybe the limit was reached or the user declined.
}
```

With that out of our way, we can create the background task with this code. The local variable &#8220;service&#8221; is the service we discovered above.

```csharp
string taskName = service.ConnectionHostName.CanonicalName;

// Only register the task if it's not registered yet
if (BackgroundTaskRegistration.AllTasks.Values.SingleOrDefault(x => x.Name == taskName) == null)
{
    RfcommConnectionTrigger trigger = new RfcommConnectionTrigger
    {
        RemoteHostName = service.ConnectionHostName,
    };
    trigger.OutboundConnection.RemoteServiceId = service.ServiceId;

    BackgroundTaskBuilder builder = new BackgroundTaskBuilder
    {
        Name = taskName,
        TaskEntryPoint = "ObdGasHelperBackground.ObdTask",
    };
    builder.SetTrigger(trigger);
    builder.Register();
}
```

Again, the TaskEntryPoint is the full name of the WinRT class implementing the background task that we haven&#8217;t implemented yet.

### The background task

Background tasks have to be hosted in a separate WinMD file. It can&#8217;t be in your main executable. Also, you need to create a project reference from your main executable to the background task WinMD even though you technically don&#8217;t need that for the app to compile. This part is really important. If you forget it, your app will just force close without any diagnostic information when the task fires. Don&#8217;t forget to do this.

The code in the background task is really simple. The OS already opened a socket for us, so all we need to do is read and write into it:

```csharp
namespace ObdGasHelperBackground
{
    public sealed class ObdTask : IBackgroundTask
    {
        public async void Run(IBackgroundTaskInstance taskInstance)
        {
            var def = taskInstance.GetDeferral();

            var details = taskInstance.TriggerDetails as RfcommConnectionTriggerDetails;
            DataWriter wr = new DataWriter(details.Socket.OutputStream);
            DataReader rd = new DataReader(details.Socket.InputStream);

            // Party on!

            def.Complete();
        }
    }
}
```
