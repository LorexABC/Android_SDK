# Android_SDK
 Android SDK
Usedesk Android SDK

Demo app
The "Example" folder of this repository contains a demo project that serves as an example of embedding the Usedesk chat SDK into a client application. You can use it to get acquainted with the basic functionality of the application and test the SDK.

Adding a library to the project
Minimal Android version
At the moment, the minimum version of the OS that is supported by the SDK — Android 5.0 (API 21)

SDK components
Chat SDK — chat library

Chat GUI — library for embedding ready-to-use GUI (includes Chat SDK)

KnowledgeBase SDK — library for working with the Knowledge Base

KnowledgeBase GUI — library for embedding ready-to-use GUI (includes KnowledgeBase SDK)

Steps to add the SDK to a project
Add a line to build.gradle of your project:

allprojects {
    repositories {
        ...
        maven { url "https://jitpack.io" }
    }
}
Add to the dependencies build.gradle of your module following lines:

//Chat SDK
implementation "com.github.usedesk.Android_SDK:chat-sdk:$usedeskSdkVersion"
//Chat GUI
implementation "com.github.usedesk.Android_SDK:chat-gui:$usedeskSdkVersion"
//Knowlage Base SDK
implementation "com.github.usedesk.Android_SDK:knowledgebase-sdk:$usedeskSdkVersion"
//Knowlage Base GUI
implementation "com.github.usedesk.Android_SDK:knowledgebase-gui:$usedeskSdkVersion"
Add to Manifest file:

<uses - permission android : name ="android.permission.INTERNET" / >

<!--To attach a photo from a camera to chat-- >
<uses - permission android : name ="android.permission.CAMERA" / >

<!--Only when using the foreground notification service -->
<uses - permission android : name ="android.permission.FOREGROUND_SERVICE" / >
Initializing a Chat
Parameters used in SDK configuration
Chat configuration is set in the file UsedeskChatConfiguration:

Where * — required parameter

Parameter	Type	Description
urlChat*	String	Server URL for SDK chats
By default: pubsubsec.usedesk.ru
If you use server version of Usedesk on your own server, value may be different for you. Check with support for valid URL — support@usedesk.com
urlChatApi*	String	URL to work with Usedesk API
By default: secure.usedesk.ru/uapi
If you use server version of Usedesk on your own server, value may be different for you. Check with support for valid URL — support@usedesk.com
companyId*	String	Company ID in Usedesk
How to find a company ID
channelId*	String	ID of the chat channel through which messages from the application will be placed at Usedesk
How to create and set up a channel
messagesPageSize	Int	Number of loaded messages when starting the chat
When client open a chat, a specified number of messages are loaded. As client scrolls chat, 20 more messages are loaded
clientToken	String?	A unique token that uniquely identifies the user and his conversation
The token is provided in the callback after the initialization of the chat and is linked to the mail-phone-user name.
To identify different users on the same device, you must store and pass the received token to the initialization method
By specifying null the library itself will use the saved token on the device used earlier with the same clientEmail, clientPhoneNumber, clientName fields in the configuration.
By specifying "" the saved token will not be used
clientEmail	String?	Client Email
clientName	String?	Client name
clientNote	String?	Text of note
clientPhoneNumber	Long?	Client Phone
clientAdditionalId	String?	Additional customer ID
clientInitMessage	String?	Automatic message
Sent immediately after initialization on behalf of the client
clientAvatar	String?	Customer avatar image
Path to the image file.
If set, then sdk will send the avatar at a time
additionalFields	Map<Long, String>	Collection of additional request fields
Where key is the field ID, value is the field value.
Field values depend on the type, for checkboxes - "true" / "false", for list boxes - text that exactly matches the text of the list value, for text - any text.
additionalNestedFields	List<Map<Long, String>>	List of collections of nested lists
Each list item is a collection of values of one nested list, where key is the field ID, value is the field value with text that exactly matches the text of the list value.
Local notification service configuration
The SDK can send notifications about new messages if the application is running and a connection to the chat server is established.

To enable the local notification service, you need to create 2 own classes:

