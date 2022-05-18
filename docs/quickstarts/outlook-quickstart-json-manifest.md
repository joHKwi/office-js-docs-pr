---
title: Build an Outlook add-in with a Teams manifest (preview)
description: Learn how to build a simple Outlook task pane add-in with a JSON manifest.
ms.date: 05/24/2022
ms.prod: outlook
ms.localizationpriority: high
---

# Build an Outlook add-in with a Teams manifest (preview)

In this article, you'll walk through the process of building an Outlook task pane add-in that displays a property of a selected message, triggers a notification on the reading pane, and inserts text into a message on the compose pane. This add-in will use a preview version of the JSON-formatted manifest that Teams extensions, like custom tabs and messaging extensions, use. For more information about this manifest, see [Teams manifest for Office Add-ins (preview)](../develop/json-manifest-overview.md).

The new manifest is available for preview and we encourage experienced add-in developers to experiment with it. It should not be used in production add-ins. The preview is only supported on subscription Office on Windows. 

> [!TIP]
> If you want to build an Outlook add-in using the XML manifest, see [Build your first Outlook add-in](outlook-quickstart.md).

## Create the add-in

You can create an Office Add-in with a JSON manifest by using the [Yeoman generator for Office Add-ins](../develop/yeoman-generator-overview.md). The Yeoman generator creates a Node.js project that can be managed with Visual Studio Code or any other editor.

### Prerequisites

[!include[Set up requirements](../includes/set-up-dev-environment-beforehand.md)]

