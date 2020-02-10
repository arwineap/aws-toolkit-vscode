# Design Proposal: Local Debugging Experience for SAM Applications

Current Proposal Stage: General Experience

Future Proposal Stages (as future PRs):

-   General Experience In-depth
-   Architecture

Previous Proposal Stages:

-   None

## Overview

The Local Debugging features released in version 1.0 are limited, and have some design limitations. TODO Reference Issue. This proposal improves the user experience with additional ways to locally debug SAM Applications, and disambiguates some of the unspecified behaviors.

Users can Locally Debug SAM Applications in the following ways:

-   Debug Configurations - Launch a Debugging session using the Debug Panel in VS Code and pressing F5.
-   Local SAM Templates View - One UI Location to see and act on all SAM Applications / Functions
-   CodeLenses on Lambda Handlers - Locally run and debug a Lambda handler function without any SAM Template associations

### Terminology

TODO : Fill this section
CodeLens
Debug Configuration

    Debug Configurations are entries in a `launch.json` file that VS Code and Extensions use to define Debug sessions. TODO provide link to VS Code Debugging. By supporting Debug Configuration, users can press F5 (or the Debug button) to start a Debugging session.

SAM Template

## Debug Configurations

Debug Configurations allow users to press F5 (or the Debug button) to start a Debugging session. Debug Configurations of type `AWS-SAM-Local` can target a resource in a SAM Template, or directly target a Lambda handler in a code file. The Debug Configuration contains enough information to orchestrate a series of SAM CLI calls to build and invoke a SAM Application.

The Debug Configuration is only way to invoke the debugger. All of the local SAM debugging experiences build on this facility.

### Debug Configuration Variants

#### Debug Configurations that target a SAM Template & Resource

This experience is suitable for projects that have already defined their resources in a SAM Template.

When a Debug Configuration targets a resource in a SAM Template, it contains:

-   a path to a SAM Template file
-   the name of a resource within the template.

The following take place when this debug session is started:

-   the Debug Configuration is validated as follows. Failures prevent the debug session from proceeding:

    -   the SAM Template exists
    -   the Resource exists in the SAM Template
    -   the Resource's Runtime is supported by Toolkit

-   the SAM Application is built from the SAM Tempate
-   the resource's runtime is used to prepare for debugging
    -   Python: The lambda handler is wrapped by another method which starts the VS Code python debugger (ptvsd) and waits for a debugger to attach
    -   dotnetcore: the dotnetcore debugger is installed
-   the referenced SAM Application resource is invoked
-   the appropriate language debugger is connected to the running program

#### Debug Configurations that target a Lambda handler directly

This experience is suitable for prototyping some code before adding it into the SAM Template, or for working with code that does not belong to a SAM Application.

When a Debug Configuration targets a Lambda handler directly, it contains:

-   a path to the file containing the handler
-   a path representing the root of the application
-   a path to the manifest file (eg: `package.json` for Javascript)
-   the name of the handler
    -   JS/Python: this is the function name
    -   dotnetcore: this is the fully qualified assembly name
-   the runtime to use

The following takes place when this debug session is started:

-   the Debug Configuration is validated as follows. Failures prevent the debug session from proceeding:

    -   the code file exists
    -   the manifest file exists
    -   the Resource's Runtime is supported by Toolkit

-   a temporary SAM Application is created, containing a single resource populated by the configuration details
-   the SAM Application is handled in the same manner as above
-   the temporary SAM Application is then disposed

TODO : Unknown : Can we invoke local Run by other means?

### Sample

TODO : Sample SAM Template
TODO : Sample Debug Configuration

TBD Future Proposal Stage:

-   configuring overrides for the template/resource

### Defining Debug Configurations

Multiple options are available for users to create and define Debug Configurations.

#### Manual launch.json editing

Users open their `launch.json` file and add a Debug Configuration of type `AWS-SAM-Local`. Intellisense provides assistance around available fields, field descriptions, and missing field validation. User documentation is necessary, however the schema is simple.

Once entered, they can select their configuration from the Debug Panel dropdown, and initiate a debug session by pressing F5.

#### Template

VS Code provides "Add Configuration..." functionality, providing users with a list of Debug Configuration templates. An entry for "Local Serverless Application Debugging" produces a `AWS-SAM-Local` configuration with the minimum required fields for users to fill in. The templating system does not allow for further interactions with the user before producing a configuration, however the field descriptions and validation will assist users in filling in the configuration.

