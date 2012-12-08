---
title: scraping broken SSL pages with node.js
layout: post
---
To collect the data for [courseoff.com](https://courseoff.com) I scrape university websites on demand. This means that I have to perform http queries to the sites and then parse the resulting pages. Usually, this is not a problem as node.js has very simple APIs for performing both http and https requests. However, some schools have old webservers which make this task trickier.

Scraping a normal, up to date website, over SSL is not a problem; however, when you encounter a site with a broken web browser ssl implementation things go downhill fast.

You might see an error such as the following when trying to curl the url.

{% highlight text %}
curl: (35) error:14077417:SSL routines:SSL23_GET_SERVER_HELLO:sslv3 alert illegal parameter
{% endhighlight %}

This was strange to me as my browser loaded the page just fine. Chrome does however give a hint as to something being wrong. When I click on the green lock icon, I get the usual certificate notice as well as an additional piece of information:

> The connection had to be retried using SSL 3.0. This typically means that the server is using very old software and may have other security issues.

Clearly something is not right with the webserver here and that is what is causing curl, and subsequently my nodejs requests, to fail.

Doing some basic digging, I was able to find this nice little tidbit in the openssl documentation:

> **SSL\_OP\_ALLOW\_UNSAFE\_LEGACY_RENEGOTIATION**
>
> Allow legacy insecure renegotiation between OpenSSL and unpatched clients or servers. See the **SECURE RENEGOTIATION** section for more details.

Now that I had this option to try, I had to figure out a way to make node.js use it. Reading some code led me to understand that the 'Agent' is the object which handles the options for http(s) requests which are ultimately passed to the SSL context. When creating the agent, you pass in an options object, this object contains two fields which need to be used for this case: secureProtocol and secureOptions.

> **secureProtocol** specifies which negotiation method to use  
> **secureOptions** are the flags passed to the context to change the behavior

My final agent object had the following form:
{% highlight javascript %}
var agent = new Agent({
  secureProtocol: 'SSLv3_method',
  secureOptions: require('constants').SSL_OP_DONT_INSERT_EMPTY_FRAGMENTS
});
{% endhighlight %}

My https request options object just need to include the 'agent' key and have it set to the agent object I created. This filtered down to the underlying ssl context and set the context options. The request was a success!

This will only work with node.js >= 0.6.3 because the constants are only available after that. Other constants from the bug workaround section of the openssl documentation are also available for your needs.

## References

### ssl docs
http://www.openssl.org/docs/ssl/SSL\_CTX\_set_options.html

