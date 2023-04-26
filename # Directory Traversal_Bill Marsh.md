# Directory Traversal
<!-- Hola Amigo! Here are the edits I made using visualstudio, enjoy! //-->
In this section we're going to talk about a type of attack known
as _Directory Traversal_. 

The types of parameters that most often fall prey to a Directory
Traversal attack are those that work with files on the system. This
can be something as mundane as an image or as exotic as a file itself.
Further, any parameters that manage loading objects will be targets
for us.
<!-- Line 8: Punctuation. Coma is missing after the word image. Line 9:Replace the word “further” with “furthermore”.Line 10: Eliminate the phrase “for us” at the end of the paragraph as it is not necessary. //-->

Consider the below, where we are located in the <fp>/home/kali</fp> directory. 

<!-- Line 13: Myabe eliminate the phrase “consider the below” and replace with “Consider the following,” or “In this example, we are located in the..” //-->

In the below commands we are instructing Linux to perform the <pr>cat</pr>
command on a file two directories back from our current location.

```
kali@kali:~$ pwd
/home/kali
                           
kali@kali:~$ cat ../../etc/passwd                                  
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:101:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
. . .
```

After traversing two directories back, we move into the <pr>/etc/</pr>
directory and land on the passwd file.

<!--Line 50: Does Traversing need to have a preposition after it? Try using "traversing from" //-->

When working with a Windows machine, we can specify both the <pr>../</pr> and
<pr>..\\</pr> traversal strings. Windows considers these statements as one
in the same.[^DirTrav2]

Furthermore, with regards to the verification of a Directory
Traversal on Windows, we are presented with many potential
candidate files. The <pr>/etc/passwd</pr> equivalent so-to-speak
or the file which is most likely going to be present on the target
machine is <fp>C:/Windows/win.ini</fp>. We can always take a note
from file inclusion wordlists with regards to confirmation of
Directory Traversal on a target Windows machine by exploring known
file-inclusion wordlists.

<!-- Line 59: Replace “with regards” to “in regards.” Line 61: Eliminate the phase so-to-speak as it is not necessary.<//--> 

With regards to normalization of traversal strings, often times
web clients will resolve an endpoint before we even send a request.

For example, lets suppose that we use _curl_ on the megacorpone
domain. We will provide _curl_ a specific, known URI
(<fp>/about.html</fp>), but append a single traversal string and
"<fp>index.html</fp>" (<fp>/../index.html</fp>), then examine the results of
the GET method.

<!-- Line 73: Replace “lets” with “let’s”. Curl should also be capitalized. Line 75: Why is index.html used twice and why is one in parentheses? Do we need both? //-->