### Toolkit Command

A Toolkit Command provides a more interactive means of producing the configuration. Users are presented with a list of all SAM Templates detected in their Workspace. After selecting a template, users are shown a list of the template's resources that are lambda handlers. Debug Configuration entries will be auto-generated for selected resources, and written into `launch.json`.

This Command is accessed from the Command Palette. TODO : TBD : Are there any other menus it makes sense to add it to?

#### Automatic creation of Debug Configurations

VS Code provides extensions with the ability to automatically produce Debug Configuration entries. The automatically generated configuration entries are written to the workspace's `launch.json` file. TODO : Link API ProvideX call. The toolkit's Local Debug Configuration provider scans a workspace for all SAM Template files, and produces a Debug Configuration for every template resource that is a lambda handler.

VS Code only uses this functionality when a workspace does not contain a `launch.json` file. The other approaches to creating configurations help with this shortcoming by providing users with ways to create new Debug Configuration entries after their project (and workspace) have been initially set up. Additionally, users can delete their `launch.json` file, and have VS Code regenerate Debug Configuration entries into the file.

## Local SAM Templates View

A panel showing all of the SAM Templates that exist in a workspace provides a way of grouping all SAM related operations together.

The view is a tree where each root-level node represents a SAM Template file in the workspace. Each SAM Template node's children represent that template's resources that are lambda handlers. The Toolkit watches for SAM Template files in the workspace. As SAM Template files are found/created/deleted/modified, the View is updated to reflect the templates and resources available to work with.

This View resides next to the AWS Explorer View in the Side Bar for the AWS Panel. Users that aren't interested in SAM Template operatons can elect to hide the View. More information about managing Views in VS Code can be found [here](https://code.visualstudio.com/docs/getstarted/userinterface#_views).

### Template Node Operations

-   Jump to File (double click, context menu) - Opens the SAM Template file in VS Code
-   Generate Debug Configurations (context menu) - generates Debug Configuration entries for each of the child node resources
-   Deploy (context menu) - deploys SAM Application to AWS

### Template Resource Node Operations

-   Jump to Resource (context menu) - Opens the SAM Template and places the cursor at the corresponding resource
-   Jump to Code (double click, context menu) - Determines the function associated with the template resource and opens the file in VS Code (this might be tricky to do)
-   Generate Debug Configuration (context menu) - generates Debug Configuration entries for each of the child node resources and saves them in `launch.json`
-   Run Locally (context menu) - Invokes this template resource locally without attaching a debugger
    -   TODO : TBD : Run Local without Debug Config / Debugger?
-   Debug Locally (context menu) - Invokes this template resource locally and attaches a debugger
    -   this generates the same debug configuration as "Generate Debug Configuration", and tells VS Code to start it instead of saving it to the launch.json file
-   TODO : TBD : Configure?

### View Operations (not node specific)

-   Create New SAM App - Launches the workflow to create a new SAM Application

## CodeLenses on Lambda Handlers

A set of CodeLenses appear above function signatures that are recognized as Lambda handlers. The CodeLenses are detailed below. These CodeLenses allow users to locally invoke the current function code as a Lambda handler.

This debugging experience differs from the other ones in that the Lambda handler is run independently of any SAM Templates. All other debugging experiences invoke a resource that is already defined in a SAM Template. When these CodeLenses are used, a temporary SAM Template is produced that only contains the function of interest. When the debugging session is completed, the temporary SAM Template is deleted.

### Run Locally

The function associated with this CodeLens is placed into a new (temporary) SAM Application and then invoked without attaching a debugger.

### Debug Locally

The function associated with this CodeLens is placed into a new (temporary) SAM Application, and then invoked. A debugger is then attached to the invoked program.

### Configure

Allows users to configure the environment that the associated function can be locally run or debugged in.

Examples of what can be configured include:

-   Environment variables
-   Event payload (the object passed into the Lambda function when it is invoked)
-   Runtime
-   Root folder - This is the folder that is used as the root folder when the function is invoked. By default, it uses the folder containing the code file
-   Path to Manifest file relative to the root folder
    -   For a Javascript program, the `package.json` file
    -   For a Python program, the `requirements.txt` file

---

Additional Ideas

-   CodeLenses on SAM Templates - Template-level operations (create Debug Configurations for a Resource?)

TODO : Appendix: Section comparing proposal to existing feature
