EasyService
===========

**EasyService** is a .NET library intended to simplify development and maintenance of Windows services. With just a few lines of code you will get a complete Windows service that can be debugged using the debugger and installed by running ```service.exe install```.

Benefits
--

- Focus on your service logic instead of learning details of .NET Service API and InstallUtil.
- Transform any console application into a self-installing Windows service.
- Painless debugging. It is much easier to debug a console application than a service. 
- Access all startup and service recovery settings not supported by .NET framework.

Quick Start
--

1. Create a new console application project.
2. Reference ```EasyService.dll```.
3. Copy example code shown below to your project's ```Program.cs```.
4. Change ```// pretend to work hard``` line to do something useful... or not.
5. Build the project. 
6. Install and start your new service by executing ```service.exe install``` and ```service.exe start```.

Done!

```c#

using System;
using System.Threading;
using EasyService;

class MiniService : ServiceBase
{
    protected override void MainLoop(CancellationToken stopEvent)
    {
        try
        {
            // do some useful work here {

            while (true)
            {
                try
                {
                    // pretend to work hard
                    stopEvent.WaitHandle.WaitOne(5000);

                    // periodically check stop event
                    stopEvent.ThrowIfCancellationRequested();
                }
                catch (Exception)
                {
                    // recover from errors or just sleep for some time
                    if (stopEvent.WaitHandle.WaitOne(30000))
                        break;
                }
                finally
                {

                }
            }

            // } 
        }
        catch (OperationCanceledException)
        {
            // ignore
        }
    }
	
    static void Main(string[] args)
    {
        // Initialize your service
        var service = new MiniService();

        // Configure service hosting
        var settings = new ServiceHostingSettings
        {
            ServiceName = "MiniService",
            DisplayName = "Minimal Service",
            Description = "Minimal service description",
            StartMode = ServiceStartMode.Automatic,
            ServiceAccount = ServiceAccount.LocalSystem,
            RecoveryOptions = new ServiceRecoveryOptions
            {
                FirstFailureAction = ServiceRecoveryAction.RestartService,
                SecondFailureAction = ServiceRecoveryAction.RestartService,
                SubsequentFailureAction = ServiceRecoveryAction.RestartComputer,
                ResetFailureCountWaitDays = 1,
                RestartServiceWaitMinutes = 3,
                RestartSystemWaitMinutes = 3
            }
        };

        // Parse command-line options and execute
        ServiceHost.Run(service, settings, args);
    }
}


```

Command-line Reference
--

A Windows service built with **EasyService** supports command-line arguments which can be used to install, uninstall, start, and stop the service. If no command is provided the service will be started as a console application.

service.exe [command]

Command|Description
---------------------------------|-----
                                 |Runs the service as a console application
help                             |Displays help
install [account] [password]     |Installs the service. You can also set log-on account and password.
uninstall                        |Uninstalls the service
start                            |Starts the service if it is not already running
stop                             |Stops the service if it is running




