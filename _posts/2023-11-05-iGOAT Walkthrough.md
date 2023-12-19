---
title: iGOAT Walkthrough
categories: Mobile iOS
mermaid: true
---

In this entry I will solve the vulnerable iOS app based on the OWASP Mobile Application Security framework vulnerabilties.

# What is iGOAT?

[OWASP iGoat (Swift)](https://github.com/OWASP/iGoat-Swift) is a Swift-based version of the original iGoat project, which was initially implemented in Objective-C. It serves as a deliberately vulnerable iOS application designed for educational purposes. With iGoat (Swift), you can gain hands-on experience in both exploiting and defending against security vulnerabilities commonly found in iOS Swift applications.

Documentation: https://docs.igoatapp.com/

Covered topics:

- Reverse Engineering
- Runtime Analysis
- Data Protection (Rest)
- Data Protection (Transit)
- Key Management
- Tampering
- Injection Flaws
- Broken Cryptography
- Memory Management
- URL Scheme Attack
- Social Engineering
- SSL Pinning
- Authentication
- Jailbreak Detection
- Side Channel Data Leaks
- Cloud Misconfiguration
- Crypto Challenges

## Data Protecion 

### 1\. Core data Storage

In this exercise, the app stores data in Core Data Format. Your task is to locate the CoreData file and find the sensitive data that it contains.

Core Data is a framework provided by Apple for data storage and management in iOS applications. It allows developers to work with structured data, typically using a database-like approach, to store, retrieve, and manage data efficiently. It's commonly used for persisting application data. The location of Core Data storage, if using SQLite, can be found in the Library/Application Support or Library/Documents. To easliy find database files do the following search: `find ./ -name "*.sqlite" -or -name "*.db"`

The first step to locate the CoreData file and the sensitive data is finding the &lt;AppID&gt; for iGoat application. For that run `frida-ps -Uai`. This command lists running processes on a USB-connected iOS device.

![68135546f491856f8ae630f7e8e60094.png](/assets/img/screenshots/iGoat/68135546f491856f8ae630f7e8e60094.png)

Then, move to /var/mobile/Containers/data/Application/ and do grep with &lt;AppID&gt;: `find * | grep -i igoat`

![bfaf025e7397cf763d2f590ac235e633.png](/assets/img/screenshots/iGoat//bfaf025e7397cf763d2f590ac235e633.png)

Now move to /610A25B0-5BF6-4BD0-A227-F51AD187523F/Library/Application Support and try to access the Core Data Application files using Sqlite3: `sqlite3 CoreData.sqlite`

![c80948bc756798648a25901462263ca8.png](/assets/img/screenshots/iGoat/c80948bc756798648a25901462263ca8.png)

### 2\. Plist Storage

In this exercise the app stores data using a Plist file in the application sandbox.Your task is to locate the Plist file and find the sensitive data that it contains.

The directory /private/var/mobile/Containers/Data/Application/ in an iOS device's file system typically contains application-specific documents and data that are meant to be accessible to the user and may be created, modified, or managed by the app.

Move to /private/var/mobile/Containers/Data/Application/&lt;AppUUID&gt;/Documents/

There Credentials.plist is found, to be able to read it convert it using plist: `plutil -convert xml1 OWASP.iGoat-Swift.plist`

![04ca5f1c8a19102dbd989c342115d47e.png](/assets/img/screenshots/iGoat/04ca5f1c8a19102dbd989c342115d47e.png)

### 3\. NSUserDefaults Storage

In this exercise the app stores data using a NSUserDefaults file in the application sandbox.Your task is to locate the NSUserDefaults file and find the sensitive data that it contains.
In iOS, NSUserDefaults is a way to store small amounts of data persistently. The data is stored as a property list file (.plist) in the app's sandboxed file system. The location of the NSUserDefaults file is typically in Library/Preferences folder.

Move to /var/mobile/Containers/data/Application/ and do grep with &lt;AppID&gt;:

Once in the folder, as the .plist file is not readable, convert the file in xml format using plutil, and read it using cat:

`plutil -convert xml1 OWASP.iGoat-Swift.plist`

![d6dec888899f4714798077897a6f2292.png](/assets/img/screenshots/iGoat/d6dec888899f4714798077897a6f2292.png)

### 4\. YAP Storage

In this exercise, you’ll exploit an app that unsafely stores sensitive data locally on the iOS device. In the lesson, the app stores data using a YAP Storage file.

YapDatabase is a popular open-source library for iOS and macOS that provides a high-performance, extensible database framework. It's designed to be used within iOS and macOS applications to manage and store structured data efficiently. Normally, the files are located in Library/Application Support or Library/Documents

Move to /var/mobile/Containers/Data/Application/610A25B0-5BF6-4BD0-A227-F51AD187523F/Library/Application Support

Here you will find YapDatabase.sqlite file. penm iot using sqlite3 YapDatabase.sqlite

![c22e6d16bd67d8ccfd0e6d4f69f665f9.png](/assets/img/screenshots/iGoat/c22e6d16bd67d8ccfd0e6d4f69f665f9.png)

### 5\. Realm Data Storage

In this exercise, you’ll exploit an app that unsafely stores sensitive data locally on the iOS device. In the lesson, the app stores data using a Realm Storage file.

Realm is a popular mobile database framework for iOS and other platforms. It provides a convenient way to work with and store data, and it offers features like object-oriented data modeling and automatic data synchronization. When you create a Realm database in your iOS app, it stores data in a file with a .realm extension. This file is the core storage for the app's data.

&nbsp;To easily find the files do the following search:  `find * | -i grep "realm"`

After that you will find /var/mobile/Containers/Data/Application/610A25B0-5BF6-4BD0-A227-F51AD187523F/Documents/default.realm

To open realm files, realm sutdio is needed, for that, i will [download](https://studio-releases.realm.io/latest/download/linux-appimage) it on Linux

`chmod +x Realm\ Studio-14.0.4.AppImage`

`./Realm\ Studio-14.0.4.AppImage`

Once opened, I will transfer the realm file using SCP to Linux

`scp -P 2222 root@localhost:/var/mobile/Containers/Data/Application/610A25B0-5BF6-4BD0-A227-F51AD187523F/Documents/default.realm ~/Downloads`

After that open the retrieved file in Realm Studio:

![a879e8a2963eabfc4c32daedcc09a251.png](/assets/img/screenshots/iGoat/a879e8a2963eabfc4c32daedcc09a251.png)

### 6\. CouchBase Storage

In this exercise, you’ll exploit an app that unsafely stores sensitive data locally on the iOS device. In the lesson, the app stores data using a CouchBase file.

Couchbase Mobile is a NoSQL database platform designed for mobile and edge computing applications. It provides a database solution for mobile and embedded systems that need to work offline or with intermittent connectivity. Files are typically located on /var/mobile/Containers/Data/Application/{App-UUID}/Documents

To easily find the files do the following search:

`find * | grep -i "couchbase"`

`find * | grep -i "cblite2"`

After finding the file mobile/Containers/Data/Application/610A25B0-5BF6-4BD0-A227-F51AD187523F/Library/Application Support/CouchbaseLite/couchbasedb.cblite2/db.sqlite3, open it using sqlite3:

![c21853f2da09a38caea743762acc76e9.png](/assets/img/screenshots/iGoat/c21853f2da09a38caea743762acc76e9.png)

### 7\. Cookie Storage

The cookies are stored in the device storage and an attacker with physical access to the device can steal the cookies.

In iOS, Cookie Storage is a component of the WebKit framework that allows you to manage and work with HTTP cookies. Cookies are small pieces of data sent by a web server to a web client (such as a web browser or a mobile app) and are typically used for maintaining user sessions, storing user preferences, and tracking user behavior on websites.

To easily find the files do the following search: `find * | grep ".binarycookies"`

Now, to open the binary cookie, use [BinaryCookieReader](https://github.com/as0ler/BinaryCookieReader)

Before that transfer the file using scp to the linux machine:

`scp -P 2222 root@localhost:/var/mobile/Containers/Data/Application/610A25B0-5BF6-4BD0-A227-F51AD187523F/Library/Cookies/com.OWASP.iGoat.binarycookies ~/Downloads`

Read the cookies and find the stored info

`python BinaryCookieReader/BinaryCookieReader.py com.swaroop.iGoat.binarycookies`

