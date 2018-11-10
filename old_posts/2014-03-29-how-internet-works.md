---
id: 214
title: How Internet Works
author: rahular
layout: post
excerpt: The nitty gritties of how the internet works and a brief explanation of what happens inside the computer when you navigate to a website 
guid: http://rahular.com/?p=214
permalink: /how-internet-works/
categories:
  - Misc
---
So people keep talking about how the internet has changed the world and how they cannot imagine life without it. But most of them do not understand how the internet works. Here I try to explain it&#8217;s inner workings to the best of my knowledge (This is in no way a complete description of what is going on).

So what do we do when we want to log on to Facebook assuming the browser is already open? Type in the URL right? So what happens when you type? Here&#8217;s what!

  * A mechanical switch short-circuits a pull-up R1 resistor end to ground
  * A special multiplexer translates it into a message to reduce the number of wires
  * The message interpreted by a CPU embedded in the keyboard
  * It is then translated to a USB protocol message and modulated as a series of electric impulses of alternating voltage between 0V and 5V
  * A USB receiving hub measures the samples line voltage periodically
  * The Host Hub Controller translates the message to bits
  * These bits enters PC through the USB bus controller, connected to PCIE bus through a combination of IRQ notifications and a DMA transfer issued by the Bus Driver
  * The Bus Driver interprets the message and forwards it along the driver stack, ultimately to an HID driver
  * The HID driver talks to the OS (assume Windows), ultimately resulting in a window message sent to a window belonging to the browser* *process
  * WM\_KEYDOWN is translated to WM\_CHAR by *DefWindowProc()*. While key is down, multiple WM_CHARs may be created.
  * The browser application catches WM_CHAR to add another character to the URL bar and issues re-rendering of UI
  * UI rendering engines translate unicode codepoints to a graphical image by loading respective fonts
  * A graphics engine computes the new image of the whole area to avoid flicker, and puts it pixel-by-pixel to the screen

That was straightforward. Right? :) Once you have typed the whole URL and pressed the *enter* key, the page starts to load. Almost instantaneously. In this *almost instantaneous* time, these things happen.

- The browser&#8217;s initial task is to figure out the IP address for Facebook. The DNS lookup proceeds as follows : 
      * **Browser cache**: The browser caches DNS records for some time. Interestingly, the OS does not tell the browser the time-to-live for each DNS record, and so the browser caches them for a fixed duration (varies between browsers, 2 – 30 minutes). If the browser cache does not contain the desired record, then all of the below caches are checked.
      * **OS cache** : The browser makes a system call (<em>gethostbyname</em> in Windows). The OS has its own cache.
      * **Router cache** : The request continues on to your router, which typically has its own DNS cache.
      * **ISP DNS cache** : The next place checked is the cache ISP’s DNS server. With a cache, naturally.
      * **Recursive search** : Your ISP’s DNS server begins a recursive search, from the root name server, through the<em> .com</em> top-level name server, to Facebook’s name server. Normally, the DNS server will have names of the<em> .com</em> name servers in cache, and so a hit to the root name server will not be necessary.
- Once it gets the IP address, the browser sends a HTTP request to the web server.We can be pretty sure that Facebook’s homepage will not be served from the browser cache because dynamic pages expire either very quickly or immediately (expiry date set to past). So, the browser will send this request to the Facebook server : 

``` 
GET http://facebook.com/ HTTP/1.1
Accept: application/x-ms-application, image/jpeg, application/xaml+xml, ...
User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; ...
Accept-Encoding: gzip, deflate
Connection: Keep-Alive
Host: facebook.com
Cookie: datr=1265876274-...; locale=en_US; lsd=WW...; c_user=2101...
```
    
- The <em>GET</em> request names the URL to fetch: “http://facebook.com/”. The browser identifies itself (User-Agent header), and states what types of responses it will accept (Accept and Accept-Encoding headers). The connection header asks the server to keep the TCP connection open for further requests.
    
- The request also contains the cookies that the browser has for this domain. As you probably already know, cookies are *key-value* pairs that track the state of a web site in between different page requests. And so the cookies store the name of the logged-in user, a secret number that was assigned to the user by the server, some of user’s settings, etc. The cookies will be stored in a text file on the client, and sent to the server with every request.
    
- The Facebook server responds with a permanent redirect. <span style="line-height: 1.5;">This is the response that the Facebook server sent back to the browser request:

```
HTTP/1.1 301 Moved Permanently
Cache-Control: private, no-store, no-cache, must-revalidate, post-check=0,
      pre-check=0
Expires: Sat, 01 Jan 2000 00:00:00 GMT
Location: http://www.facebook.com/
P3P: CP="DSP LAW"
Pragma: no-cache
Set-Cookie: made_write_conn=deleted; expires=Thu, 12-Feb-2009 05:09:50 GMT;
      path=/; domain=.facebook.com; httponly
Content-Type: text/html; charset=utf-8
X-Cnection: close
Date: Fri, 12 Feb 2010 05:09:51 GMT
Content-Length: 0
```
        
- The server responded with a 301 Moved Permanently response to tell the browser to go to “http://www.facebook.com/” instead of “http://facebook.com/”.
        
- The browser follows the redirect. The browser now knows that "http://www.facebook.com/" is the correct URL to go to, and so it sends out another GET request:

```
GET http://www.facebook.com/ HTTP/1.1
Accept: application/x-ms-application, image/jpeg, application/xaml+xml, ...
Accept-Language: en-US
User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; ...
Accept-Encoding: gzip, deflate
Connection: Keep-Alive
Cookie: lsd=XW...; c_user=21...; x-referer=...
Host: www.facebook.com
```
            
The meaning of the headers is the same as for the first request.
            
- The server 'handles' the request. The server will receive the GET request, process it, and send back a response. This may seem like a straightforward task, but in fact there is a lot of interesting stuff that happens here – even on a simple site like my blog, let alone on a massively scalable site like Facebook.
                  
**Web server software : **The web server software (e.g., IIS or Apache) receives the HTTP request and decides which request handler should be executed to handle this request. A request handler is a program (in ASP.NET, PHP, Ruby, etc) that reads the request and generates the HTML for the response.In the simplest case, the request handlers can be stored in a file hierarchy whose structure mirrors the URL structure, and so for example &#8220;<http://example.com/folder1/page1.aspx>&#8220; URL will map to file /httpdocs/folder1/page1.aspx. The web server software can also be configured so that URLs are manually mapped to request handlers, and so the public URL of page1.aspx could be &#8220;http://example.com/folder1/page1&#8243;.

**Request handler : **The request handler reads the request, its parameters, and cookies. It will read and possibly update some data stored on the server. Then, the request handler will generate a HTML response.

The server sends back a HTML response. Here is the response that the server generated and sent back  :

```
HTTP/1.1 200 OK
Cache-Control: private, no-store, no-cache, must-revalidate, post-check=0,
    pre-check=0
Expires: Sat, 01 Jan 2000 00:00:00 GMT
P3P: CP="DSP LAW"
Pragma: no-cache
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
X-Cnection: close
Transfer-Encoding: chunked
Date: Fri, 12 Feb 2010 09:05:55 GMT

2b3
��������T�n�@����
...
```
                
The entire response is 36 kB, the bulk of them in the byte blob at the end that I trimmed.
                
The **Content-Encoding** header tells the browser that the response body is compressed using the *gzip* algorithm. After decompressing the blob, you’ll see the HTML you’d expect:
                
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"   
      "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" 
      lang="en" id="facebook">
<head>
<meta http-equiv="Content-type" content="text/html; charset=utf-8" />
<meta http-equiv="Content-language" content="en" />
...
```
                
In addition to compression, headers specify whether and how to cache the page, any cookies to set (none in this response), privacy information, etc.
                
- The browser begins rendering the HTML. Even before the browser has received the entire HTML document, it begins rendering the website.
- The browser then sends requests for objects embedded in HTML. As the browser renders the HTML, it will notice tags that require fetching of other URLs. The browser will finally send a GET request to retrieve each of these files.

Here are a few URLs that a visit to Facebook retrieved :

                    
```
Images
http://static.ak.fbcdn.net/rsrc.php/z12E0/hash/8q2anwu7.gif
http://static.ak.fbcdn.net/rsrc.php/zBS5C/hash/7hwy7at6.gif
...

CSS style sheets
http://static.ak.fbcdn.net/rsrc.php/z448Z/hash/2plh8s4n.css
http://static.ak.fbcdn.net/rsrc.php/zANE1/hash/cvtutcee.css
...

JavaScript files
http://static.ak.fbcdn.net/rsrc.php/zEMOA/hash/c8yzb6ub.js
http://static.ak.fbcdn.net/rsrc.php/z6R9L/hash/cq2lgbs8.js
...
```
                    
Each of these URLs will go through process a similar to what the HTML page went through. So, the browser will look up the domain name in DNS, send a request to the URL, follow redirects, etc.
                    
However, static files – unlike dynamic pages – allow the browser to cache them. Some of the files may be served up from cache, without contacting the server at all. The browser knows how long to cache a particular file because the response that returned the file contained an Expires header. Additionally, each response may also contain an ETag header that works like a version number – if the browser sees an ETag for a version of the file it already has, it can stop the transfer immediately.
                    
The browser sends further asynchronous (AJAX) requests. In the spirit of Web 2.0, the client continues to communicate with the server even after the page is rendered. For example, Facebook chat will continue to update the list of your logged in friends as they come and go. To update the list of your logged-in friends, the JavaScript executing in your browser has to send an asynchronous request to the server.

The asynchronous request is a programmatically constructed GET or POST request that goes to a special URL. In the Facebook example, the client sends a POST request to &#8220;http://www.facebook.com/ajax/chat/buddy_list.php&#8221; to fetch the list of your friends who are online.This pattern is sometimes referred to as “AJAX”, which stands for “Asynchronous JavaScript And XML”, even though there is no particular reason why the server has to format the response as XML. For example, Facebook returns snippets of JavaScript code in response to asynchronous requests.
                    
That&#8217;s a lot of work! But we didn&#8217;t even touch upon how these calls, packets, etc travel back and forth within the internet. For that we have to start with the OSI model&#8217;s physical layer, how electrons travel in a circuit and some other good stuff. But that&#8217;s for another day!
                    
#### Sources

* http://goo.gl/5mRQau
* http://goo.gl/OGWyrT
