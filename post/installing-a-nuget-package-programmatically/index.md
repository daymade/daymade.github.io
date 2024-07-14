
<!--more-->

## Steps
```C#
// dotnetcore
var proc1 = new Process()
{
    StartInfo = new ProcessStartInfo()
    {
        FileName = "dotnet",
        Arguments = $@"add ""{activeProject.FullName}"" package ""{packageId}"""
    },
};
proc1.Start();
proc1.WaitForExit();
```

```C#
// traditional dotnet framework projects
private bool InstallNuGetPackage(Project project, string package)
{
    bool installedPkg = true;
    var dte = Package.GetGlobalService(typeof(DTE)) as DTE;
    try
    {
        var componentModel = (IComponentModel)Package.GetGlobalService(typeof(SComponentModel));
        IVsPackageInstallerServices installerServices = componentModel.GetService<IVsPackageInstallerServices>();
        if (!installerServices.IsPackageInstalled(project, package))
        {
            dte.StatusBar.Text = @"Installing " + package + " NuGet package, this may take a minute...";

            var installer = componentModel.GetService<IVsPackageInstaller2>();
            installer.InstallPackage(null, project, package, (System.Version)null, false);
            dte.StatusBar.Text = @"Finished installing the " + package + " NuGet package";
        }
    }
    catch (Exception ex)
    {
        installedPkg = false;
        dte.StatusBar.Text = @"Unable to install the  " + package + " NuGet package";
    }
    return installedPkg;
}
```

## Reference

* https://docs.microsoft.com/en-us/nuget/visual-studio-extensibility/nuget-api-in-visual-studio (official )
* http://tylerhughes.info/archive/2015/05/06/installing-a-nuget-package-programmatically/ (not working)
