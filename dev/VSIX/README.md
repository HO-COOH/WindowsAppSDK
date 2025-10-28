# Windows App SDK Visual Studio Extension

This repository contains the Visual Studio 2022 extension that provides project and item templates for [Windows App SDK](https://github.com/microsoft/WindowsAppSDK) (WinUI 3) development. The extension enables developers to quickly create Windows desktop applications using either C++/WinRT or C#.

## 📋 Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [Development Guide](#development-guide)
- [Template Wizard System](#template-wizard-system)
- [Contributing](#contributing)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)

## Overview

This extension provides:
- **Project Templates**: Complete application templates that create fully-configured Windows App SDK projects
- **Item Templates**: Individual file templates (pages, controls, resources) that can be added to existing projects
- **Automated NuGet Package Installation**: Wizard-based system that ensures required packages are installed automatically
- **Multi-Language Support**: Separate extensions for C++ and C# with localized resources

### Key Features

- ✅ Visual Studio 2022 (Dev17) support
- ✅ Both C++/WinRT and C# language support
- ✅ Desktop and platform-agnostic templates
- ✅ Automatic NuGet package dependency installation
- ✅ Standalone and Component deployment modes
- ✅ Localized in 14 languages

## Project Structure

```
WindowsAppSDK.Extension.sln                    # Main solution file
│
├── Extension/                                 # VSIX extension packages
│   ├── Cpp/Dev17/                            # C++ extension for VS 2022
│   │   ├── WindowsAppSDK.Cpp.Extension.Dev17.csproj
│   │   ├── VSPackage.cs                      # Package registration and initialization
│   │   ├── Component/                        # Component mode deployment files
│   │   │   └── source.extension.vsixmanifest
│   │   └── Standalone/                       # Standalone mode deployment files
│   │       └── source.extension.vsixmanifest
│   │
│   └── Cs/Dev17/                             # C# extension for VS 2022
│       ├── WindowsAppSDK.Cs.Extension.Dev17.csproj
│       ├── VSPackage.cs
│       ├── Component/
│       │   └── source.extension.vsixmanifest
│       └── Standalone/
│           └── source.extension.vsixmanifest
│
├── ProjectTemplates/                          # Full project templates
│   ├── Desktop/                              # Desktop-specific applications
│   │   ├── CppWinRT/
│   │   │   ├── PackagedApp/                  # MSIX-packaged C++ app (separate packaging project)
│   │   │   ├── SingleProjectPackagedApp/     # Self-contained MSIX C++ app
│   │   │   └── UnitTestApp/                  # C++ unit test project
│   │   └── CSharp/
│   │       ├── ClassLibrary/                 # C# class library
│   │       ├── PackagedApp/                  # MSIX-packaged C# app (separate packaging project)
│   │       ├── SingleProjectPackagedApp/     # Self-contained MSIX C# app
│   │       └── UnitTestApp/                  # C# unit test project
│   │
│   └── Neutral/                              # Platform-agnostic projects
│       └── CppWinRT/
│           └── RuntimeComponent/             # Windows Runtime Component
│
├── ItemTemplates/                             # Individual file templates
│   ├── Desktop/                              # Desktop-specific items
│   │   ├── CppWinRT/
│   │   │   └── BlankWindow/                  # New window for desktop apps
│   │   └── CSharp/
│   │       └── BlankWindow/
│   │
│   └── Neutral/                              # Platform-agnostic items
│       ├── CppWinRT/
│       │   ├── BlankPage/                    # Empty XAML page
│       │   ├── UserControl/                  # Reusable user control
│       │   ├── TemplatedControl/             # Custom control with template
│       │   ├── ResourceDictionary/           # XAML resource dictionary
│       │   └── Resw/                         # String resource file
│       └── CSharp/
│           ├── BlankPage/
│           ├── UserControl/
│           ├── TemplatedControl/
│           ├── ResourceDictionary/
│           └── Resw/
│
├── Shared/                                    # Common code across extensions
│   ├── WizardImplementation.cs               # Core NuGet package installation logic
│   ├── WizardInfoBarEvents.cs                # InfoBar UI event handlers
│   └── OutputWindowHelper.cs                 # VS Output window utilities
│
├── BuildOutput/                               # Build artifacts and intermediate files
│   └── obj/                                  # Platform-specific outputs (AnyCPU, x86, x64, ARM64)
│
├── Directory.Build.props                      # Shared MSBuild properties
├── extension.manifest.json                    # Extension marketplace metadata
└── global.json                               # .NET SDK version specification
```

### Directory Organization Philosophy

| Folder | Purpose |
|--------|---------|
| **Desktop** | Templates for traditional desktop applications (uses Win32 window hosting) |
| **Neutral** | Platform-agnostic templates that work in both desktop and UWP contexts |
| **CppWinRT** | C++ language projection for Windows Runtime APIs |
| **CSharp** | C# language templates |
| **Component** | Extension deployment bundled with Visual Studio |
| **Standalone** | Extension deployment via Marketplace or manual VSIX install |

## Architecture

### Visual Studio Extension Basics

This project creates **VSIX packages** (Visual Studio Extension) that integrate with Visual Studio 2022. Key concepts:

#### AsyncPackage
- Base class for VS packages with asynchronous initialization
- Prevents blocking the UI thread during extension startup
- See `Extension/*/Dev17/VSPackage.cs` for implementation

#### VSTemplate Format
- XML-based format defining templates (`.vstemplate` files)
- Contains metadata, file structure, and wizard associations
- Example: `ProjectTemplates/Desktop/CSharp/PackagedApp/*.vstemplate`

#### IWizard Interface
- Enables custom logic during project/item creation
- Our implementation installs NuGet packages automatically
- Implemented in `Shared/WizardImplementation.cs`

#### VSIX Container
- ZIP-based package format (`.vsix`)
- Contains assemblies, templates, assets, and manifests
- Built automatically by the VSSDK build tools

#### Deployment Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **Standalone** | Distributed via VS Marketplace or manual install | Public releases, developer downloads |
| **Component** | Bundled with Visual Studio installation | Microsoft-managed VS components |

The deployment mode is controlled by the `$(Deployment)` MSBuild property in `Directory.Build.props` (defaults to `Standalone`).

### Why Separate C++ and C# Extensions?

The solution builds two separate VSIX files for architectural and practical reasons:

1. **Template Set Isolation**: C++ developers only see C++/WinRT templates, C# developers only see C# templates
2. **Dependency Management**: Different SDK requirements (C++/WinRT vs .NET)
3. **Installation Size**: Users only install what they need
4. **Marketplace Discovery**: Separate listings improve discoverability by language preference

## Getting Started

### Prerequisites

#### Required Software
- **Visual Studio 2022** (17.0 or later)
  - Workload: **Visual Studio extension development** (includes VSSDK)
  - Workload: **Desktop development with C++** (for C++ templates)
  - Workload: **.NET desktop development** (for C# templates)
- **Windows SDK** 10.0.26100 or later
- **.NET Framework 4.7.2** (for extension projects)
- **.NET SDK 3.0** (specified in `global.json`)

#### Recommended Knowledge
- C# (for extension development)
- MSBuild and .csproj file structure
- Visual Studio extensibility concepts
- Windows App SDK / WinUI 3 fundamentals
- NuGet package management

### Building the Solution

1. **Clone the repository**
   ```powershell
   git clone https://github.com/microsoft/WindowsAppSDK.git
   cd WindowsAppSDK/dev/VSIX
   ```

2. **Open the solution**
   ```powershell
   # Using Visual Studio
   .\WindowsAppSDK.Extension.sln
   
   # Or via command line
   devenv .\WindowsAppSDK.Extension.sln
   ```

3. **Restore NuGet packages**
   ```powershell
   # Visual Studio does this automatically, or manually:
   dotnet restore
   ```

4. **Build the solution**
   ```powershell
   # In Visual Studio: Ctrl+Shift+B
   # Or via MSBuild:
   msbuild WindowsAppSDK.Extension.sln /p:Configuration=Debug /p:Platform="Any CPU"
   ```

   Build outputs:
   - VSIX packages: `BuildOutput/obj/AnyCPUDebug/Standalone/WindowsAppSDK.*.Extension.Dev17/`
   - Templates: Packaged inside the VSIX files

### Configuration Options

Edit `Directory.Build.props` to customize build behavior:

```xml
<PropertyGroup>
  <!-- Switch between Standalone and Component deployment -->
  <Deployment>Standalone</Deployment>
  
  <!-- Control automatic deployment to experimental instance -->
  <DeployExtension>false</DeployExtension>
  
  <!-- Specify Windows App SDK version for templates -->
  <WindowsAppSdkVersion>1.5.0</WindowsAppSdkVersion>
</PropertyGroup>
```

## Development Guide

### Testing the Extension Locally

#### Using the Visual Studio Experimental Instance

Visual Studio provides an isolated "Experimental Instance" for testing extensions without affecting your main installation:

1. **Set up debugging**
   - Open solution in Visual Studio
   - Set either `WindowsAppSDK.Cpp.Extension.Dev17` or `WindowsAppSDK.Cs.Extension.Dev17` as the startup project
   - Press **F5** to launch the experimental instance

2. **The experimental instance will open** with your extension installed
   - All templates are available in the New Project dialog
   - Test template creation, wizard behavior, and NuGet installation

3. **Debugging tips**
   - Set breakpoints in `WizardImplementation.cs` to debug wizard logic
   - Use `System.Diagnostics.Debug.WriteLine()` for diagnostic output
   - Check the Output window (Debug pane) for messages
   - Review `%LOCALAPPDATA%\Microsoft\VisualStudio\17.0_<id>Exp\ActivityLog.xml` for errors

#### Manual VSIX Installation

For testing the packaged VSIX:

```powershell
# Navigate to build output
cd BuildOutput\obj\AnyCPUDebug\Standalone\WindowsAppSDK.Cs.Extension.Dev17\

# Install the VSIX (adjust path as needed)
.\WindowsAppSDK.Cs.Extension.Dev17.Standalone.vsix
```

Uninstall via: **Extensions > Manage Extensions > Installed**

### Adding a New Project Template

Follow these steps to add a new project template (example: C# "Empty App"):

#### 1. Create the Template Project

```powershell
# Create new template project folder
mkdir ProjectTemplates\Desktop\CSharp\EmptyApp
cd ProjectTemplates\Desktop\CSharp\EmptyApp
```

#### 2. Create the `.csproj` File

Create `WinUI.Desktop.Cs.EmptyApp.csproj`:

```xml
<Project ToolsVersion="Current">
  <Import Project="Sdk.props" Sdk="Microsoft.NET.Sdk" />
  <PropertyGroup>
    <ProjectGuid>{GENERATE-NEW-GUID}</ProjectGuid>
    <Configuration>Debug</Configuration>
    <Platform>AnyCPU</Platform>
    <OutputType>Library</OutputType>
    <RootNamespace>WinUI.Desktop.Cs.EmptyApp</RootNamespace>
    <AssemblyName>WinUI.Desktop.Cs.EmptyApp</AssemblyName>
    <TargetFramework>net472</TargetFramework>
    <GeneratePackageOnBuild>True</GeneratePackageOnBuild>
    <IncludeBuildOutput>false</IncludeBuildOutput>
    <SuppressDependenciesWhenPacking>true</SuppressDependenciesWhenPacking>
    <IncludeContentInPack>true</IncludeContentInPack>
    <ContentTargetFolders>content</ContentTargetFolders>
  </PropertyGroup>
  <ItemGroup>
    <Content Include="**\*" Exclude="*.csproj;*.user;bin\**\*;obj\**\*" />
    <Compile Remove="**\*.cs" />
  </ItemGroup>
  <Import Project="Sdk.targets" Sdk="Microsoft.NET.Sdk" />
</Project>
```

#### 3. Create the `.vstemplate` File

Create `WinUI.Desktop.Cs.EmptyApp.vstemplate`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<VSTemplate Version="3.0.0" xmlns="http://schemas.microsoft.com/developer/vstemplate/2005" Type="Project">
  <TemplateData>
    <Name ID="1100" Package="FIXME-PACKAGEGUID" />
    <Description ID="1101" Package="FIXME-PACKAGEGUID" />
    <Icon>Icon.ico</Icon>
    <TemplateID>Microsoft.WinUI.Desktop.Cs.EmptyApp</TemplateID>
    <ProjectType>CSharp</ProjectType>
    <DefaultName>App</DefaultName>
    <ProvideDefaultName>true</ProvideDefaultName>
    <CreateNewFolder>true</CreateNewFolder>
    <CreateInPlace>true</CreateInPlace>
    <LanguageTag>csharp</LanguageTag>
    <PlatformTag>windows</PlatformTag>
    <ProjectTypeTag>desktop</ProjectTypeTag>
    <ProjectTypeTag>WinUI</ProjectTypeTag>
  </TemplateData>
  <TemplateContent>
    <Project File="EmptyApp.csproj" ReplaceParameters="true">
      <!-- Add your template files here -->
      <ProjectItem>App.xaml</ProjectItem>
      <ProjectItem>App.xaml.cs</ProjectItem>
      <ProjectItem>MainWindow.xaml</ProjectItem>
      <ProjectItem>MainWindow.xaml.cs</ProjectItem>
    </Project>
    <CustomParameters>
      <CustomParameter Name="$NuGetPackages$" Value="Microsoft.WindowsAppSDK;Microsoft.Windows.SDK.BuildTools"/>
    </CustomParameters>
  </TemplateContent>
  <WizardExtension>
    <Assembly>WindowsAppSDK.Cs.Extension.Dev17, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null</Assembly>
    <FullClassName>WindowsAppSDK.TemplateUtilities.NuGetPackageInstaller</FullClassName>
  </WizardExtension>
</VSTemplate>
```

#### 4. Add Template Files

Add your actual template files (`.cs`, `.xaml`, `.csproj`, etc.) with parameter replacement:

```csharp
// Example: MainWindow.xaml.cs
namespace $safeprojectname$
{
    public sealed partial class MainWindow : Window
    {
        public MainWindow()
        {
            this.InitializeComponent();
        }
    }
}
```

**Common Template Parameters:**
- `$projectname$` - User-specified project name
- `$safeprojectname$` - Project name with invalid chars removed
- `$guid1$` - GUID for use in code
- `$NuGetPackages$` - Custom parameter for wizard (semicolon-separated package IDs)

#### 5. Register the Template in the Extension

Edit `Extension/Cs/Dev17/WindowsAppSDK.Cs.Extension.Dev17.csproj` and add:

```xml
<ItemGroup>
  <ProjectReference Include="$(ProjectTemplatesDir)Desktop\CSharp\EmptyApp\WinUI.Desktop.Cs.EmptyApp.csproj">
    <Project>{YOUR-NEW-GUID}</Project>
    <Name>WinUI.Desktop.Cs.EmptyApp</Name>
    <VSIXSubPath>ProjectTemplates</VSIXSubPath>
    <ReferenceOutputAssembly>false</ReferenceOutputAssembly>
    <IncludeOutputGroupsInVSIX>TemplateProjectOutputGroup;</IncludeOutputGroupsInVSIX>
  </ProjectReference>
</ItemGroup>
```

#### 6. Add to Solution

Add the new `.csproj` to `WindowsAppSDK.Extension.sln` in the appropriate solution folder.

#### 7. Add Localized Strings

Add name/description strings to `Extension/Cs/Common/VSPackage.resx` and localized `.resx` files in language subfolders (cs-CZ, de-DE, etc.).

#### 8. Build and Test

```powershell
# Rebuild solution
msbuild WindowsAppSDK.Extension.sln /t:Rebuild /p:Configuration=Debug

# Press F5 to test in experimental instance
```

### Adding a New Item Template

Similar process but simpler (no project structure):

1. Create folder in `ItemTemplates/[Desktop|Neutral]/[CppWinRT|CSharp]/YourItemName/`
2. Create `.csproj`, `.vstemplate`, and template files
3. Add `<ProjectReference>` to the extension `.csproj`
4. Item templates typically don't need the NuGet wizard (packages already installed at project level)

### Template Naming Conventions

Follow these patterns for consistency:

| Component | Pattern | Example |
|-----------|---------|---------|
| Folder Name | `PascalCase` | `PackagedApp`, `BlankPage` |
| Project File | `WinUI.[Desktop\|Neutral].[Cs\|CppWinRT].TemplateName.csproj` | `WinUI.Desktop.Cs.PackagedApp.csproj` |
| VSTemplate File | Matches project name | `WinUI.Desktop.Cs.PackagedApp.vstemplate` |
| Template ID | `Microsoft.WinUI.[Context].[Lang].TemplateName` | `Microsoft.WinUI.Desktop.Cs.PackagedApp` |

### Localization

All user-visible strings should be localized:

1. **Add string resources** to `Extension/[Cpp|Cs]/Common/VSPackage.resx`
   - Template names (ID 1000-1999)
   - Template descriptions (ID 2000-2999)

2. **Localize to 14 languages**:
   - cs-CZ (Czech), de-DE (German), en-US (English), es-ES (Spanish)
   - fr-FR (French), it-IT (Italian), ja-JP (Japanese), ko-KR (Korean)
   - pl-PL (Polish), pt-BR (Portuguese), ru-RU (Russian), tr-TR (Turkish)
   - zh-CN (Chinese Simplified), zh-TW (Chinese Traditional)

3. **Reference in `.vstemplate`**:
   ```xml
   <Name ID="1000" Package="{PackageGuid}" />
   <Description ID="2000" Package="{PackageGuid}" />
   ```

## Template Wizard System

The template wizard automates NuGet package installation to ensure projects have the required dependencies for Windows App SDK.

### Why the Wizard is Necessary

Windows App SDK projects require specific NuGet packages:
- `Microsoft.WindowsAppSDK` - Core Windows App SDK runtime
- `Microsoft.Windows.SDK.BuildTools` - Build-time SDK projections
- `Microsoft.Windows.CppWinRT` - C++/WinRT language projection (C++ only)

Without these packages, projects won't compile. The wizard ensures they're installed automatically during project creation.

### Architecture Overview

```
Project Creation Flow
│
├─ User creates project from template
│   └─ Visual Studio processes .vstemplate file
│
├─ IWizard.RunStarted()
│   ├─ Parse $NuGetPackages$ parameter
│   ├─ Initialize VS services (IComponentModel, IVsPackageInstaller)
│   └─ Subscribe to SolutionRestoreFinished event
│
├─ IWizard.ProjectFinishedGenerating()
│   ├─ Detect project type (C++ vs C#)
│   │
│   ├─ [C++ Projects]
│   │   ├─ Install packages immediately with progress dialog
│   │   └─ Show errors via MessageBox if installation fails
│   │
│   └─ [C# Projects]
│       └─ Wait for solution restore (handled in event)
│
├─ IVsNuGetProjectUpdateEvents.SolutionRestoreFinished
│   ├─ [C# Projects Only]
│   ├─ Install packages after NuGet restore completes
│   └─ Show errors via InfoBar if installation fails
│
└─ IWizard.RunFinished()
    └─ [C++ Projects Only] Show final error summary if needed
```

### Key Implementation Details

#### Source File: `Shared/WizardImplementation.cs`

The `NuGetPackageInstaller` class implements `IWizard` and handles the entire lifecycle:

```csharp
public partial class NuGetPackageInstaller : IWizard
{
    // Project type detection
    internal static Guid SolutionVCProjectGuid = new Guid("8BC9CEB8-8B4A-11D0-8D11-00A0C91BC942");
    
    // Key services
    private IComponentModel _componentModel;           // VS service locator
    private IVsPackageInstaller _installer;           // NuGet package installer
    private IVsThreadedWaitDialog2 _waitDialog;       // Progress dialog
    private IVsNuGetProjectUpdateEvents _nugetEvents; // Restore completion events
    
    // State management
    private Project _project;                         // Current project
    private IEnumerable<string> _nuGetPackages;       // Packages to install
    private Dictionary<string, Exception> _failedPackageExceptions; // Error tracking
}
```

#### Why C++ and C# are Handled Differently

| Aspect | C++ Projects | C# Projects |
|--------|--------------|-------------|
| **Project System** | Legacy VC project system | New SDK-style project system |
| **NuGet Integration** | Limited, requires immediate install | Full support, uses restore system |
| **Installation Timing** | `ProjectFinishedGenerating()` | `SolutionRestoreFinished` event |
| **Progress UI** | Modal progress dialog (`IVsThreadedWaitDialog2`) | Non-blocking InfoBar |
| **Error Display** | MessageBox + Output window | InfoBar + Output window |

**Rationale**: C# projects have a restore phase that conflicts with immediate package installation. Installing packages before restore completes causes `InvalidOperationException`. C++ projects don't have this restore phase, so they can install immediately.

### Wizard Lifecycle Methods

#### 1. RunStarted()
```csharp
public void RunStarted(
    object automationObject, 
    Dictionary<string, string> replacementsDictionary, 
    WizardRunKind runKind, 
    object[] customParams)
```

**Called**: When user initiates project creation  
**Purpose**: Initialize services and parse template parameters

```csharp
// Parse NuGet packages from template
if (replacementsDictionary.TryGetValue("$NuGetPackages$", out string packages))
{
    _nuGetPackages = packages.Split(';').Where(p => !string.IsNullOrEmpty(p));
}

// Subscribe to restore events
_nugetProjectUpdateEvents.SolutionRestoreFinished += OnSolutionRestoreFinished;
```

#### 2. ProjectFinishedGenerating()
```csharp
public void ProjectFinishedGenerating(Project project)
```

**Called**: After project files are created on disk  
**Purpose**: Install packages for C++ projects

```csharp
_project = project;
Guid projectGuid = GetProjectGuid(project);

// Only handle C++ projects here
if (projectGuid.Equals(SolutionVCProjectGuid))
{
    ThreadHelper.JoinableTaskFactory.Run(async () =>
    {
        await InstallNuGetPackagesAsync();
    });
}
```

#### 3. OnSolutionRestoreFinished()
```csharp
private void OnSolutionRestoreFinished(IReadOnlyList<string> projects)
```

**Called**: After NuGet restore completes (event handler)  
**Purpose**: Install packages for C# projects

```csharp
await ThreadHelper.JoinableTaskFactory.SwitchToMainThreadAsync();

Guid projectGuid = GetProjectGuid(_project);

// Only handle non-C++ projects here
if (!projectGuid.Equals(SolutionVCProjectGuid))
{
    // Unsubscribe to prevent duplicate calls
    _nugetProjectUpdateEvents.SolutionRestoreFinished -= OnSolutionRestoreFinished;
    
    _ = joinableTaskFactory.RunAsync(InstallNuGetPackagesAsync);
}
```

#### 4. RunFinished()
```csharp
public void RunFinished()
```

**Called**: After all wizard operations complete  
**Purpose**: Show final error summary for C++ projects

```csharp
if (_projectGuid.Equals(SolutionVCProjectGuid) && _failedPackageExceptions.Count > 0)
{
    var errorMessage = CreateErrorMessage(ErrorMessageFormat.MessageBox);
    MessageBox.Show(errorMessage, "Missing Package Reference(s)", ...);
    ShowOutputWindow(CreateDetailedErrorMessage());
}
```

### Package Installation Process

#### Core Installation Logic

```csharp
private async Task InstallNuGetPackagesAsync()
{
    // Get NuGet installer service
    IVsPackageInstaller installer = _componentModel.GetService<IVsPackageInstaller>();
    
    // Show progress dialog (for C++ projects)
    if (_waitDialog != null)
    {
        _waitDialog.StartWaitDialog(null, "Installing NuGet packages into project...", ...);
    }
    
    // Install each package
    foreach (var packageId in _nuGetPackages)
    {
        try
        {
            await Task.Run(() => installer.InstallPackage(
                source: null,              // Use default sources
                project: _project,
                packageId: packageId,
                version: "",               // Use latest version
                ignoreDependencies: false
            ));
        }
        catch (Exception ex)
        {
            // Track failures for error reporting
            _failedPackageExceptions[packageId] = ex;
        }
    }
    
    // Close progress dialog
    if (_waitDialog != null)
    {
        _waitDialog.EndWaitDialog(out int canceled);
    }
    
    // Show errors if any packages failed
    if (_failedPackageExceptions.Count > 0)
    {
        await DisplayInfoBarAsync(CreateErrorMessage(ErrorMessageFormat.InfoBar));
    }
}
```

### Error Handling and Reporting

The wizard has a comprehensive error handling system:

#### Error Storage
```csharp
private Dictionary<string, Exception> _failedPackageExceptions = new Dictionary<string, Exception>();
```

Each failed package installation is tracked with its exception for detailed reporting.

#### InfoBar Integration (C# Projects)

When packages fail to install, an InfoBar appears at the top of Visual Studio:

```
[ProjectName] Unable to add references to the following packages: Package1, Package2. 
This is an environment error. Please add package references before building the project.

[Manage NuGet Packages] [See error details] [X]
```

**Implementation** (`WizardInfoBarEvents.cs`):

```csharp
public class NuGetInfoBarUIEvents : IVsInfoBarUIEvents
{
    public void OnActionItemClicked(IVsInfoBarUIElement infoBarUIElement, IVsInfoBarActionItem actionItem)
    {
        if (actionItem is InfoBarHyperlink hyperlink)
        {
            if (hyperlink.Text == "Manage NuGet Packages")
            {
                // Opens NuGet Package Manager
                dte?.ExecuteCommand("Project.ManageNuGetPackages");
            }
            else if (hyperlink.Text == "See error details")
            {
                // Shows Output window with detailed exception info
                OutputWindowHelper.ShowMessageInOutputWindow(_detailedErrorMessage);
            }
        }
    }
}
```

#### Output Window Integration

Detailed error information is written to the Visual Studio Output window (General pane):

```
Missing Package References for MyProject:
Microsoft.WindowsAppSDK - System.InvalidOperationException: Package is already being installed.
Microsoft.Windows.SDK.BuildTools - System.Net.WebException: Unable to connect to remote server.

Please manually add package references before building.
```

**Implementation** (`OutputWindowHelper.cs`):

```csharp
public static void ShowMessageInOutputWindow(string message, bool clearPane = true)
{
    ThreadHelper.ThrowIfNotOnUIThread();
    
    IVsOutputWindow outputWindow = ServiceProvider.GlobalProvider.GetService(typeof(SVsOutputWindow)) as IVsOutputWindow;
    Guid guidGeneral = Microsoft.VisualStudio.VSConstants.GUID_OutWindowGeneralPane;
    
    outputWindow.GetPane(ref guidGeneral, out IVsOutputWindowPane pane);
    pane.Activate();
    pane.OutputStringThreadSafe(message);
    
    // Show the Output window
    dte?.ExecuteCommand("View.Output");
}
```

### Template Parameter Configuration

Templates specify which packages to install via the `$NuGetPackages$` parameter:

```xml
<VSTemplate>
  <TemplateContent>
    <CustomParameters>
      <!-- Semicolon-separated list of package IDs -->
      <CustomParameter Name="$NuGetPackages$" Value="Microsoft.WindowsAppSDK;Microsoft.Windows.SDK.BuildTools"/>
    </CustomParameters>
  </TemplateContent>
  <WizardExtension>
    <Assembly>WindowsAppSDK.Cs.Extension.Dev17, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null</Assembly>
    <FullClassName>WindowsAppSDK.TemplateUtilities.NuGetPackageInstaller</FullClassName>
  </WizardExtension>
</VSTemplate>
```

**Rules**:
- Use semicolon (`;`) as separator
- Specify package ID only (version is auto-resolved to latest compatible)
- No spaces around semicolons
- Empty strings are filtered out

### Threading and Async Considerations

The wizard carefully manages threading to comply with Visual Studio's threading rules:

#### JoinableTaskFactory Pattern
```csharp
await ThreadHelper.JoinableTaskFactory.SwitchToMainThreadAsync();
```

This ensures UI-related operations run on the UI thread without blocking or causing deadlocks.

#### Thread-Safe Package Installation
```csharp
foreach (var packageId in _nuGetPackages)
{
    await Task.Run(() => installer.InstallPackage(...));
}
```

Package installation is offloaded to a background thread to prevent UI freezing.

### Debugging the Wizard

To debug wizard behavior:

1. **Set breakpoints** in `WizardImplementation.cs`:
   - `RunStarted()` - Check parameter parsing
   - `ProjectFinishedGenerating()` - Verify project type detection
   - `InstallNuGetPackagesAsync()` - Monitor installation process

2. **Enable diagnostic logging**:
   ```csharp
   System.Diagnostics.Debug.WriteLine($"Installing package: {packageId}");
   ```

3. **Check Activity Log** for VS errors:
   ```
   %LOCALAPPDATA%\Microsoft\VisualStudio\17.0_<id>Exp\ActivityLog.xml
   ```

4. **Test failure scenarios**:
   - Disconnect from internet (causes package download failure)
   - Use invalid package IDs
   - Test with/without NuGet.config

## Contributing

We welcome contributions! Here's how to get started:

### Workflow

1. **Fork the repository** on GitHub
2. **Create a feature branch**
   ```powershell
   git checkout -b feature/my-new-template
   ```
3. **Make your changes** following the guidelines below
4. **Test thoroughly** in the VS Experimental Instance
5. **Commit with clear messages**
   ```powershell
   git commit -m "Add C# Console App template for Windows App SDK"
   ```
6. **Push to your fork**
   ```powershell
   git push origin feature/my-new-template
   ```
7. **Open a Pull Request** to the main repository

### Contribution Guidelines

#### Code Style
- Follow existing C# coding conventions
- Use meaningful variable/method names
- Add XML doc comments for public APIs
- Keep methods focused and concise

#### Template Guidelines
- **Naming**: Follow existing naming conventions (see table above)
- **Structure**: Match the organization of similar templates
- **Parameters**: Use standard template parameters (`$projectname$`, `$safeprojectname$`, etc.)
- **Localization**: Add strings to all 14 language resource files
- **Testing**: Verify template creation, build, and run on all configurations

#### Pull Request Checklist
- [ ] Code builds without errors or warnings
- [ ] Template creates projects successfully
- [ ] Projects build and run correctly (Debug and Release)
- [ ] NuGet packages install automatically via wizard
- [ ] All localized strings are present
- [ ] Changes are documented (if adding features)
- [ ] No breaking changes to existing templates

### Testing Requirements

Before submitting a PR, verify:

1. **Template Creation**: Template appears in New Project dialog with correct icon/description
2. **Project Generation**: Creates all expected files with correct content
3. **Build Success**: Project builds for all target platforms (x86, x64, ARM64 where applicable)
4. **Runtime**: Application launches and functions correctly
5. **NuGet Wizard**: Packages install automatically, errors handled gracefully
6. **Multi-Language**: Test both C++ and C# if changes affect shared code

## Testing

### Unit Testing

Currently, this solution does not include automated unit tests. Manual testing in the VS Experimental Instance is required.

### Manual Testing Checklist

#### For New/Modified Project Templates
- [ ] Template appears in correct category (File > New > Project)
- [ ] Icon and description display correctly
- [ ] Project creates without errors
- [ ] All files are generated correctly
- [ ] NuGet packages install automatically
- [ ] Project builds successfully (Debug, Release)
- [ ] Project runs without errors
- [ ] Test on clean VM or container

#### For New/Modified Item Templates
- [ ] Template appears in correct category (Project > Add New Item)
- [ ] Item adds to project correctly
- [ ] Generated code compiles
- [ ] No duplicate files created

#### For Wizard Changes
- [ ] C++ projects install packages immediately
- [ ] C# projects install after restore
- [ ] Progress dialog shows for C++
- [ ] InfoBar shows for C# on failure
- [ ] MessageBox shows for C++ on failure
- [ ] Output window shows detailed errors
- [ ] "Manage NuGet Packages" link works
- [ ] No deadlocks or UI freezes

#### For Localization
- [ ] Test at least 3 languages (en-US, de-DE, ja-JP recommended)
- [ ] All strings display (no resource keys visible)
- [ ] Text fits in UI elements

### Testing in a Clean Environment

To ensure templates work for end users:

1. **Use Windows Sandbox** or **Hyper-V VM**
2. **Fresh VS 2022 installation** with minimal workloads
3. **Install your VSIX** manually
4. **Create project from template**
5. **Build and run** without modifications

This catches issues with:
- Missing dependencies
- Hardcoded paths
- Machine-specific configuration

## Troubleshooting

### Common Issues

#### Issue: "Could not obtain IVsPackageInstaller service"

**Symptoms**: Wizard fails silently, packages not installed

**Causes**:
- NuGet.VisualStudio package not referenced correctly
- VS services not initialized

**Solutions**:
1. Verify `NuGet.VisualStudio` PackageReference in extension `.csproj`
2. Check that `IComponentModel` is obtained successfully
3. Review ActivityLog.xml for service errors

---

#### Issue: "Package is already being installed" (InvalidOperationException)

**Symptoms**: Package installation fails with exception during project creation

**Causes**:
- Installing packages before NuGet restore completes (C# projects)
- Double-installation attempt

**Solutions**:
- Ensure C# projects only install in `OnSolutionRestoreFinished`
- Add guard check: `if (!projectGuid.Equals(SolutionVCProjectGuid))`
- Unsubscribe from event after first call

---

#### Issue: Templates don't appear in New Project dialog

**Symptoms**: Extension installs but templates are missing

**Causes**:
- Template not referenced in extension `.csproj`
- `.vstemplate` file has errors
- VSIX manifest doesn't include templates

**Solutions**:
1. Check `<ProjectReference>` in `WindowsAppSDK.*.Extension.Dev17.csproj`
2. Verify `<VSIXSubPath>` is set correctly
3. Validate `.vstemplate` against schema
4. Rebuild solution and reinstall VSIX
5. Clear VS template cache: Delete `%LOCALAPPDATA%\Microsoft\VisualStudio\17.0_<id>\ComponentModelCache`

---

#### Issue: "The template specified cannot be found"

**Symptoms**: Error when creating project from template

**Causes**:
- Template file path mismatch
- ProjectTemplateLink points to wrong file

**Solutions**:
1. Verify `ProjectTemplateLink` paths in `.vstemplate`
2. Ensure linked templates exist and have correct names
3. Check for typos in file references

---

#### Issue: Template parameter not replaced ($projectname$ appears literally)

**Symptoms**: Generated code contains `$projectname$` instead of actual name

**Causes**:
- `ReplaceParameters="true"` not set
- File not included in template

**Solutions**:
1. Add `ReplaceParameters="true"` to `<Project>` or `<ProjectItem>` in `.vstemplate`
2. Ensure file is listed in `<TemplateContent>`

---

#### Issue: Extension doesn't load in Experimental Instance

**Symptoms**: F5 debugging launches VS but extension not present

**Causes**:
- Extension previously disabled
- VSIX manifest errors
- Build failure

**Solutions**:
1. Check **Extensions > Manage Extensions > Installed** (enable if disabled)
2. Review build output for errors
3. Check `%LOCALAPPDATA%\Microsoft\VisualStudio\17.0_<id>Exp\Extensions\`
4. Reset Experimental Instance:
   ```powershell
   devenv /RootSuffix Exp /ResetSettings
   ```

### Logs and Diagnostics

| Log Location | Contains |
|--------------|----------|
| `%LOCALAPPDATA%\Microsoft\VisualStudio\17.0_<id>Exp\ActivityLog.xml` | VS initialization, extension loading, service errors |
| Visual Studio **Output** window (Debug pane) | `Debug.WriteLine()` output from extension code |
| Visual Studio **Output** window (General pane) | Wizard error details |
| **Extensions and Updates** > **Installed** | Extension load status |

### Getting Help

- **GitHub Issues**: [WindowsAppSDK Issues](https://github.com/microsoft/WindowsAppSDK/issues)
- **Discussions**: [WindowsAppSDK Discussions](https://github.com/microsoft/WindowsAppSDK/discussions)
- **Documentation**: [Windows App SDK Docs](https://docs.microsoft.com/windows/apps/windows-app-sdk/)

## License

This project is licensed under the MIT License - see the [LICENSE](Extension/LICENSE) file for details.

---

## Additional Resources

### Visual Studio Extensibility
- [Visual Studio SDK Documentation](https://docs.microsoft.com/visualstudio/extensibility/)
- [VSIX Project Templates](https://docs.microsoft.com/visualstudio/extensibility/creating-a-vsix-package)
- [Template Wizards](https://docs.microsoft.com/visualstudio/extensibility/how-to-use-wizards-with-project-templates)

### Windows App SDK
- [Windows App SDK GitHub](https://github.com/microsoft/WindowsAppSDK)
- [WinUI 3 Documentation](https://docs.microsoft.com/windows/apps/winui/winui3/)
- [C++/WinRT Documentation](https://docs.microsoft.com/windows/uwp/cpp-and-winrt-apis/)

### NuGet Integration
- [NuGet.VisualStudio API](https://docs.microsoft.com/nuget/visual-studio-extensibility/nuget-api-in-visual-studio)
- [IVsPackageInstaller Interface](https://docs.microsoft.com/dotnet/api/nuget.visualstudio.ivspackageinstaller)

---

**Maintained by**: Microsoft Windows App SDK Team  
**Last Updated**: October 2025
