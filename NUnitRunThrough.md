# What is the NUnit TestLink Adapter? #
The test link adapter is an Addin for the NUnit unit testing framework. It exports test results into Testlink, which is a test management application.

# Introduction #

To make use of the NUnit TestLink Adapter (NTA), you must first add a new attribute to each TestFixture. This attribute, called TestLinkFixture takes parameters that tell the adapter where to find the test cases to record results to.
Typically the tests are run as part of a build script or a batch file. The NTA is an extension to the command line runner.
When the tests have been run, the extension is called with the test results.
The extension parses the results, loads the test assemblies that have been run and extracts the test link fixture. It then contacts Testlink to look for test cases with the same name as they test methods in the unit test.
If the test case does not exist, NTA will create them and assign them to the Testplan.
It then records the test result against that test case.
# Installation #
## Run Time Files ##
Copy all the DLLs into the AddIn directory of the NUnit folder:

## Invocation ##
In this example the console runner is used. However you can also use the NUNIT GUI:
```
Nunit-Console.exe [unitTestAssembly]
```
## TestLink Setup ##
In order for this to work you need to have in testlink:
  1. Enabled the API (see installation manual)
  1. Created an account  for testing
  1. Created an ApiKey for this account
  1. Setup a test project
  1. Set up test suite(s) (top level test suite) for the test cases
  1. In the test project set up a test plan
  1. Optionally created test cases in the test suite and assigned to the test plan
  1. Have an active build defined in the testplan

Step 7 really is optionally because:
  1. You must make sure you name the test cases exactly as they are defined in your unit tests
  1. If you use advanced features such as contract verifiers and row tests in MBUnit you have to guess what these test cases will be named because they are dynamically generated
  1. If you don’t create test cases, NTA will do it for you – much easier

## Unit Test setup ##
Write your test cases as you are used to.
Then have the project reference TestLinkFixture.dll and add a Using Statement for it.
Add the TestLinkFixture to your testfixture.
The parameters are:
  * URL of the TestLink Api
  * UserName under which the test cases are authored
  * An ApiKey (provided by the testlink application)
  * The test project name
  * The test plan in the test project
  * The test suite name that will contain the test cases

Example
```
using NUnit.Framework;
using Meyn.TestLink;

namespace NunitAddOnSampleTests
{
    [TestFixture]
    [TestLinkFixture(
        Url = "http://localhost/testlink/lib/api/xmlrpc.php",
        ProjectName = "TestLinkApi",
        UserId = "admin",
        TestPlan = "Automatic Testing",
        TestSuite = "nunitAddOnSampleTests",
        DevKey = "ae28ffa45712a041fa0b31dfacb75e29")]
    public class SampleFixture
    {
        [Test]
        public void Test2Succeed()
        {
            Assert.IsTrue(true);
        }
    }
}
```

Voila.
This will create a test case named Test2Succeed and record test case result against it.
## Limitations ##
This has been tested with NUnit V 2.6.2 & C# and Testlink V 1.9.3

It should work with VB.NET.

## Known Defect ##
The NUnit GUI WILL NOT list the addon in its add on pages. Turning on detailed logging reveals that the Addon is compiled with a newer version of .Net runtime.
However it still works.

You use this code at your own risk.


## Troubleshooting ##
If the add in does not appear to work you need to take a look at the
NUnit log out put:

  1. Open the NUnit GUI and select: Tools>Settings
  1. Under Advanced Settings > Internal Trace set Verbose
  1. Note the path for the log file in that tab. It is something like  `C:\Users\<username>\AppData\local\Nunit\Logs`
  1. Start Nunit.exe
  1. Check the log file called `nunit-agent_<nnnn>.log`

you should find something like this
```
00:43:32.505 Info  [ 1] ServiceManager: Initializing AddinManager
00:43:32.508 Debug [ 1] ServiceManager: Request for service IAddinRegistry satisfied by AddinRegistry
00:43:32.515 Debug [ 1] AddinManager: Loaded CookComputing.XmlRpcV2.dll
00:43:32.519 Debug [ 1] AddinManager: Loaded log4net.dll
00:43:32.529 Debug [ 1] AddinManager: Loaded TestLinkAPI.dll
00:43:32.531 Debug [ 1] AddinManager: Loaded TestLinkFixture.dll
00:43:32.532 Debug [ 1] AddinManager: Loaded TlNUnitAddOn.dll
00:43:32.534 Debug [ 1] AddinManager: Registered addin: TestLinkAddon
```

If you find something like this:
```
19:16:39.545 Error [ 1] AddinManager: Failed to loadC:\Program Files (x86)\NUnit 2.6.2\bin\addins\TestLinkAPI.dll
System.BadImageFormatException: Could not load file or assembly 'TestLinkAPI' or one of its dependencies. This assembly is built by a runtime newer than the currently loaded runtime and cannot be loaded.
File name: 'TestLinkAPI' ---> System.BadImageFormatException: Could not load file or assembly 'TestLinkAPI' or one of its dependencies. This assembly is built by a runtime newer than the currently loaded runtime and cannot be loaded.
File name: 'TestLinkAPI'
```

then most likely the version of NUnit is newer than the version that was used as a reference when the AddOn was compiled.

In this case you need to get the sources, open them in VS 2010 Express or VS 2010, point the references for NUnit to the version you are using and recompile.