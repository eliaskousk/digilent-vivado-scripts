# Digilent Vivado Scripts

## Introduction
This repository contains a set of scripts for creating, maintaining, and releasing git repositories containing minimally version-controlled Vivado and Xilinx SDK projects. These scripts have only been tested with Vivado 2018.2; they may or may not work with newer or older versions of Vivado. A Python 3.6.3 (or newer) installation is required to use these scripts. As of time of writing, no additional Python modules 

----------------
## Python Frontend
A front-end script, git_vivado.py, is provided to parse command line arguments and call into Vivado at the command line.  This script has three subcommands: "checkout", "checkin", and "release". Each of these subcommands has its own help menu, which explains the arguments that can be passed to the script, as well as show what the default values of each of these arguments will be. All paths passed to the script are assumed to be relative to the current working directory.

**Note**: *Each script can instead be manually sourced within the TCL console in Vivado. When doing so, take care to properly set the `argv` variable, as described in each scripts' Example Usage subsection*

--------------
## Commands / Scripts
### Checkout
#### Description
This subcommand calls into digilent_vivado_checkout.tcl in order to create a Vivado project, in the form of an XPR file, using the sources and scripts contained in the project repository. If a hardware handoff file and SDK projects are present in the repository, the SDK workspace is initialized in the specified workspace directory.
#### Optional Arguments
1. `-r <repo>`: Path to the repository directory. Default: `<digilent-vivado-scripts>/..`
1. `-x <xpr>`: Path to the project .xpr file the repo is to be checked out into. Default: `<repo>/proj/<repo name>.xpr`
1. `-w <workspace>`: Path to the directory to be used as the SDK workspace. Default: `<repo>/sdk`
1. `-v <version>`: Vivado version number. Default contained in config.ini. (Python only)
#### Example Usage
##### Python:
> <code>python git_vivado.py checkin -r D:/Github/Zybo-Z7-10-HDMI</code>

##### TCL:
> <code>set argv "-r D:/Github/Zybo-Z7-10-HDMI"</code>

> <code>source digilent-vivado-checkout.tcl</code>

-----------
### Checkin
#### Description
This subcommand calls into digilent_vivado_checkin.tcl in order to collect sources and generate needed scripts from a Vivado project into the repository structure described below. Files required for checkout *that are not already present* in the repository (such as project_info.tcl and gitignores), are automatically created. These files are not overwritten if they already exist.
#### Optional Arguments
1. `-r <repo>`: Path to the repository directory. Default: `<digilent-vivado-scripts>/..`
1. `-x <xpr>`: Path to the project .xpr file to be processed for checkin. Default: `<repo>/proj/*.xpr`
1. `-w <workspace>`: Path to the SDK workspace associated with the project. If non-default, the workspace is copied to `<repo>/sdk`. Default: `<repo>/sdk`
1. `-no_hdf`: Flag used to prevent overwriting of the hardware handoff. If this flag is not present and the bitstream is up-to-date, the hardware handoff file is checked in to `<repo>/hw_handoff`
1. `-v <version>`: Vivado version number. Default contained in config.ini. (Python only)
#### Example Usage
##### Python:
> <code>python git_vivado.py checkin -r D:/Github/Zybo-Z7-10-HDMI</code>

##### TCL:
> <code>set argv "-r D:/Github/Zybo-Z7-10-HDMI"</code>

> <code>source digilent-vivado-checkin.tcl</code>

-----------
### Release
#### Description
This subcommand collects all required files and produces a release ZIP archive. It cannot be run from the TCL Console, as Python is required to manage zipping and unzipping of files. A release consists of the repo's README, an archived Vivado project, with XPR and all dependencies, and the SDK workspace. The Vivado project must have an up-to-date bitstream in order for the release to be generated.
#### Optional Arguments
1. `-r <repo>`: Path to the repository directory. Default: `<digilent-vivado-scripts>/..`
1. `-x <xpr>`: Path to the project .xpr file to be released. Default: `<repo>/proj/*.xpr`
1. `-w <workspace>`: Path to the SDK workspace associated with the project. Default: `<repo>/sdk`
1. `-z <archive>`: Path where the resulting ZIP file will be placed. Default: `<repo>/release/<repo name>-<version>-#.zip`
1. `-temp <temp_dir>`: Path to a safe place to put temporary files. Default: `<repo>/release/temp`
1. `-v <version>`: Vivado version number. Default contained in config.ini. (Python only)
#### Example Usage
##### Python:
> <code>python git_vivado.py release -r D:/Github/Zybo-Z7-10-HDMI</code>

##### TCL:
> <code>set argv "-r D:/Github/Zybo-Z7-10-HDMI"</code>

> <code>source digilent-vivado-release.tcl</code>

