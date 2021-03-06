These instructions apply only to the "Shareabouts Web" application.
If you are also building and installing the Shareabouts API yourself,
it has its own documentation.

You can of course deploy to any server that supports Django.

Deploying to a PaaS provider
----------------------------

At OpenPlans, we have been deploying Shareabouts to DotCloud internally, so many
of the files necessary are already in the repository.  We also have the files
necessary for deploying to Heroku.  Other PaaS providers should be simple
variations on these.

* Create a new application:

  *DotCloud*

      dotcloud create <instance name>

  *Heroku*

      heroku apps:create <instance name>

* Push to the application

  *DotCloud*

      dotcloud push <instance name> -b master

  Note you should either push all your changes to your master
  repository (eg. github or whatever you're using for version
  control);  otherwise you must use the dotcloud push --all option.

  For more options, see `dotcloud push --help`

  *Heroku*

      git push heroku master:master

* Set your flavor, and dataset API key and root URL:
  
  You will need your dataset root API URL for this step.  Suppose you are using an API server hames *api.shareabouts.org* with a username *mjumbewu* and a dataset called *niceplaces*. In this case, your dataset root will he `http://api.shareabouts.org/api/v1/datasets/mjumbewu/niceplaces/`.  In general, it will always be `http://<api server>/api/v1/datasets/<username>/<dataset slug>/`.

  *DotCloud*

         dotcloud var set <instance name> SHAREABOUTS_FLAVOR=<flavor name> \
                                          SHAREABOUTS_DATASET_ROOT=<dataset root url> \
                                          SHAREABOUTS_DATASET_KEY=<dataset api key>
	                                     
  *Heroku*
  
         heroku config:set SHAREABOUTS_FLAVOR=<flavor name> \
                           SHAREABOUTS_DATASET_ROOT=<dataset root url> \
                           SHAREABOUTS_DATASET_KEY=<dataset api key>

Should be all done!


Deploying to WebFaction
-----------------------

0. Create a Django application using Django 1.4 and Python 2.7 in the control panel (Applications -> Add New Application). In this case an application called 'shareabouts_front' was created.

1. SSH into your server and check out the project alongside wherever WebFaction created your `myproject` app:

        cd webapps/shareabouts_front
        git clone git://github.com/openplans/shareabouts.git

2. Install pip and (optionally) virtualenv:

        easy_install-2.7 pip
        easy_install-2.7 virtualenv

3. Create a virtual environment.  This is not technically necessary, but is recommended if you have any other non-Shareabouts Python applications running on your server, or if you plan to in the future.  For a brief introduction to virtual environments in Python see http://iamzed.com/2009/05/07/a-primer-on-virtualenv/ or the virtualenv docs at http://www.virtualenv.org/:

        virtualenv venv --no-site-packages
        source venv/bin/activate

4. Install the project dependencies:

        pip install -r shareabouts/requirements.txt

5. Edit the apache2/conf/http.conf file. This is so that Apache knows where to find the project's dependencies, and how to run the WSGI app:

        ...
        SetEnvIf X-Forwarded-SSL on HTTPS=1
        ThreadsPerChild 5

        #####
        # THIS LINE WAS CHANGED to refer to your Shareabouts project path
        WSGIDaemonProcess shareabouts_front processes=2 threads=12 python-path=/home/<HOME_DIR>/webapps/shareabouts_front:/home/<HOME_DIR>/webapps/shareabouts_front/shareabouts/src:/home/<HOME_DIR>/webapps/shareabouts_front/lib/python2.7

        WSGIProcessGroup shareabouts_front
        WSGIRestrictEmbedded On
        WSGILazyInitialization On

        #####
        # AND THIS LINE WAS CHANGED to refer to your Shareabouts WSGI module
        WSGIScriptAlias / /home/<HOME_DIR>/webapps/shareabouts_front/shareabouts/src/project/wsgi.py


6. Edit the WSGI module (shareabouts/src/project/wsgi.py).  This is so that the project runs in the same environment where all of its dependencies have been installed  *If you did not set up a virtual environment, you can skip this step*:

   After...

        os.environ.setdefault("DJANGO_SETTINGS_MODULE", "project.settings")

   Add...

        activate_this = os.path.expanduser("~/webapps/shareabouts_front/venv/bin/activate_this.py")
        execfile(activate_this, dict(__file__=activate_this))


7. Update the shareabouts/src/project/urls.py file to be able to find the site's static assets.  **NOTE: it would be better if this pointed to an actual static file server.  See http://docs.webfaction.com/software/django/config.html#serving-django-static-media for more information**:

   At the top, add...

        from django.contrib.staticfiles.urls import staticfiles_urlpatterns

   And change

        urlpatterns = patterns('',

   to

        urlpatterns = staticfiles_urlpatterns() + patterns('',
