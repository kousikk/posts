<!--
3
kousikk
01 Dec
2013
flask, mod_wsgi, apache, setup, tech
-->

After spending nearly four hours trying to setup Flask with Apache, I feel confident enough to write this post.<br>
So the first question,

Why?
----
Because, the as it is so astutely demonstrated in the [Flask Docs](http://flask.pocoo.org/docs/deploying/#deployment-options), the web server included with Flask is minimal and should not be used in production.<br>

What?
-----
So what Web-server am I going to deploy my application with? I prefer Apache because the process is well documented(/me disagrees though).<br>

How?
----
Ah, here comes the biggest trouble of your life. I will see if I can summarize the process in *7 steps.* [Look Here] [1].<br><br>
\*&nbsp;&nbsp;&nbsp;You need apache. This can be done with, `sudo apt-get install apache2.`<br><br>
\*&nbsp;&nbsp;&nbsp;You need mod_wsgi. I know mod stands for module(something that is plugged into Apache to provide additional functionality). But what in the name of God is wsgi? Its nothing but an interface, that helps our Apache Web-server(which can't execute python code), to send my HTTP request to the python Web-application and send it away back to the server, so that the server can serve it to the user. To install mod_wsgi, `sudo apt-get install libapache2-mod-wsgi`.<br><br>
\*&nbsp;&nbsp;&nbsp;You need the [Flask application code](https://github.com/programmerlicense/myblog). You can do a `git-clone`. Save it in a directory, say your home directory.<br><br>
\*&nbsp;&nbsp;&nbsp;Now, you need to setup apache in such a way, so that it knows, when a particular URL is given, it has to execute this python application and serve back the results. This is done with WSGI Script Alias. This specifies a mapping from your URL to the python application to execute. All WSGI applications are nothing but Python programs, that are executed. Catch is, the WSGI application contained within the code file specified should be called 'application'. Just to make things cleaner and seperate our Flask application from our WSGI application that will be executed, we will create one more file called, router.wsgi, in your home directory with the following content:<br><br>
`import sys, os`<br>
`os.environ['HOME'] = '/home/plicense' #This line can be ignored`<br>
`sys.path.insert(0,'/home/plicense/public_html/apps/myblog')`<br>
`from router import app as application`<br>
`application.debug = True`<br><br><Br>
\*&nbsp;&nbsp;&nbsp;The WSGI part of setting up apache is over. Lets come to the Apache part of setting up apache. Again, to clean things up, lets create a new Virtual Host. Create a file called `blog` in your `/etc/apache2/sites-available` directory, with the following content:<Br><br>
`<VirtualHost \*:80>`  
        `# ---- Configure VirtualHost Defaults ----`  
    `ServerAdmin <kousikkumar771@gmail.com>`   
        `DocumentRoot /home/plicense/public_html/http`   
        `<Directory />`   
                `Options FollowSymLinks`   
                `AllowOverride None`   
        `</Directory>`     
        `<Directory /home/plicense/public_html/http/>`   
                `Options Indexes FollowSymLinks MultiViews`   
                `AllowOverride None`   
                `Order allow,deny`   
                `allow from all`   
        `</Directory>`   
        `# ---- Configure WSGI Listener(s) ----`   
        `WSGIDaemonProcess flaskapp user=plicense group=plicense threads=5`   
        `WSGIScriptAlias /blog /home/plicense/public_html/wsgi/router.wsgi`   
        `<Directory /home/plicense/public_html/http/flasktest1>`   
                `WSGIProcessGroup flaskapp`   
                `WSGIApplicationGroup %{GLOBAL}`   
                `Order deny,allow`   
                `Allow from all`   
        `</Directory>`   
        `# ---- Configure Logging ----`  
    `ErrorLog /home/plicense/public_html/logs/error.log`   
    `LogLevel debug`   
    `CustomLog /home/plicense/public_html/logs/access.log combined`   
`</VirtualHost>`  


For this virtual host to work, you have to have a directory hierarchy like this:   
/home/plicense/  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|<Br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-\-\-\->public_html<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-\-\-\-\->apps&nbsp;#Contains&nbsp;your&nbsp;python&nbsp;application<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-\-\-\-\->logs&nbsp;#For&nbsp;storing&nbsp;<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-\-\-\-\->wsgi&nbsp;#Contains&nbsp;your&nbsp;wsgi&nbsp;file<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-\-\-\-\->http&nbsp;#Contains&nbsp;default&nbsp;html&nbsp;files<br>
\*&nbsp;&nbsp;&nbsp;Phew that's all. Now you need to enable your site. This can be done with [`a2ensite blog`] [2].<Br><br>
\*&nbsp;&nbsp;&nbsp;Now restart Apache with `sudo service apache2 restart`.<br><br>

And Tada! `curl -O http://localhost/blog` should give you the output of the function in your application that over-rides the `app.route('/')`.  

[1]: http://ubuntu.com/ "Your operating system must be Ubuntu 12.04."  
[2]: http://manpages.ubuntu.com/manpages/precise/man8/a2ensite.8.html "To disable your site, use `a2dissite blog`."  