Service inherited from UsedeskForegroundNotificationsService, where you can override the following methods:
Method	Returning type	Event description
getContentPendingIntent	PendingIntent?	Action on click on a notification
getDeletePendingIntent	PendingIntent?	Action on removing a notification
getClosePendingIntent	PendingIntent?	Action on closing a foreground notification
getChannelId	String	Notification channel number
getChannelTitle	String	Notification channel name
createNotification	Notification?	Creating a notification
Factory inherited from UsedeskNotificationsServiceFactory to override the method:
Method	Returning type	Event description
getServiceClass	Class <?>	Class of service
After creating classes, you can use the factory in the SDK:

UsedeskChatSdk.setNotificationsServiceFactory(CustomNotificationsServiceFactory())
Using with GUI
To run the SDK with a ready-made chat user interface (GUI), use UsedeskChatScreen. For example, using the newInstance method:

supportFragmentManager.beginTransaction()
    .replace(
        R.id.container,
        UsedeskChatScreen.newInstance(chatConfiguration)
    ).commit()
To use with Jetpack Navigation, you can use the createBundle method, for example:

navController.navigate(
    R.id.action_configurationScreen_to_usedeskChatScreen,
    UsedeskChatScreen.createBundle(chatConfiguration)
)
The newInstance and createBundle methods take the following arguments:

Argument	Type	Description
chatConfiguration	UsedeskChatConfiguration	UsedeskChatScreen assumes the responsibility of calling the UsedeskChatSdk.setConfiguration method
customAgentName	String?	If set, all agent names in the chat will be replaced by the value of the parameter
rejectedFileExtensions	Collection<String>?	List of file extensions marked as dangerous (the onFileClick method of the parent is called anyway)
messagesDateFormat	String?	If set, changes the format of the message group date display
messageTimeFormat	String?	If set, changes the format of the message time display
adaptiveTextMessageTimePadding	Boolean	If true is set, shifts the text of messages relative to the time
groupAgentMessages	Boolean	If true is set, groups messages from the same agent
For the fragment to work fully it is necessary to:

Pass the onBackPressed event by calling the same method on the fragment, which will return true if the event was handled, or false if not
Example:

override fun onBackPressed() {
    val fragment = getCurrentFragment()
    if (fragment is UsedeskFragment && fragment.onBackPressed()) {
        return
    }
}
Implement the IUsedeskOnFileClickListener interface by overriding the onFileClick method
Example:

override fun onFileClick(usedeskFile: UsedeskFile) {
    supportFragmentManager.beginTransaction()
        .replace(R.id.container, UsedeskShowFileScreen.newInstance(usedeskFile))
        .commit()
    //or
    navController.navigate(
        R.id.action_usedeskChatScreen_to_usedeskShowFileScreen,
        UsedeskShowFileScreen.createBundle(usedeskFile)
    )
}
Implement the IUsedeskOnDownloadListener interface as a parent by overriding the onDownload method.

To bind a ViewModel lifecycle to a parent, you need to implement the IUsedeskChatViewModelStoreOwner interface

To receive a client token, implement the IUsedeskOnClientTokenListener interface as a parent

Implement the IUsedeskOnChatInitedListener interface as a parent to track when the chat is initialized

To correctly work with a camera photo attachment, you need to add the following lines to the AndroidManifest.xml file:

<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.provider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/usedesk_provider_paths" />
</provider>
To be able to display video in full screen, the IUsedeskOnFullscreenListener interface must be implemented
Using without a GUI
To work with chat without GUI you need to perform the following steps:

Set the configuration and initialize chat:
UsedeskChatSdk.setConfiguration(UsedeskChatConfiguration())
val usedeskChat = UsedeskChatSdk.init(requireContext())
//or
val usedeskChat = UsedeskChatSdk.init(requireContext(), UsedeskChatConfiguration())
Get an instance of IUsedeskChat after initialization. To do this, call:
val usedeskChat = UsedeskChatSdk.requireInstance()
Add an event listener:
val listener = object : IUsedeskActionListener {}
usedeskChat.addListener(listener)
In order to remove the listener, you need to call the corresponding method:

