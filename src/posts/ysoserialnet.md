---
layout: post.njk
title: so you think you found a deserialization sink?
description: exploiting deserialization vectors the easy way, and the hard way. but mostly the easy way.
date: 2026-07-08
tags: posts
permalink: "/blog/cereal/"
---

##### *these thoughts were handcrafted without the use of ai. for the adhd among us, key points are outlined at the end.*

# 0. approach

i want to go over a few topics based on some research i've been doing lately:

1. identifying vulnerable sinks
2. exploitation the easy way - using ysoserial.net on linux (debian, for kali/parrot users)
3. exploitation the hard way

this will be mostly practical, some theoretical. while veterans of the craft may not get a lot of value here, i won't be explaining the entire concept of serialization for beginners.
the intended audience is for those who understand serialization at a basic level, understand it can be exploited, but lack the tools/knowledge to do so themselves.

we will be focusing on **.net** vectors, but the 'easy way' maps 1:1 to exploiting java deserialization via the og ysoserial.


# 1. let that sink in

i consider there to be 3 types of sinks:

## the golden sinks

if you find `new BinaryFormatter();` in an application running in 2026, congratulations. you're about to mint your very own cve.

[microsoft](https://learn.microsoft.com/en-us/dotnet/standard/serialization/binaryformatter-security-guide) specifically calls out *hard-banned* serializer types due to their ability to perform unrestricted polymorphic deserialization. 

>`BinaryFormatter` specifically isn't even packaged in the .net runtime as of .net 9, and anything above .net 9+ has to install the unsupported `System.Runtime.Serialization.Formatters` package and set a scary `EnableUnsafeBinaryFormatterSerialization` flag.

golden sinks include:
- `SoapFormatter`
- `NetDataContractSerializer`
- `LosFormatter`
- `ObjectStateFormatter`

>*"back up. wtf is unrestricted polymorphic deserialization?"*

i said i wasn't going to go over the concepts but i agree this sounds hard to understand. it's not. it means the malicious input can specify target classes arbitrarily, all the time.

if you look at the implementation of the [`BinaryFormatter` class](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.serialization.formatters.binary.binaryformatter?view=net-11.0-pp), you'll notice the `Deserialize` method takes 1 parameter: `Stream`. unlike other serialization classes, the type decision is fundamentally payload-driven.

this property is what makes these serializers "**golden sinks**" - they are insecure by design, and will deserialize any object you throw at it. the easy mode of exploitation we'll discuss is more than likely going to entail a l33t copy/paste after running ysoserial.

>*"but what if i strictly bind the serialized type names to the expected runtime types?"*

ok, so you read the microsoft docs but you didn't see the "(insecure)" right before that. that's fine. we'll discuss that in the third type of sink, but it's still bad.

## the porcelain sinks

these are the sinks that could be perfectly normal, or could be hiding a secret puddle of mold and rot underneath. a few of these are offered as preferred alternatives to golden sinks by microsoft, but they still allow for exploitation in certain cases.

some porcelain sinks:
- `XmlSerializer`
- `Json.NET`
- `DataContractSerializer`

let's look at a couple of rough examples:

### safe
```csharp
static void Main(string[] args) {
    using var SafeObject = new StringReader(UserProvidedXml);

    XmlSerializer slzr = new XmlSerializer(typeof(SafeStuff));
//                                           ^ this is important
    slzr.Deserialize(SafeObject);
}

public class SafeStuff {
    private String _stuff;

    public String stuff
    {
        get { return _stuff; }
        set { _stuff = value; }
    }
}
```
in the above, a user passing an object of type `DangerousStuff` would throw an exception at deserialization.


### unsafe
(this code mocks a lot of xml parsing)
```csharp
static void Main(string[] args) {
    using var UnsafeObject = new StringReader(UserProvidedXml);
    string UserTypeName = UnsafeObject.GetAttribute("UserType");

    XmlSerializer slzr = new XmlSerializer(Type.GetType(UserTypeName));
//                                                 ^ this is dangerous
    slzr.Deserialize(UnsafeObject);
}

public class DangerousStuff {
    private String _command;
    public String command
    {
        get {return _command; }
        set {
            _command = value;
            ExecuteCommand();
        }
    }

    private void ExecuteCommand()
    {
        // execute some process with _command;
    }
}
```
in the above exmaple, we allow arbitrary types to be passed based on the user input. this is really no different than a *golden sink* in the sense that we control any type passed to it.

> note: `XmlSerializer` has some interesting properties around what it can serialize - public properties and fields of an object. this leads to us having to use some potentially complex gadget chains if we want to go the hard way.

essentially, **porcelain** sinks are those that can be conditionally dangerous. finding a call to `Json.NET` doesn't mint you a cve, but if they set `TypeNameHandling` to anything other than `None`, you're looking at some good ol' mold.

the key to porcelain sinks is to understand what makes them dangerous: allowing deserialization of arbitrary types. however, this isn't always necessary. the third sink is where restricted type deserialization is found (and admittedly, is a bit out of my research zone)

## the prison sinks (steel?)

i have limited research in this area so far, but it's still theoretically possible for deserialization to be exploitable in a restricted type situation. you just have to pray that restricted type has some reachable dangerous behaviors. personally, i'd look for more golden and porcelain sinks before sticking around here.

even if rce isn't possible, primitives can lead to things like memory/cpu exhaustion, parser bombs, etc. which are still nasty. to address the "strict binder" question you might have asked earlier, you are still allowing a potentially dangerous type to be deserialized. imagine your `AllowedType` has an `[OnDeserialized]` callback that can be abused in some way. risk reduced, but not gone.

if you're a masochist, this is where the "hard way" will come in handy - you certainly won't be able to use the "easy way" for exploitation as the required types are likely going to be pretty custom to the application.

---

# 2. i came here to hack, not read (easy mode)

ok, let's walk through ysoserial.net and hack a little app.

i'm going to imagine you're not hacking on bare metal windows (if you are, i'm sorry. vmware is free btw.) and walk through getting ysoserial.net working on debian-based systems.

## installing wine, winetricks, mono-complete
- ysoserial.net requires a .net environment. many gadgets rely on .net 4.8.
- wine is a translator for windows api calls to allow for .net to run on our linux os.
- winetricks helps out with installing .net 4.8.
- .net 4.8 requires 32-bit wine prefix.

i could see why people get stuck here!
```
$ wget https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
$ chmod +x winetricks
$ sudo mv winetricks /usr/local/bin

$ sudo apt update
$ sudo dpkg --add-architecture i386
$ sudo apt install mono-complete wine32:i386

// optional, if you already had 64-bit wine installed 
$ unset WINEPREFIX WINEARCH
$ rm -rf ~/.wine

$ WINEARCH=win32 wineboot -i
$ WINEARCH=win32 winetricks dotnet48
```

wait for all the long installation stuff to be done. maybe scary errors in terminal. ignore.

## installing ysoserial.net

download latest release, unzip:
```
https://github.com/pwntester/ysoserial.net/releases
```

now you're ready to point and shoot.

## the vulnerable app

> this application was built with ai to demonstrate the poc. the full source can be found on [my github](https://github.com/zylideum/deserialization-demos).


### golden sink

the application contains one golden sink that should set off those cve-hunting alarms: `NetDataContractSerializer`. if we follow its instantiation, we can see a user-controlled `reader` object is passed to `serializer.ReadObject();` in the `DeserializeFromBase64Header()` method.

![the netdatacontractserializer sink](/assets/images/post2/goldensink.png)

since it's a **golden sink**, there is no type information provided to the method, and any type is accepted. we can fire up ysoserial.net and get some rce pretty quickly.

1. `cd` to your unzipped ysoserial.net directory
2. `wine ysoserial.exe -f NetDataContractSerializer -g TypeConfuseDelegate -c "calc.exe"`
    - when using ysoserial.net, you'll want to identify two things: the sink (`-f NetDataContractSerializer`) and the gadget (`-g TypeConfuseDelegate`). you should use the project readme to understand what certain gadgets require, but if you're lazy you can always shotgun gadgets and see what sticks. probably don't do that on an actual engagement.
3. tailor payload to application. in this case, the app expects input as a base64-encoded header.
4. encode what ysoserial.net spits out for us (`-o` flag does this!), inject into the vulnerable source (`X-Demo-Object` header), and voila!
5. do it again but with a reverse shell instead of a calculator...

![ysoserial header payload for rce](/assets/images/post2/goldenexploit.png)

didn't i tell you that was easy?

in a **porcelain sink** situation, we might have to assess the payload a bit more. for example, ysoserial.net will generate `ObjectDataProvider` payloads with a `<root>` tag. if the app expects a different xml root or certain nodes, we'd have to tweak this. but ysoserial.net will still cover the necessary gadget chaining and type handling.

### sink checklist
- what controls the serialized object?
- which formatter is being used?
- do we control the type?
- is there a binder or type allowlist?

---

# 3. i came here to grind, not hack (hard mode)

in my experiense so far, custom gadget chains are more necessary in java applications than .net. `ObjectDataProvider` is a super common gadget included in `System.Windows.Data` and there are often widely-used classpaths in .net that are exploitable.

if you look at the og java ysoserial repo, you'll notice 'dependencies' are called out, unlike in ysoserial.net. this is because target classpaths or versions may not exist in the application. there are totally different gadgets for java deserialization based on `commons-collections:3.1` vs. `commons-collections4:4.0` for example.

the point of the above is to say that sometimes, both versions of ysoserial may not include usable gadgets for the application. this is where it's up to us as researchers to hunt for our own.

this is where i'm reaching the limit of my teachable value, as i haven't had an instance where discovering a novel gadget chain was pertinent yet. however, i can point to the earlier unsafe usage of the `XmlSerializer` example to demonstrate.

say for whatever reason, the `XmlSerializer` only accepted certain types and rendered ysoserial moot. assuming one of the types is of `DangerousStuff`, we could tamper with our serialized object to execute an arbitrary command. this is an extremely basic example, but you can imagine having to find a nested type or dangerous setters to utilize to our advantage. perhaps there is a dangerous method elsewhere that we can wrap in our `ObjectDataProvider`/`ExpandedWrapper` class that ysoserial.net makes use of.

my goal is to continue exploiting deserialization until the day ysoserial no longer works, and i'm forced to do it the hard way!

---

if you learned anything at all from this, please consider [following me](https://ashtonr.com/contact/)!

# 4. key points

> - there are different types of deserialization sinks that vary in danger (which translates to ease of exploitation)

> - section 2 goes over a practical user guide for ysoserial.net, which translates to java's ysoserial, and how to use it for the dangerous sinks.

> - deserialization paths that restrict types can still be dangerous - it's up to us to find out if we can still find rce through novel gadgets or consider alternative impact like resource exhaustion. 