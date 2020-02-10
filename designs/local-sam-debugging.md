# Local Debugging Experience for SAM Applications

Current Proposal Stage: General Experience

Future Proposal Stages (as future PRs):

-   General Experience In-depth
-   Architecture

Previous Proposal Stages:

-   None

## Introduction

TODO : Intro

While this document's main focus is on debugging capabilities in the toolkit, there are places where the experience around invoking without the debugger (aka "running") is also discussed.

TODO : A limited selection of programming languages are supported in the Toolkit.

### Terminology

TODO : Fill this section

#### CodeLens

CodeLenses are visual decorators anchored to a document location. They are used to convey information and/or provide links that trigger an action. They are a presentation-only mechanic and do not reside within a file. Additional information and examples about CodeLenses can be found [on the VS Code blog](https://code.visualstudio.com/blogs/2017/02/12/code-lens-roundup).

#### Debug Configuration

Debug Configurations are JSON entries within the `.vscode/launch.json` file optionally located in each VS Code workspace. These are user managed, defining what programs can be debugged. Presing F5 (or the Debug button) starts a Debugging session for the currently selected Debug Configuration. VS Code extensions can provide and implement Debug Configuration types in addition to those available in VS Code.

More information about VS Code Debugging can be found [in the VS Code Documentation](https://code.visualstudio.com/docs/editor/debugging).

#### SAM Template

A SAM Template defines a Serverless Application's resources, and supporting code. This is used by the SAM CLI to build, run, package, and deploy the Application.

Additional information about SAM can be found at:

-   [SAM Homepage](https://aws.amazon.com/serverless/sam/)
-   [What Is the AWS Serverless Application Model (AWS SAM)?](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
-   [SAM CLI GitHub Repo](https://github.com/awslabs/aws-sam-cli)

## Overview

The following scenarios are supported for Locally Running and Debugging with the Serverless Application Model:

-   Invoking SAM Template Lambda Function Resources
-   API Gateway requests against SAM Template Lambda Function Resources
-   Invoking standalone Lambda Function Handlers
-   API Gateway requests against standalone Lambda Function Handlers

---

Users can Locally Debug SAM Applications in the following ways:

-   Debug Configurations - Launch a Debugging session using the Debug Panel in VS Code and pressing F5.
-   Local SAM Templates View - One UI Location to see and act on all SAM Applications / Functions
-   CodeLenses on Lambda Handlers - Locally run and debug a Lambda handler function without any SAM Template associations

## What can be Debugged Locally

### SAM Template Resources

SAM Template Resources of type `AWS::Serverless::Function` represent Lambda functions. The corresponding Lambda function code (if present) can be locally Run or Debugged. The SAM CLI is used to invoke the Lambda function similar to how it is run in the cloud, and a debugger can be attached to the Lambda function code.

If the SAM Template Resource contains an event of type [Api](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-property-function-api.html), the SAM CLI can also be used to invoke the Lambda function in a manner similar to how they are invoked through API Gateway.

### Standalone Lambda Function Handlers

Lambda Function Handler code can be locally Run or Debugged, even if it does not belong to a SAM Application. A temporary SAM Application is produced behind the scenes to contain the handler of interest, and the SAM CLI is used to invoke the Lambda function, as outlined in SAM Template Resources. Afterwards, the temporary SAM Application is removed.

In this mode, any SAM Templates that a Handler is associated with are ignored. This prevents confusion/errors introduced when trying to resolve between SAM Template Resource handlers with code (examples include incorrectly determining a function's lambda handler string, or situations where more than one resource references the same function).

The Toolkit does not provide support for locally running or debugging standalone Lambda function handlers in a manner emulating API Gateway. The code should be referenced from a SAM Template to use the API Gateway style debugging mentioned in the earlier section.

## What can be configured for a Debug session?

The following parameters influence a debug seession.

| Property                | Description                                          | Used by Standalone Lambda Handler | Used by SAM Template Resources |
| ----------------------- | ---------------------------------------------------- | --------------------------------- | ------------------------------ |
| SAM Template            | Path to SAM Template file                            |                                   | x                              |
| SAM Template Resource   | Name of resource within SAM Template                 |                                   | x                              |
| SAM Template Parameters | Values to use for SAM Template Parameters            |                                   | x                              |
| Environment Variables   | Environment Variables exposed to the Lambda Function | x                                 | x                              |
| Input Event             | Payload passed to the invoked Lambda Function        | x                                 | x                              |
| Runtime                 | Runtime of Lambda Function to invoke                 | x                                 |                                |
| Handler                 | Lambda Function Handler to invoke                    | x                                 |                                |
| Timeout                 | Timeout threshold for Lambda function                | x                                 |                                |
| Memory                  | Memory provided to Lambda function                   | x                                 |                                |

The following SAM CLI related arguments are relevant to debugging both standalone lambda function handlers and sam template resources. For reference see the [sam build](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html) command.

| Property                                                         | Default Value                    |
| ---------------------------------------------------------------- | -------------------------------- |
| Build SAM App in container                                       | false                            |
| Skip new image check                                             | false                            |
| use a docker network                                             | empty string (no docker network) |
| additional build args (passed along to `sam build` calls)        | empty string                     |
| additional local invoke args (passed along to `sam local` calls) | empty string                     |

The following AWS related arguments are relevant to debugging both standalone lambda function handlers and sam template resources:

-   Credentials
-   Region

## Local Debugging Experiences

CC: what is/isn't supported, and what it looks like for each experience

### Debug Configurations

The Toolkit implements a Debug Configuration type `aws-sam`. When run, this configuration type:

-   validates debug configuration inputs
-   uses SAM CLI to build a SAM Application
-   uses SAM CLI to invoke a SAM Template Resource
-   attaches a debugger to the SAM invocation
-   if the debug configuration is for a local api gateway invoke, the debugger is detached after the http request is made, but SAM CLI remains active. The debug configuration implementation terminates the SAM CLI session to prevent a proliferation of CLI processes.

In the most basic form, the debug configuration references a SAM Template file location, and a Resource within that file. Other execution parameters can be configured, but are optional.

Debugging local lambda invokes and local api gateway invokes each require slightly different inputs. The `aws-sam` Debug Configuration uses different request types to accommodate these variations.

These debug configurations are authored in a json file. The toolkit assists with this as follows:

-   autocompletion with descriptions is provided for `aws-sam` related fields
    -   There is no autocompletion available for specific values in a configuration. For example, if a user types in the location of a SAM Template file, there is no filesystem-based autocompletion. The Debug Configuration validates the configuration and notifies of errant values when it is run.
-   snippets to produce typical (or starter) `aws-sam` debug configurations
-   when no launch.json file is present in a workspace, VS Code exposes functionality that allows users to request auto-generated Debug Configurations. In this situation, the toolkit generates an `aws-sam` Debug Configuration for all `AWS::Serverless::Function` Resources detected within all SAM Templates located in the workspace.

Example Debug Configuration entries can be found in the Appendix - Sample Debug Configurations

Debug Configurations are the idiomatic approach to running and debugging software in VS Code. This is the main experience for debugging SAM Template resources in the toolkit.
Standalone Lambda function handlers are not supported through Debug Configurations.

### CodeLenses

Toolkit settings can be used to enable and disable CodeLenses.
CodeLenses only appear for languages/runtimes that the Toolkit has implemented Debug support for.

#### CodeLenses in SAM Template files

The following CodeLenses appear above every template resource of type `AWS::Serverless::Function`:

-   Run Locally - See below for details
-   Debug Locally - See below for details
-   Configure - allows the user to configure a limited set of arguments that are used with the Run and Debug CodeLenses
    -   Anything that can be defined by the SAM Template would not be configurable in here
    -   This covers aspects like input event, and SAM CLI related arguments
-   Add Debug Configuration - Utility feature to produce a skeleton Debug Configuration in `launch.json` for users

When clicked, the Run and Debug CodeLenses locally invoke their associated Template Resource. The following takes place:

-   The SAM Application is built
-   The associated SAM Template resource is invoked, using configurations set with the Configure CodeLens
-   (If the Debug CodeLens was clicked) The VS Code debugger is attached to the invoked resource

When clicked, the Configure CodeLens opens a (JSON) configuration file that resides in the workspace and is managed by the Toolkit. The configuration file is used for each SAM Template Resource within the workspace. Users have autocompletion support with this file. A rich UI is not considered at this time, but the door remains open to adding a visual editor in the future based on user feedback.

The Run and Debug CodeLenses perform a regular local invoke on a resource. These CodeLenses do not perform API Gateway style invokes.

#### CodeLenses in Code files

CodeLenses in code files provides support for debugging Standalone Lambda function handlers.

The following CodeLenses appear over any function that appears to be an eligible Lambda Handler:

-   Run Locally - See below for details
-   Debug Locally - See below for details
-   Configure - allows the user to configure arguments that are used with the Run and Debug CodeLenses (see What can be configured for a Debug session?)

When clicked, the Run and Debug CodeLenses locally invoke the Lambda handler function they represent. These Lambda handlers are invoked independent of SAM Templates that exist in the users workspace. The following takes place:

-   A temporary SAM Template is produced, which contains one resource that references the Lambda handler
-   The temporary SAM Application is built
-   The resource in the temporary SAM Template is invoked, using configurations set with the Configure CodeLens
-   (If the Debug CodeLens was clicked) The VS Code debugger is attached to the invoked resource

When clicked, the Configure CodeLens opens a (JSON) configuration file that resides in the workspace and is managed by the Toolkit. All standalone handlers within a workspace will have their configurations stored in this file. Users have autocompletion support with this file. A rich UI is not considered at this time, but the door remains open to adding a visual editor in the future based on user feedback.

These CodeLenses do not perform API Gateway style invokes.

Some users may find CodeLenses within code files distracting, particularly if they are using the Toolkit for features not related to local debugging. Toolkit settings can be used to enable and disable CodeLenses.

### User Interface

A UI is provided to support API Gateway based local debugging of SAM Template Resources. The view resembles a simple REST request workbench. After selecting a SAM Template, and a resource from that template, users craft a REST request (GET, POST, ect, as well as query string and body). Submitting the request (through a Run or Debug button) performs the following:

-   build the SAM Application
-   invoke the SAM Template Resource in api gateway mode
-   send the REST request to the invoked SAM application
-   (if debugging)
    -   attach a debugger to the invoked SAM application
    -   once the lambda handler exits, the debug session ends. The Toolkit terminates the invoked SAM application
-   (if running)
    -   once a response is received, the toolkit terminates the invoked SAM application
-   The request, response, response code, and sam cli output are output to the toolkit's OutputChannel

Users have the option to customize the SAM invocation in the same way as CodeLenses in SAM Template files.

---

## Local Debug Arguments

### Concept

```json
{
    "configurations": [
        {
            "name": "a",
            "type": "aws-sam",
            "request": "template-invoke",
            "samTemplate": {
                "path": "some path",
                "resource": "HelloWorldResource",
                "parameters": {
                    "param1": "somevalue"
                }
            },
            "environmentVariables": {
                "envvar1": "somevalue",
                "envvar2": "..."
            },
            "event": {
                "path": "somepath",
                "json": {
                    // some json
                    // path or json, not both
                }
            },
            "sam": {
                "containerBuild": false,
                "skipNewImageCheck": false,
                "dockerNetwork": "aaaaa",
                "buildArguments": "--foo",
                "localArguments": "--foo"
            },
            "aws": {
                "credentials": "profile:default",
                "region": "us-west-2"
            }
        },
        {
            "name": "a2",
            "type": "aws-sam",
            "request": "template-api",
            "samTemplate": {
                "path": "some path",
                "resource": "HelloWorldResource",
                "parameters": {
                    "param1": "somevalue"
                }
            },
            "environmentVariables": {
                "envvar1": "somevalue",
                "envvar2": "..."
            },
            // If event is missing, don't terminate
            "event": {
                "api": {
                    "path": "/bee",
                    "method": "get",
                    "query": "aaa=1&bbb=2",
                    "body": "text - can we do this?"
                }
            },
            "sam": {
                "containerBuild": false,
                "skipNewImageCheck": false,
                "dockerNetwork": "aaaaa",
                "buildArguments": "--foo",
                "localArguments": "--foo"
            },
            "aws": {
                "credentials": "profile:default",
                "region": "us-west-2"
            }
        }
        // lambda invoke -- programmatically generate the template equivalents
        // {
        //     "name": "a3",
        //     "type": "aws-sam",
        //     "request": "lambda-invoke"
        // },
        // {
        //     "name": "a4",
        //     "type": "aws-sam",
        //     "request": "lambda-api"
        // }
    ]
}
```

---

### template invoke

#### from UI

? No UI ?

-   pick template + resource
-   other configuration
-   select event
-   Buttons: Run, Debug, Save to Debug Config

### template start-api

-   sam build
-   sam local start-api
-   make http request
-   surface results (statusCode, sam output, response)
-   terminate process

*   What about keeping it running?

#### from UI

-   pick template + resource
-   ? No other configuration ?
-   set path
-   set method
-   set query, body
-   Buttons: Start, Request, End, Debug Toggle

### Defining Debug Configurations

Multiple options are available for users to create and define Debug Configurations.

TODO : SEE : https://code.visualstudio.com/docs/editor/debugging#_add-a-new-configuration

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

## Appendix

### Differences between this doc and v1.0.0 of Toolkit

-   Lambda Handlers no longer associated with SAM Templates

### Sample Debug Configurations

---

Additional Ideas

-   CodeLenses on SAM Templates - Template-level operations (create Debug Configurations for a Resource?)

TODO : Appendix: Section comparing proposal to existing feature

# SCRAP

## Old Overview

The Local Debugging features released in version 1.0 are limited, and have some design limitations. TODO Reference Issue. This proposal improves the user experience with additional ways to locally debug SAM Applications, and disambiguates some of the unspecified behaviors.
