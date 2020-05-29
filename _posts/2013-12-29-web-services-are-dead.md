---
layout: post
title: "Web services are dead, long live web services!"
description: ""
category:
tags: []
date: 2013-12-29 15:51 UTC
---

ASMX web services have been defined a legacy technology for a while now. In fact, Microsoft itself put a disclaimer on the MSDN documentation page regarding ASMX.

![ASMX warning](/images/asmx_warning.png)

This is not going to be news to the most of us, but we still kept creating services with ASMX. The problem is that I keep seeing projects implementing web services with ASMX instead of WCF. So why are we working with a legacy technology, while Microsoft keeps recommending against it?

Because WCF is a pain. It can do everything, but it always felt clunky for something small and simple like a web service. Have you ever seen the standard configuration of a WCF service, the one that is added by default when you create a service?

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <system.serviceModel>
        <bindings>
            <basicHttpBinding>
                <binding name="BasicHttpBinding_IService1" closeTimeout="00:01:00"
                    openTimeout="00:01:00" receiveTimeout="00:10:00" sendTimeout="00:01:00"
                    allowCookies="false" bypassProxyOnLocal="false"
                    hostNameComparisonMode="StrongWildcard" maxBufferSize="65536"
                    maxBufferPoolSize="524288" maxReceivedMessageSize="65536"
                    messageEncoding="Text" textEncoding="utf-8" transferMode="Buffered"
                    useDefaultWebProxy="true">
                    <readerQuotas maxDepth="32" maxStringContentLength="8192"
                        maxArrayLength="16384" maxBytesPerRead="4096"
                        maxNameTableCharCount="16384" />
                    <security mode="None">
                        <transport clientCredentialType="None" proxyCredentialType="None"
                            realm="" />
                        <message clientCredentialType="UserName" algorithmSuite="Default" />
                    </security>
                </binding>
            </basicHttpBinding>
        </bindings>
        <client>
            <endpoint address="http://localhost:36906/Service1.svc" binding="basicHttpBinding"
                bindingConfiguration="BasicHttpBinding_IService1" contract="IService1"
                name="BasicHttpBinding_IService1" />
        </client>
    </system.serviceModel>
</configuration>
{% endhighlight %}

It makes me want to poke my eyes out. StrongWildCard? ReaderQuotas? It is a huge amount of unnecessary cryptic information. In fact, it is so bad that in the book "Programming WCF Services" there is an explicit essential coding guideline for configuration files: "Do not use SvcUtil or Visual Studio 2010 to generate a config file". What about sharing the session with ASP.NET? Yes, it might be frowned upon, but it worked out of the box with ASMX. Oh right, you have to extend the configuration setting the aspNetCompatibilityEnabled attribute to true. As if the configuration weren't large enough.

To make the matter worse, we are also deceived when we want to create a new web service. Suppose that we are asked to create a web service to expose a certain functionality of our application, when we go to Visual Studio we are faced with this:

![Add Item dialogue](/images/addwebservice.png)

Since we are about to create a web service we should probably choose the "Web Service" item from the list, right? Guess what that is? An ASMX Web Service. So much for legacy technology, this surely does not help the adoption of WCF services.

All in all, it has been difficult to convince developers to use WCF in the last years. Luckily Microsoft noticed, hence things have changed and with .NET 4.5 they simplified things a bit. Enter .NET 4.5 and the WCF "Simplification Features".

For starters, there is a new default configuration.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <system.serviceModel>
        <bindings>
            <basicHttpBinding>
                <binding name="BasicHttpBinding_IService1" />
            </basicHttpBinding>
        </bindings>
        <client>
            <endpoint address="http://localhost:36906/Service1.svc" binding="basicHttpBinding"
                bindingConfiguration="BasicHttpBinding_IService1" contract="IService1"
                name="BasicHttpBinding_IService1" />
        </client>
    </system.serviceModel>
</configuration>
{% endhighlight %}

Much less intimidating than the older one, isn't it? And we don't need to bother to configure the ASP.NET compatibility mode, since it is activated by default now, meaning that we can access ASP.NET session out of the box. There are some other improvements that you should check out on the [dedicated page on MSDN](http://msdn.microsoft.com/en-us/library/hh309266.aspx).
