---
title: Introduction to hooking with Frida
tags: Frida Hooking
categories: Mobile Android
---
In this entry I will try to explain the of hooking in mobile applications. This is technique employed to intercept and modify the behavior of functions. For this tests I will be using Frida, an open-source toolkit to inject their scripts into running processes, enabling real-time manipulation and analysis.

# Introduction to Frida

### What is Frida?

Frida is like having a magic lens for mobile apps. It allows developers and pentesters to see the intricate details of apps running on your smartphone or computer.

With Frida, you can inject custom JavaScript code into the running app. For example, you could use Frida to intercept network requests and responses in real-time, allowing you to inspect the data being sent and received by the app. This dynamic analysis provides insights into the app's behavior that static analysis alone might miss.

### Setting up Frida

Install Frida using pip:

`pip install frida-tools`

After that, download [frida server](https://github.com/frida/frida/releases) in the device, depending in you architecture you should download one or the other:

![16d5f5311d1790238d5f7ad14d4bbeb5.png](/assets/img/screenshots/hooking_frida/16d5f5311d1790238d5f7ad14d4bbeb5.png)

Since I'm using an x86 Genymotion emulator, I will download the x86 version. If you want to check my set up visit this [blog post](https://v3l4r10.github.io/posts/Android-pentesting-lab-set-up/) where I explain how to do it

If you don't know about your device's architecture, use this command:

`adb shell getprop ro.product.cpu.abi`

![e9a1e1bcd5660ee194ed68223d26c7f3.png](/assets/img/screenshots/hooking_frida/e9a1e1bcd5660ee194ed68223d26c7f3.png)

Once downloaded, decompress the file and move it to /data/local/tmp Android folder and run Frida server:

![ea7ec26bf3f50d62ccd02dbe91f7a131.png](/assets/img/screenshots/hooking_frida/ea7ec26bf3f50d62ccd02dbe91f7a131.png)

`xz -d frida-server-16.1.8-android-x86.xz`

`mv frida-server-16.1.8-android-x86 frida-server`

`adb push frida-server /data/local/tmp`

`adb shell chmod +x /data/local/tmp/frida-server`

`adb shell "/data/local/tmp/frida-server &"0`

## Basic commands

List all the installed applications: `frida-ps -Uai  |  grep  <appName>`

Attach to an application: `frida -U -f <AppID>`

# Introduction to objection

### What is objection?

Objection is a runtime mobile exploration toolkit, powered by Frida. It was built with the aim of helping assess mobile applications and their security posture without the need for a jailbroken or rooted mobile device.

### Setting up objection

Install objection using pip

`pip3 install objection`

To connect objection to a runtime application, first list all the applications using frida and copy the desired PID:

`frida-ps -Uai  |  grep  <appName>`

![4c30d0532c882d07d2af7eacfbd9942a.png](/assets/img/screenshots/hooking_frida/4c30d0532c882d07d2af7eacfbd9942a.png)

After that, run the objection with the app identifier:

`objection --g <PID> explore`

### Basic commands

List the enviroment paths: env

Attempts to disable SSL Pinning on Android devices.: `android sslpinning disable`

Attempts to disable root detection on Android devices.: `android root disable`

Command execution: `android shell_exec whoami`

List activities, receivers and service: `android hooking list activities`

Search Classes: `android hooking search classes <appIdentifier>`

List all loaded classes: `android hooking list classes`

Hooking (watching) a class: `android hooking watch class_method asvid.github.io.fridaapp.MainActivity --dump-args --dump-backtrace --dump-return`

Dump all memory: `memory dump all <local destination>`

Search and write in memory:

`memory search "<pattern eg: 41 41 41 ?? 41>" (--string) (--offsets-only)`

`memory write "<address>" "<pattern eg: 41 41 41 41>" (--string)`

# H00k1ng!

Android hooking involves intercepting and manipulating the execution flow of Android applications at runtime, typically using tools like Frida to modify or analyze their behavior.

## Application analysis

Before hooking a method, we have to understand how the application works, how to hooke a hookeable method and how to do it, so lets test it!

The functionality is quite simple, it asks us to enter a number and returns an output:

![b4b780ff4cf73e64c726127477019ffd.png](/assets/img/screenshots/hooking_frida/b4b780ff4cf73e64c726127477019ffd.png)

After the dynamic analysis, we will use [jadx](https://github.com/skylot/jadx) to decompile the code and understand the backend. Open jadx and decompile the APK by using the GUI:

`./Android/jadx/build/jadx/bin/jadx-gui`

Once opened, go to MainActivty file. As it names indicatd, this file represents the main activity of the application. It includes the activity's lifecycle methods (onCreate, onStart, onResume, etc.) and the logic associated with handling user interface elements and user interactions.

![6d5f8eb443a6f3e0a81dfd957b404b35.png](/assets/img/screenshots/hooking_frida/6d5f8eb443a6f3e0a81dfd957b404b35.png)

Briefly, what application does the following:

1.  When the app starts (onCreate), it generates a random number between 1 and 100 (get\_random) and sets up UI components
2.  After button click, it checks if the entered number and passes that integer to a method called 'check'.
    
3.  Along with the input number, the random integer value is passed. Matches a condition based on the random number: 
    
    ```
    void check(int i, int i2)
    
    if ((i * 2) + 4 == i2)
    ```
    
4.  f conditions are met, it shows a success message and transforms the "AMDYV{WVWT\_CJJF\_0s1}" string. If not, it shows a failure message.
    

## Hooking a method

Knowing how the code works, we can conclude that there are 2 ways to obtain the random number:

- Hooking the get\_random() function.

As t the random number is generated within the get\_random() method, we can hook this method to obtain the returned value. Other option is to overwrite the return value because get\_random() returns the value we provided to the check() function.

- Hooking the check() function.

The check() method takes arguments that include the random number, so we can consider hooking this method to intercept the arguments and reveal the random number.

### Template:

```Java
Java.perform(function (){

var <class_reference>= Java.use("<package_name>.<class>");  
<class_reference>.<method_to_hook>.implementation = function(<args>){

/*  
OUR OWN IMPLEMENTATION OF THE METHOD  
*/

}

})
```

*Java.perform* is a Frida function that establishes a specialized context for script interaction with Java code in Android applications. It serves as a gateway, enabling access and manipulation of the Java code executing within the app. Once within this context, various actions such as method hooking or accessing Java classes become possible.

To interact with a specific Java class, you utilize the *Java.use* function with the following syntax:

`var <class_reference> = Java.use("<package_name>.<class>");`

Here, &lt;class\_reference&gt; becomes a variable representing the Java class in the target Android application. The Java.use function takes the class name as an argument, where &lt;package\_name&gt; denotes the application's package name, and &lt;class&gt; denotes the desired class for interaction.

To hook a method within the chosen class, you use the following structure:

`<class_reference>.<method_to_hook>.implementation = function(<args>) {`

Within the specified class, &lt;class\_reference&gt;.&lt;method\_to\_hook&gt; references the method you intend to hook. This is the point where you define custom logic to execute when the hooked method is called. &lt;args&gt; stands for the arguments passed to the function.

### Hooking get\_random() \[option 1\]

First of all, in order to get the package name, we will run `frida-ps -Uai` to list all the packages:

![4c30d0532c882d07d2af7eacfbd9942a.png](/assets/img/screenshots/hooking_frida/4c30d0532c882d07d2af7eacfbd9942a.png)

Going back to the code we should set the class\_reference to *MainActivity*:

![e261baeebf31652953bb9a3d73aca95f.png](/assets/img/screenshots/hooking_frida/e261baeebf31652953bb9a3d73aca95f.png)

Next, we will modify the script to include our custom implementation of the method. The method to hook is *get\_random.*

Basically, In our script, we replaced the original implementation of the get\_random() method with our custom one, but we did not provide a return value. This return value is assigned to the i variable and is used in the check() function. So, let's try providing a return value. You can use any value. Our script should be something like the following, save it to *hook.js.* 

```Java
Java.perform(function (){
 
var a= Java.use("com.ad2001.frida0x1.MainActivity");
a.get_random.implementation = function(){
    
console.log("This method is hooked");
console.log("Returning 1")
    
return 1;

}

})
```

Once we have it, run frida 

`frida -U -f com.ad2001.frida0x1 -l ./hook.js`

![c5258597345788d49c0bbbe1b4376923.png](/assets/img/screenshots/hooking_frida/c5258597345788d49c0bbbe1b4376923.png)

Now, 1 will be passed to the check() function. Let's calculate the value so that we can satisfy the if check and obtain the flag.

if ((i \* 2) + 4 == i2) --> 1 \* 2 +4 = 6. If we enter 6 in the form, we can obtain the flag

![89609315f5a6708bef34b652bfbd770e.png](/assets/img/screenshots/hooking_frida/89609315f5a6708bef34b652bfbd770e.png)

### Hooking get\_random() \[option 2\]

Now, let's try to retrieve the originally generated random value. To achieve this, we need to obtain the return value from the original *get\_random()*. 

In this case, the first snippet keeps the same, but instead of modifying the return value directly, it calls the original *get\_random* method using *this.get\_random()* and logs its return value.

```Java
Java.perform(function (){
 
var a= Java.use("com.ad2001.frida0x1.MainActivity");
a.get_random.implementation = function(){
    
console.log("This method is hooked");
var ret_val = this.get_random();
console.log("The return value is "+ ret_val);  
return ret_val; //returning the original random value from the get_random method

}

})
```

After crafting the script, run frida and get the value:

![8089089a1358e7d676931d34b53cb863.png](/assets/img/screenshots/hooking_frida/8089089a1358e7d676931d34b53cb863.png)

\*Remember to bypass the check() function in order to get the flag:

if ((i \* 2) + 4 == i2)  --> 36 \*2 +4 = 76

### Hooking check() 

In order to hook the method, lets read the code again: 

![cfc2a0eb5cbc820b7c79937810406bff.png](/assets/img/screenshots/hooking_frida/cfc2a0eb5cbc820b7c79937810406bff.png)

Upon inspecting the parameters of the check function, the initial parameter, i, signifies the random number, while the second one, i2, represents the user-inputted number. Let's use Frida to intercept and display both of these parameters.

When intercepting methods with arguments, it's crucial to define the expected argument types using the overload(arg\_type) keyword. Ensure that your implementation includes these specified arguments when hooking the method. In our case, the check() function accepts two integer arguments, and we can indicate this as follows:

```Java
Java.perform(function (){
 
var a= Java.use("com.ad2001.frida0x1.MainActivity");
a.check.overload('int', 'int').implementation = function(a,b){  //The function takes two arguments ;check(random,input)
console.log("The random number is "+a);
console.log("The user input is "+b);
    
}

})
```

After getting these args, we need to guarantee the uninterrupted operation of the check function to generate the flag. The primary goal is to extract the random value without compromising the overall functionality of the function. Consequently, we can straightforwardly invoke the original check() function,

```
Java.perform(function() {
    var a = Java.use("com.ad2001.frida0x1.MainActivity");
    a.check.overload('int', 'int').implementation = function(a, b) {
        // The function takes two arguments; check(random, input)
        console.log("The random number is " + a);
        console.log("The user input is " + b);
        this.check(a, b); // Call the check() function with the correct arguments
    }
});
```

After that, run frida again frida -U -f com.ad2001.frida0x1 -l ./hook.js

![029bb579a3de41fb66cc57078959bd7e.png](/assets/img/screenshots/hooking_frida/029bb579a3de41fb66cc57078959bd7e.png)

In the realm of Frida, mastering the art of method hooking involves tapping into the essential principles of extracting both arguments and return values. This process serves as the gateway to unlocking the full potential of Frida's capabilities. As we journey through this series, we'll delve deeper into the intricacies of Frida, exploring its powerful features and uncovering the myriad possibilities it offers for application analysis and manipulation.

# Resources:

https://github.com/DERE-ad2001/Frida-Labs

[https://learnfrida.info/advanced\\\_usage/](https://learnfrida.info/advanced%5C_usage/)

https://codeshare.frida.re/

https://github.com/asvid/FridaApp

https://github.com/sensepost/objection

https://book.hacktricks.xyz