usedeskChat.removeListener(listener)
When you have finished working with the chat, to free the resources you need to call following method:
UsedeskChatSdk.release(false)
If you pass false to the method, the resources will only be released if all listeners have been deleted. If you pass the value true, the resources will be released immediately.

Attempting to retrieve an instance without initializing or after release will raise an exception.

Use interface to listen to chat events IUsedeskActionListener:
Method	Event description
onModel	Chat model, new, updated and deleted messages
onException	The exception that has risen
Starting and stopping the local notification service
Starting the notification service:

UsedeskChatSdk.startService(context)
Stopping the notification service:

UsedeskChatSdk.stopService(context)
Error Logging
To log server response processing errors, you can use the class UsedeskLog:

enable() — enabling logging.
disable() — disabling logging.
addLogListener(logListener: (String) -> Unit) — adding the log listener.
removeLogListener(logListener: (String) -> Unit) — removing the log listener.
Initializing the Knowledge Base
Configuration
The configuration of the Knowledge Base is set in the file UsedeskKnowledgeBaseConfiguration:

Where * — required parameter

Parameter	Type	Description
accountId*	String	Knowledge Base ID
How to create a Knowledge Base
token*	String	Usedesk API Token
How to get API Token
clientEmail	String?	Client email
clientName	String?	Client name
Using with the GUI
Use UsedeskKnowledgeBaseScreen to use a ready-made user interface, for example using the newInstance method:

supportFragmentManager().beginTransaction()
    .replace(
        R.id.container,
        UsedeskKnowledgeBaseScreen.newInstance(
            configuration = UsedeskKnowledgeBaseConfiguration(),
            withSupportButton = true,
            deepLink = DeepLink.Article(articleId = 123L, noBackStack = true)
        )
    ).commit()
You can use the createBundle method for use with Jetpack Navigation.

Example:

navController.navigate(
    R.id.action_configurationScreen_to_usedeskKnowledgeBaseScreen,
    UsedeskKnowledgeBaseScreen.createBundle(
        configuration = UsedeskKnowledgeBaseConfiguration(),
        withSupportButton = true,
        deepLink = DeepLink.Article(articleId = 123L, noBackStack = true)
    )
)
For the fragment to work properly it is necessary to:

Pass onBackPressed events by calling a similar method on the fragment, which will return true if the event was handled, or false if not.
Example:

override fun onBackPressed() {
    val fragment = getCurrentFragment()
    if (fragment is UsedeskFragment && fragment.onBackPressed()) {
        return
    }
}
Implement the IUsedeskOnSupportClickListener interface as a parent, overriding the onSupportClick() method,
Example:

override fun onSupportClick() {
    supportFragmentManager().beginTransaction()
        .replace(R.id.container, UsedeskChatScreen().newInstance())
        .commit()
}
To process clicks on links in articles, you need to implement IUsedeskOnWebUrlListener interface as a parent.
Using without a GUI
To work with the Knowledge Base without GUI you need to perform the following steps:

Initialize the Knowledge Base:
val usedeskKnowledgeBase =
    UsedeskSdk.initKnowledgeBase(context, UsedeskKnowledgeBaseConfiguration())
//or
UsedeskKnowledgeBaseSdk.setConfiguration(UsedeskKnowledgeBaseConfiguration())
val usedeskKnowledgeBase = UsedeskSdk.initKnowledgeBase(context)
Get a class object at any place:
val usedeskKnowledgeBase = UsedeskKnowledgeBase.requireInstance()
Release the object:
UsedeskKnowledgeBaseSdk.release()
If you try to retrieve an instance without initializing the Knowledge Base, or after the object has been released, an exception will be raised.

In addition, you can change an existing language or add a new one. To do this, you need to copy the resources from the files that refer to @string/usedesk_string and add them to the strings.xml of your project, substituting the necessary values:
common-gui strings.xml
chat-gui strings.xml
knowledgebase-gui strings.xml
In the case of changing string resource references when customizing an application, changing string resources in this way may not have the desired result.