```
kali@kali:~$ curl -v http://www.megacorpone.com/about.html/../index.html |grep -i 'GET'
* Connected to <fp>www.megacorpone.com (149.56.244.87) port 80 (#0)</fp>
> <pr>GET /index.html HTTP/1.1</pr>
> <pr>Host: www.megacorpone.com</pr>
> User-Agent: curl/7.74.0
> Accept: */*
. . .
```
> Listing {#l:dirtrav-endpoints} - Example Using _Curl_ 

This will ensure that the web browser is removed from the equation if
traversal is being re-routed back to the web-root. Instead of having
a standard traversal string of "<pr>../</pr>" we would URL encode this value to
be "<pr>..%2F</pr>". 

The resulting GET request in _Burp Suite's Repeater_ would become
such as the below:

<!-- Replace “such as the below” with a better transition. Maybe use “the following”. //-->

```
GET /files/..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd HTTP/1.0 
```
> Listing {#l:dirtrav-sampleGet} - Sample URL Encoded GET Request.

The sample GET request above might be more successful when
testing for _Directory Traversal_ if the target web server is
utilizing programmatic endpoints. Using _Burp Suite's Repeater_,
we have effectively bypassed any intrusion into our attack,
and sending the request with URL encoding makes for very
thorough traversal checks on a programmatic endpoint such as
<fp>https://megacorpone.com/files/../../../../etc/passwd</fp>, where
anything past <fp>/files/</fp> is treated as a parameter.

<!-- Line 110: Attack should be the end of the sentence. Add a period at the end of attack and start a new sentence. //-->

Now that we've covered some theory on _Directory Traversal_ and
traversal strings, let's move on to explore a topic known as
**Suggestive Parameters**.

<!-- Line 118: Replace “we’ve” with “we have”. Line 120: Not sure parenthesis are needed over suggestive parameters. I highlighted in bold using Markdown language. It might be a good idea to eliminate all parenthesis where they are not needed throughout the document and replace with bold or italics. .//-->

## Understanding Suggestive Parameters

At various points in this module we will refer to "Suggestive
Parameters". A **suggestive parameter** is one that gives away what it's
doing or the types of data values it works with. Let's explore some
examples which may be considered a suggestive parameter.

<!-- Line 127: Punctuation error after the first usage of "Suggestive Perameters." The period needs to be before the end of the quotations. Also not sure it has to be in quotations, I made bold in markdown.//-->

Consider the following GET request:

```
GET /search/Hello%20World! HTTP/1.1
```
> Listing {#l:dirtrav-suggestive-sample-get-1} - Sample Search Request

We would consider the above request suggestive in nature becauase
it seems the search GET parameter will perform a search for anything
after <fp>/search/</fp>. Thus, anything we type into the URL Address Bar
after <fp>/search/</fp> will query a database management system (DBMS) or
flat-file system of some kind to retrieve search results.

<!--Line 140: Misspell: Because. //-->

Let's review another suggestive parameter example. Let's imagine
an administrative interface allowing us to click on files in the
corresponding directory to retrieve them.

<!--Line 149: You could either add the word “is” after “interface” or put a coma there to add a pause. //-->

Suppose the below request:
<!-- Line 154: Wrong word usage. Change "Suppose" to "Consider" //-->
```
GET /admin/dashboard/manage/handler.aspx?file=ourFile.jpeg HTTP/1.1
```
> Listing{#l:dirtrav-suggestive-sample-get-1} - Sample File Retrieval Request

In the sample request above, we'll notice yet another suggestive
parameter, file. This parameter name suggests that whatever
happens on the back end, the file parameter will manage file
handling or data handling for files to some extent. These types of
parameters (files and locations) are even more suggestive in-nature
and could be examined further for poor input sanitization.

<!-- Line 161: Eliminate the word “yet” as it is not needed. Line 162: Eliminate the coma after parameter. //-->

Let's examine a few additional suggestive parameters for a deeper
understanding:

```
?file=
?f=
/file/someFile

?location=
?l=
/location/someLocation

search=
s=
/search/someSearch

?data=
?d=
/data/someData

?download=
?d=
/download/someFileData
```
> Listing {#l:dirtrav-suggestive-samples} - Sample Suggestive Parameters

These are only a few examples to shed some light on what we consider a
suggestive parameter to be. In this module, we will be dealing with
parameters that are very suggestive in nature. Since the Directory
Traversal vulnerability deals with files, any suggestive parameter
that might read files from the local file system would be a great
target for Directory Traversal vulnerabilities.

Situations may arise where there are programmatic endpoints during a
web application assessment. For example, suppose we encountered the
URL/endpoint <fp>https://megacorpone.com/files</fp>, in which files is a
programmatic endpoint of a web stack and a user's session object which
developers will often intercept to provide handling of messages
or routing.[^ProgEnd] Athough this endpoint may still be still
susceptible to _Directory Traversal_, conventional traversal attacks
might not work and would instead re-route us to a separate endpoint.

<!-- Line 203: Replace “where there are” with “when there are”. Line 208: The word "still" is used twice. Eliminate the second "still" //-->

In the next section, we will discuss some theory on **Absolute Pathing**
and **Relative Pathing**.

## Relative vs. Absolute Pathing

Before exploring a case study, let's briefly examine the difference
between a relative path versus an absolute path.

We'll start by discussing absolute paths, then use our Kali Linux VM
and the <fp>/etc/</fp> directory to demonstrate commands that utilize
absolute paths. We'll then move onto exploring relative paths.

### Absolute Pathing

To discuss Absolute Pathing, we'll use the sample directory of <fp>/etc/</fp>
as our working directory in our Kali Lab VM. We will execute a command
from this working directory that utilizes absolute pathing.

```
kali@kali:~$ cd /etc/
                                                  
kali@kali:/etc$ pwd
/etc
                                           
kali@kali:/etc$ cat /etc/group
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mail:x:8:
news:x:9:
uucp:x:10:
man:x:12:
proxy:x:13:
kmem:x:15:
dialout:x:20:kali,root
fax:x:21:
voice:x:22:
cdrom:x:24:kali
floppy:x:25:kali
tape:x:26:
sudo:x:27:kali
audio:x:29:pulse,kali
dip:x:30:kali
www-data:x:33:
backup:x:34:
. . . 
```
> Listing {#l:dirtrav-sampleAbsolutePath} - Example Command Utilizing Absolute Pathing
<!-- Possible Listing formatting error? {#1:dirtrav-sample-Absolute Path} /-->

Notice in the above command we typed "<pr>cat</pr> <fp>/etc/group</fp>". By
specifying a forward slash "/" at the beginning of our statement, we
indicate to Linux that we're interested in the "<fp>etc</fp>" directory from
a file system perspective. This is known as Absolute Pathing since,
depending on the operating system, we specify a path that includes a
volume label, begins at the underlying root file system (or "rootfs"
for short) and concludes some number of directories down the line,
concluding with a file.

<!-- Line 271: The / is missing from etc. In this section, the Sentences feel a little too lengthy as if they continue for the entire paragraph. line 272: Maybe find a way to improve the flow of the paragraph by adding a period after Absolute Pathing and starting a new sentence. //-->

The distinct difference between specifying Absolute Pathing versus
Relative Pathing will become more clear when we contrast with a
relative command executed on the system, still from the <fp>/etc/</fp>
directory in the next section.

<!-- Line 282: Eliminate the word “still”. No coma needed after system. //-->

#### Exercise 

1. On your Kali Linux VM, perform the <pr>cat</pr> command on any file of your
choosing in <fp>/etc/</fp> and take a screenshot doing this from
<fp>/var/www/html<fp> with Absolute Pathing.

### Relative Pathing

When discussing **Relative Pathing**, we can always expect to encounter
a reference URI or directory. This reference URI can be any file
path on the target web server. However, when specifying the path,
rather than starting in the <fp>rootfs</fp> (root file system) directory and
moving all the way to some file, our query is much shorter as we
already exist in that relative directory.

<!-- Line 299: eplace “some file” with “a file”. //-->

For example, consider our Kali Lab Virtual Machine. If we traverse
into our home directory <fp>/home/kali/</fp>, we can traverse
outside of that relative directory to view a file or perform an
action by utilizing the "<pr>../</pr>" traversal string. This becomes apparent
below in which we <pr>cat<pr> the <fp>/etc/group</fp> file from within our
<fp>/home/kali/</fp> directory.

<!-- Replace "in which" with "when." //-->

For now, let's examine the command executed below from the
<fp>/etc/</fp> directory on our Kali Linux VM:

```
kali@kali:/etc$ pwd
/etc
                                    
kali@kali:/etc$ cat group     
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mail:x:8:
news:x:9:
uucp:x:10:
man:x:12:
proxy:x:13:
kmem:x:15:
dialout:x:20:kali,root
fax:x:21:
voice:x:22:
cdrom:x:24:kali
floppy:x:25:kali
tape:x:26:
sudo:x:27:kali
audio:x:29:pulse,kali
dip:x:30:kali
www-data:x:33:
backup:x:34:
```
> Listing {#l:dirtrav-relative-group} - passwd File Read From /etc/ 

<!-- Line 347: Does "Read" need to be capitalized? //-->

With our relative pathing example, we <pr>cat</pr> the group file
inside the <fp>/etc/</fp> directory. Notice the key difference that
we did not instruct the linux kernel to <pr>cat</pr> <fp>/etc/group</fp>
directly as an absolute path. Rather, our relative directory and
working directory is <fp>/etc/</fp>, so we only neded to <pr>cat</pr> the
group file. This example merely demonstrates Absolute and
Relative Pathing.

<!-- Line 352,Capitalize "Linux". Line 354, Misspell: "Needed". //-->

Let's now expand our understanding of these concepts to include
the idea that it does sometimes matter how far away we are from
the root <fp>filesystem</fp>. We can use either <fp>../../../etc/group</fp>
to read the file but in some cases we might have to use
<fp>../../../../../../../../../../../../etc/group</fp> depending on
how far away the file is from the root file system or "<fp>rootfs</fp>".
This can be the <fp>/ directory</fp> on Unix based systems or the

<!-- Line 362. Does filesystem need a space between file and system or is it one word? //-->

C:\\ directory on a Windows machine.

These concepts will be foundational to growing our _Directory Traversal_
skills. Before moving onto that topic, We'll show few contrasting
examples of absolute and relative pathing. This time however, instead
of being on our Kali Linux VM, we will be using a fictitious endpoint.

<!-- Line 373, add "a" before the word "few."//-->

Suppose we encounter the following URL:
<fp>https://megacorpone.com/index.php?image=someImage.jpg</fp>

It seems like relative pathing is being used in the above URL because
there is no file path in the parameter. The application could be
appending the "image" value to a hard-coded path.

In other words, this application could be doing something such as:
````
$file = $image_dir + $_GET[image]
````
>Listing {#l:dirtrav-relative-absolute - Sample Code Showing Image Directory and Image Name being Concactenated

<!-- Line 390.Possible listing formatting error, missing the "}". Misspell: Concatenated. //-->

With that in mind, let's pay close attention to our fictitious
image parameter in the above URL with the value of
<pr>someImage.jpg</pr>. We'll notice that relative pathing is being utilized. If
absolute pathing were being utilized, we would find that our sample
endpoint would appear very much different!

<!-- Line 399. Eliminate the word “much”. //-->

Now with absolute pathing:
<fp>https://megacorpone.com/index.php?image=/var/www/html/pub/images/someImage.jpg</fp>

However, from a programmatic standpoint as discussed above, it could
be that the image parameter is performing concatenation (the joining
together of) the base image path and the image file itself. This would
be an example of hard-coded absolute pathing.

These sample links demonstrate the key difference between a relative
path and an absolute path in web applications. We will be working with
both in the sections to come as we grow our understanding of _Directory
Traversal_ attacks.

#### Exercise

1. On your Kali Linux VM, perform the <pr>cat</pr> command on any file
in <fp>/etc/</fp> and take a screenshot doing this from the
<fp>/etc/</fp> directory with Relative Pathing.