------------------------------------
## Other Files and Overall Structure
### Configuration File
The digilent-vivado-scripts repository contains a file named "config.ini". This file contains several values used by the Python frontend to determine what the default arguments for the different subcommands should be. It has not yet been decided how this file should be managed/version-controlled. See "Known Issues" at the bottom of this document for a little more information.

### Repository Structure
In order to ensure that any changes to this repository do not break the projects that use them, it is expected that this repository will be used as a submodule of each project repository that is intended to use them.

* **\<project repo\>/digilent-vivado-scripts**: Contains the scripts described by this document.
* **\<project repo\>/hw_handoff**: Used to contain the hardware handoff.
* **\<project repo\>/proj**: Used to contain a checked-out Vivado project.
* **\<project repo\>/release**: Contains temporary files necessary to generate a release zip archive.
* **\<project repo\>/repo**: Contains local IP, IP submodules, and cached generated sources.
* **\<project repo\>/sdk**: Contains SDK projects.
  * **\<repo\>/sdk/.gitignore**: File describing which SDK files should be version controlled. Template generated by the checkin script.
  * **\<repo\>/sdk/\<app\>**: 
    * **\<repo\>/sdk/\<app\>/src/\***: 
    * **\<repo\>/sdk/\<app\>/.cproject**: 
    * **\<repo\>/sdk/\<app\>/.project**: 
  * **\<repo\>/sdk/\<bsp\>**: 
    * **\<repo\>/sdk/\<bsp\>/\*.mss**: 
    * **\<repo\>/sdk/\<bsp\>/.cproject**: 
    * **\<repo\>/sdk/\<bsp\>/.project**: 
    * **\<repo\>/sdk/\<bsp\>/.sdkproject**: 
    * **\<repo\>/sdk/\<bsp\>/Makefile**: 
  * **\<repo\>/sdk/\<hw\>**: 
    * **\<repo\>/sdk/\<hw\>/\*.bit**: 
    * **\<repo\>/sdk/\<hw\>/\*.hdf**: 
    * **\<repo\>/sdk/\<hw\>/.project**: 
* **\<project repo\>/src**: Contains source files for the Vivado Project.
  * **\<project repo\>/src/bd**: Contains a TCL script used to re-create a block design.
  * **\<project repo\>/src/constraints**: Contains XDC constraint files.
  * **\<project repo\>/src/hdl**: Contains Verilog and VHDL source files.
  * **\<project repo\>/src/ip**: Contains XCI files describing IP to be instantiated in non-IPI projects.
  * **\<project repo\>/src/others**: Contains all other required sources, such as memory initialization files.
* **\<project repo\>/.gitignore**: File describing which sources should be version controlled. Template is generated by the checkin process.
* **\<project repo\>/.gitmodules**: File describing submodules of the repository. Automatically maintained by the "git submodule" command.
* **\<project repo\>/project_info.tcl**: Script generated by first-time checkin used to save and re-apply project settings like board/part values. This can be modified after initial creation to manually configure settings that are not initially supported. **Note**: *This script should be deleted and recreated when porting a project from one board to another.*
* **\<project repo\>/README.md**: Markdown file describing the project and the process needed to use it, from downloading the release archive, to programming the FPGA.

------------
## Workflows
### 1. Creating a New Project
1. Create a new Vivado project at an arbitrary location on your computer. When exporting to and launching SDK, make sure to use exported locations and workspaces that are not "Local to Project".
2. Create a repository on Github for your project. Use the naming convention "\<board\>-\<variant number\>-\<project name\>" (for example: "Zybo-Z7-20-DMA"). Do not have Github create a gitignore file for you. Clone the repository.
3. In a command line interface (git bash is recommended) cd into the repository directory. Call "git submodule add https://github.com/artvvb/digilent-vivado-scripts" (FIXME).
4. Call "python ./digilent_vivado_scripts/git_vivado.py checkin -x \<path to XPR\>". This command can be called from anywhere in your filesystem, with relative paths changed as required. This will also create a gitignore file for the repository.
5. If required, in Xilinx SDK, right click on your application project's src folder and select "Export". Use this to copy all of the sources from "\<app\>/src" to "\<project repo\>/sdk/appsrc".
6. Create a README for the repo that specifies what the project is supposed to do and how to use a release archive for it.
7. Add, commit, and push your changes.
8. Create and upload a release ZIP to Github - see "Creating a Release Archive" below.

