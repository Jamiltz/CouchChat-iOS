# CouchChat demo for Couchbase Lite

CouchChat is a multi-user messaging app, essentially a primitive clone of the iOS Messages app. It illustrates how you can use Couchbase Lite in your mobile apps to share data across devices and among users. If you familiarize yourself with this code, you'll be ready to write your own multi-user interactive data driven applications on iOS.

There is a [tour of the data model](https://github.com/couchbaselabs/CouchChat-iOS/wiki/Chat-App-Data-Model) on the wiki.

## Architecture

There are three main components to the system:

* This app, which embeds [Couchbase Lite](https://github.com/couchbase/couchbase-lite-ios) for iOS.
* The Couchbase [Sync Gateway](https://github.com/couchbase/sync_gateway), which runs on a server and handles the synchronization connections from mobile devices.
* [Couchbase Server 2](http://www.couchbase.com/download) for data storage. (For development this is optional; you can instead use a very simple built-in data store called Walrus.)

Couchbase Server should be deployed behind your firewall (like databases normally are), and then the Sync Gateway should be deployed where it can be accessed by mobile devices from the public internet, and it can reach Couchbase Server. Mobile devices connect to the Sync Gateway, which enforces access control and update validation policies.

![Couchbase Mobile Architecture](http://jchris.ic.ht/files/slides/mobile-arch.png)

## Server setup

Before you can run the app, you'll need to install the Couchbase Sync Gateway and configure it for chat.

### Install the Sync Gateway

Installation and configuration of the Sync Gateway is beyond the scope of this README. The [Sync Gateway docs](https://github.com/couchbaselabs/sync_gateway) discuss installation. When you're done you'll be the proud owner of a `sync_gateway` command-line tool.

### Configure the gateway for chat

The CouchChat repo contains a configuration file for the gateway, `sync-gateway-config.json`. Put the path to the config file on the command line when you launch the gateway.

This config file does not contain the public (or LAN) address that your iOS app will contact. The Gateway needs to be aware of that address in order for Persona authentication to work properly. You'll provide that address as a command line option. It should consist only of the root URL of the server, including the port but without any path or trailing slash.

So your launch command will look something like:

    sync_gateway -personaOrigin="http://animal.local:4984" ~/code/CouchChat-iOS/sync-gateway-config.json

The example config file uses an in-memory Walrus bucket, so all your data will be lost when the Sync Gateway exits. To avoid this you can edit the `databases.sync_gateway.server` property of the config file to point to an existing directory where you'd like to store your data. In production you should replace the `walrus:` url with a URL to Couchbase Server's 8091 port.

## Running this app

### Install the submodules:

    git submodule init
    git submodule update

### Build/Install the Couchbase Lite framework

There are precompiled binary packages of Couchbase Lite, but they are currently (June 2013) not up to date, so the recommended way is to build the Couchbase Lite framework yourself, following [the directions in its wiki](https://github.com/couchbase/couchbase-lite-ios/wiki/Building-Couchbase-Lite#building-the-framework).

Once it is built, copy the output `CouchbaseLite.framework` into the the CouchChat repository's `Frameworks/` folder.

### Configure the server URL

Now edit the value of `kServerDBURLString` in `AppDelegate.m` to the public URL of your Sync Gateway's chat database. This should match the value of the `-personaOrigin` command-line flag given to the Sync Gateway, but with the database name (default is `chat`) appended. For example, you might change that line to:

    #define kServerDBURLString http://animal.local:4984/chat

### Build and run the app

Now you can build and run your app in the simulator or on a connected iOS device. 

After startup it will prompt you to login with Mozilla Persona. Once you are logged in, you can:

* Create a chat room
* Invite other users
* Send messages to users. 
* Attach pictures to a message (or take a picture if your device has a camera.) 

Any message in a chat room will show up on all devices that are subscribed to that room.

## Running the PhoneGap version

For extra credit you can try running the HTML5 version of CouchChat against the same Sync Gateway database. The HTML5 version of the app is under development in our [Couchbase Lite PhoneGap Kit](https://github.com/couchbaselabs/Couchbase-Lite-PhoneGap-Kit) repository.

## How the data flows

The JavaScript sync function in the server config file (`sync-gateway-config.json`) determines how data flows between mobile devices, or it can throw an error if a given update is not allowed to proceed. Read the Sync Gateway documentation for more details.

Read the [tour of the data model](https://github.com/couchbaselabs/CouchChat-iOS/wiki/Chat-App-Data-Model) to learn how the sync function for CouchChat works.