- [.NET runtime](https://dotnet.microsoft.com/download/dotnet/6.0/runtime) for Windows. One of the tools used in the preview runs on .NET.

[!INCLUDE [Yeoman generator prerequisites](../includes/quickstart-yo-prerequisites.md)]

- [Visual Studio Code (VS Code)](https://code.visualstudio.com/) or your preferred code editor

- Outlook on Windows (connected to a Microsoft 365 account)

### Create the add-in project

1. [!include[Yeoman generator create project guidance](../includes/yo-office-command-guidance.md)]

    - **Choose a project type** - `Outlook Add-in with Teams Manifest (Developer preview)`

    - **What do you want to name your add-in?** - `Outlook Add-in with Teams Manifest`

     ![Screenshot showing the prompts and answers for the Yeoman generator in a command line interface with JSON manifest option chosen.](../images/yo-office-outlook-json-manifest.png)

    After you complete the wizard, the generator will create the project and install supporting Node components.

    [!include[Yeoman generator next steps](../includes/yo-office-next-steps.md)]

1. Navigate to the root folder of the web application project.

    ```command&nbsp;line
    cd "Outlook Add-in with Teams Manifest"
    ```

### Explore the project

The add-in project that you've created with the Yeoman generator contains sample code for a very basic task pane add-in.

- The **./manifest.json** file in the root directory of the project defines the settings and capabilities of the add-in.
- The **./src/taskpane/taskpane.html** file contains the HTML markup for the task pane.
- The **./src/taskpane/taskpane.css** file contains the CSS that's applied to content in the task pane.
- The **./src/taskpane/taskpane.ts** file contains code that calls the Office JavaScript library to facilitate interaction between the task pane and Outlook.
- The **./src/command/command.html** file will be edited by WebPack at build time to insert an HTML `<script>` tag that loads the JavaScript file that is transpiled from the command.ts file.
- The **./src/command/command.ts** file has little code in it at first. Later in this article, you'll add code to it that calls the Office JavaScript library and that executes when a custom ribbon button is selected.

### Update the code

1. Open your project in VS Code or your preferred code editor.
   [!INCLUDE [Instructions for opening add-in project in VS Code via command line](../includes/vs-code-open-project-via-command-line.md)]

1. Open the file **./src/taskpane/taskpane.html** and replace the entire **\<main\>** element (within the **\<body\>** element) with the following markup. This new markup adds a label where the script in **./src/taskpane/taskpane.ts** will write data.

    ```html
    <main id="app-body" class="ms-welcome__main" style="display: none;">
        <h2 class="ms-font-xl"> Discover what Office Add-ins can do for you today! </h2>
        <p><label id="item-subject"></label></p>
        <div role="button" id="run" class="ms-welcome__action ms-Button ms-Button--hero ms-font-xl">
            <span class="ms-Button-label">Run</span>
        </div>
    </main>
    ```

1. In your code editor, open the file **./src/taskpane/taskpane.js** and add the following code within the **run** function. This code uses the Office JavaScript API to get a reference to the current message and write its **subject** property value to the task pane.

    ```js
    // Get a reference to the current message
    let item = Office.context.mailbox.item;

    // Write message property value to the task pane
    document.getElementById("item-subject").innerHTML = "<b>Subject:</b> <br/>" + item.subject;
    ```

### Try it out

[!INCLUDE [alert use https](../includes/alert-use-https.md)]

1. Run the following command in the root directory of your project. When you run this command, the local web server starts and your add-in will be [sideloaded](../outlook/sideload-outlook-add-ins-for-testing.md). 

    ```command&nbsp;line
    npm start
    ```

1. In Outlook, be sure you are using the Classic Ribbon. The remainder of these instructions assume this.  

1. View a message in the [Reading Pane](https://support.microsoft.com/office/2fd687ed-7fc4-4ae3-8eab-9f9b8c6d53f0), or open the message in its own window. A new control group named **Contoso Add-in** appears on the Outlook **Home** tab (or the **Message** tab if you opened the message in a new window). The group has a button named **Show Taskpane** and one named **Perform an action**.

1. Select the **Perform an action** button. It [executes a command](../develop/create-addin-commands?branch=outlook-json-manifest#step-5-add-the-functionfile-element), specifically, a small informational notification appears at the bottom of the message header, just below the message body.

1. Choose the **Show Taskpane** button in the ribbon to open the add-in task pane.

    > [!NOTE]
    > If you receive the error "We can't open this add-in from localhost" in the task pane, follow the steps outlined in the [troubleshooting article](/office/troubleshoot/office-suite-issues/cannot-open-add-in-from-localhost).

1. When prompted with the **WebView Stop On Load** dialog box, select **OK**.

    [!INCLUDE [Cancelling the WebView Stop On Load dialog box](../includes/webview-stop-on-load-cancel-dialog.md)]

1. Scroll to the bottom of the task pane and choose the **Run** link to copy the message's subject to the task pane.

1. End the debugging session with the following command:

    ```command&nbsp;line
    npm stop
    ```

    > [!IMPORTANT]
    > Closing the web server window does not reliably shut down the web server. If it is not properly shut down, you'll encounter problems as you change and rerun the project.

1. Close all instances of Outlook.

## Add a custom button to the ribbon

Let's add a custom button to the ribbon that inserts text into a message body.

1. Open your project in VS Code or your preferred code editor.
   [!INCLUDE [Instructions for opening add-in project in VS Code via command line](../includes/vs-code-open-project-via-command-line.md)]

1. In your code editor, open the file **./src/command/command.ts** and add the following code to the end of the file. This function will insert "Hello World" at the cursor point in message body.

    ```js
    function insertHelloWorld(event: Office.AddinCommands.Event) {
        Office.context.mailbox.item.body.setSelectedDataAsync("Hello World", {coercionType: Office.CoercionType.Text});

        // Be sure to indicate when the add-in command function is complete
        event.completed();
    }

    // Put the function on the global namespace
    g.insertHelloWorld = insertHelloWorld;
    ```

1. Open the file **./manifest/manifest.json**.

    > [!NOTE]
    > When referring to nested JSON properties, this article uses dot notation. When an item in an array is referenced, the bracketed zero-based number of the item is used. 

1. To write to a message, the add-in's permissions need to be raised. Scroll to the property `authorization.permissions.resourceSpecific[0].name` and change the value to "MailboxItem.ReadWrite.User".

1. When an add-in command executes code instead of opening a task pane, it must run the code in a JavaScript runtime that is separate from the embedded webview in which task pane code runs. So the manifest must specify an additional runtime. Scroll to the property `extension.runtimes` and add the following object to the `runtimes` array. Be sure to put a comma after the object that is already in the array. Note the following about this markup:

    - The value of the `actions.items[1].id` property is "Contoso.insertHelloWorld". In a later step, you will refer to the item by this ID.
    - The value of the `actions.items[1].name` property must be exactly the same as the name of the function that you added to the **commands.ts** file, in this case "insertHelloWorld".

    ```json
    {
        "id": "CustomButtonRuntime",
        "type": "general",
        "code": {
            "page": "https://localhost:3000/commands.html",
            "script": "https://localhost:3000/commands.js"
        },
        "lifetime": "short",
        "actions": {
            "items": [
                {
                    "id": "Contoso.insertHelloWorld",
                    "type": "execution",
                    "name": "insertHelloWorld"
                }
            ]
        }
    }
    ```

1. The **Show Taskpane** button appears on the ribbon when the user is reading an email, but the button for adding text should only appear on the ribbon when the user is composing a new email (or replying to one). So the manifest must specify a new ribbon object. Scroll to the property `extension.ribbons` and add the following object to the `ribbons` array. Be sure to put a comma after the object that is already in the array. Note the following about this markup:

    - The only value in the `contexts` array is "composeMail", so the button will appear when in a compose (or reply) window but not in a message read window where the **Show Taskpane** and **Perform an action** buttons appear. Compare this value with the `contexts` array in the existing ribbon object, whose value is `["readMail"]`.
    - The value of the `tabs[0].groups[0].controls[0].actionId` must be exactly the same as the value of `actions.items[1].id` property in the runtime object you created in an earlier step.

    ```json
    {
        "contexts": ["composeMail"],
        "tabs": [
            {
            "id": "TabDefault",
            "groups": [
                    {
                    "id": "msgWriteGroup",
                    "label": "Contoso Add-in",
                    "icons": [
                        { "size": 16, "file": "https://localhost:3000/assets/icon-16.png" },
                        { "size": 32, "file": "https://localhost:3000/assets/icon-32.png" },
                        { "size": 80, "file": "https://localhost:3000/assets/icon-80.png" }
                    ],
                    "controls": [
                            {
                                "id": "HelloWorldButton",
                                "type": "button",
                                "label": "Insert text",
                                "icons": [
                                    { "size": 16, "file": "https://localhost:3000/assets/icon-16.png" },
                                    { "size": 32, "file": "https://localhost:3000/assets/icon-32.png" },
                                    { "size": 80, "file": "https://localhost:3000/assets/icon-80.png" }
                                ],
                                "supertip": {
                                    "title": "Insert text",
                                    "description": "Inserts some text."
                                },
                                "actionId": "Contoso.insertHelloWorld"
                            }                  
                        ]
                    }
                ]
            }
        ]
    }
    ```

### Try out the updated add-in

1. Run the following command in the root directory of your project.

    ```command&nbsp;line
    npm start
    ```

1. In Outlook, open a new message window (or reply to an existing message). A new control group named **Contoso Add-in** will appear on the Outlook **Home** tab. The group has a button named **Insert text**.

1. Put the cursor anywhere in the message body and choose the **Insert text** button.

    > [!NOTE]
    > If you receive the error "We can't open this add-in from localhost" in the task pane, follow the steps outlined in the [troubleshooting article](/office/troubleshoot/office-suite-issues/cannot-open-add-in-from-localhost).

1. When prompted with the **WebView Stop On Load** dialog box, select **OK**.

    [!INCLUDE [Cancelling the WebView Stop On Load dialog box](../includes/webview-stop-on-load-cancel-dialog.md)]

    The phrase "Hello World" will be inserted at the cursor.

1. End the debugging session with the following command:

    ```command&nbsp;line
    npm stop
    ```

## See also

- [Teams manifest for Office Add-ins (preview)](../develop/json-manifest-overview.md)