### 2. Retargeting an Existing Project to use these Scripts
1. Clone (or pull) the Vivado project to be retargeted. Use its existing version control system to generate an XPR. If relevant, make sure to call "git submodule init" followed by "git submodule update" from a command line interface (git bash is recommended for Windows) from within the repo directory.
2. Open the project in Vivado and make any changes necessary (perhaps upgrading IP). When exporting to and launching SDK, make sure to use exported locations and workspaces that are not "Local to Project".
3. Clone the project repository again in a different location. Remove all files from this directory.
4. In a command line interface (git bash) cd into the clean repository directory. Call "git submodule add https://github.com/Digilent/digilent-vivado-scripts".
5. Call "python ./digilent_vivado_scripts/git_vivado.py checkin -x \<path to XPR\>". This command can be called from anywhere in your filesystem, with relative paths changed as required. This will also create a gitignore file for the repository.
6. If required, in Xilinx SDK, right click on your application project's src folder and select "Export". Use this to copy all of the sources from "\<app\>/src" to "\<second repo\>/sdk/appsrc".
7. Create a README for the repo that specifies what the project is supposed to do and how to use a release archive for it.
8. Add, commit, and push your changes.
9. Create and upload a release ZIP to Github - see "Creating a Release Archive" below.

### 3. Making Changes to a Project that uses this Submodule
1. Clone (or pull) the Vivado project to be changed.
2. In a command line interface (git bash is recommended for Windows) cd into the repository directory.
3. Call "git submodule init" followed by "git submodule update" in order to get this repositories scripts scripts.
4. Call "python ./digilent_vivado_scripts/git_vivado.py checkout". This command can be called from anywhere in your filesystem, with the relative path to git_vivado. This will also create a gitignore file for the repository. Default arguments will create the XPR at "\<project repo\>/proj/\<project name\>.xpr".
5. Open the project in Vivado and make any changes necessary (perhaps upgrading IP or fixing a bug). When exporting to and launching SDK, make sure to use exported locations and workspaces that are not "Local to Project".
6. Call "python ./digilent_vivado_scripts/git_vivado.py checkin". This command can be called from anywhere in your filesystem, with the relative path to git_vivado changed as required.
7. If required, in Xilinx SDK, right click on your application project's src folder and select "Export". Use this to copy all of the sources from "\<app\>/src" to "\<project repo\>/sdk/appsrc".
8. Make sure to update the repo's README as required.
9. Add, commit, and push your changes.
10. Create and upload a release ZIP to Github - see "Creating a Release Archive" below.

### 4. Creating a Release Archive 
1. With an open Vivado project, make sure that a bitstream has been generated.
2. Click File -> Project -> Archive. Fill in the fields in the prompt as per the table below then click "OK".

| Field                          | Value                    |
| ------------------------------ | ------------------------ |
| Archive name                   | vivado_proj              |
| Archive location               | \<project repo\>/release |
| Temporary location             | (Default)                |
| Include configuration settings | unchecked                |
| Include run results            | checked                  |
| Include local IP cache results | checked                  |

3. Navigate to the release subdirectory of the project repo and extract vivado_proj.xpr.zip. Rename the resulting folder "vivado_proj" then delete vivado_proj.xpr.zip.
4. If required, in Xilinx SDK, right click on your application project's src folder and select "Export". Use this to copy all of the sources from "\<app\>/src" to "\<project_repo\>/release/sdk_appsrc".
5. Copy README.md into the release subdirectory, and make sure that it contains project specific instructions for using the release, especially if any libraries need to be added to the BSP or if compiler settings need to be added to the application project in SDK.
6. Compress the release subdirectory into a ZIP archive called "\<project name\>-\<release version number\>.zip" (for example "Zybo-Z7-20-DMA-2018.2-3.zip").
7. Draft a new release on Github and upload the ZIP to it.

### 5. Using a Release Archive
1. Download and extract the most recent release archive from the repository's releases page.
2. Open the XPR file in the version of Vivado specified by the README.
#### Vivado
3. Use Vivado's Hardware manager to connect to and program the target FPGA board with the project's bitstream.
#### Xilinx SDK
3. If the release archive contains an sdk_appsrc folder, export the project's hardware to SDK and launch SDK. Local to project is fine in this instance.
4. Create a new application project (Empty Application template) and BSP, and follow any instructions in the project's README for how to configure them.
5. Right click on the application project's src folder and select "Import". Import all files from \<extracted location\>/sdk_appsrc, overwriting anything that already exists.
6. Program and Run the application on the target FPGA board.

## Known Issues
* Each developer needs their own version of the configuration file, config.ini, for each project they are woking on. The configuration file should be moved to somewhere outside of the repo submodule to accomodate this, in a predictable location on Linux and Windows. The python script will need to be updated to accomodate this, a location and less generic name will need to be chosen for the configuration file.
* The "archive project" functionality of Vivado may include local SDK sources in the release archive. This should be avoided so that users have a clean workspace to Export and Launch SDK into. The current process requires that the project's SDK workspace is created external to the Vivado project.
* The process of creating a release archive currently must be done manually. This process can be automated further, but will require creation of an XSCT script to collect SDK application sources.
* There is some danger that modifications to Digilent's board files may break existing projects, it may be worth considering adding the vivado-boards repository as a submodule to project repositories.
