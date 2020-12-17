
ICE on Java beginner log...

# Hello World service & client

## 1 define directory 
+---client
|   \---src
|       \---main
|           \---java
+---server
|   \---src
|       \---main
|           \---java
\---slice 

## 2 gradle init
gradle init
[basic]
[groovy]
--- settings.gradle ---
rootProject.name = 'printer'
include 'client'
include 'server'
--- build.gradle ---
plugins {
    id 'com.zeroc.gradle.ice-builder.slice' version '1.4.7' apply false
}
  
subprojects {
    apply plugin: 'java'
    apply plugin: 'com.zeroc.gradle.ice-builder.slice'
    
    slice {
        java {
            files = [file("../slice/Printer.ice")]
        }
    }
    repositories {
        mavenLocal()
    }
    dependencies {
        compile  'com.zeroc:ice:3.7.4'
    }
    jar {
        manifest {
            attributes(
                "Main-Class": project.name.capitalize(),
                "Class-Path": configurations.runtime.resolve().collect { it.getName() }.join(' ')
            )
        }
    }
}


# 3 Slice
module Demo
{
    interface Printer
    {
        void printString(string s);
    }
}

## 4 Server
--- PrinterI.java ---
public class PrinterI implements Demo.Printer
{
    public void printString(String s, com.zeroc.Ice.Current current)
    {
        System.out.println(s);
    }
}
--- server.java ---

public class Server
{
    public static void main(String[] args)
    {

        try(com.zeroc.Ice.Communicator communicator = com.zeroc.Ice.Util.initialize(args))
        {

            com.zeroc.Ice.ObjectAdapter adapter = communicator.createObjectAdapterWithEndpoints("SimplePrinterAdapter", "default -p 10000");

            com.zeroc.Ice.Object object = new PrinterI();

            adapter.add(object, com.zeroc.Ice.Util.stringToIdentity("SimplePrinter"));

            adapter.activate();

            communicator.waitForShutdown();

        }

    }

}



## 5 Client
public class Client
{
    public static void main(String[] args)
    {
        try(com.zeroc.Ice.Communicator communicator = com.zeroc.Ice.Util.initialize(args))
        {
            com.zeroc.Ice.ObjectPrx base = communicator.stringToProxy("SimplePrinter:default -p 10000");
            Demo.PrinterPrx printer = Demo.PrinterPrx.checkedCast(base);
            if(printer == null)
            {
                throw new Error("Invalid proxy");
            }
            printer.printString("Hello World!");
        }
    }
}

## 6 buid
gradle :client:build
gradle :server:client

## 7 Run
cp ice.3.7.4.jar server/build/libs/
java -jar server/build/libs/server.jar
java -jar client/build/libs/client.jar
