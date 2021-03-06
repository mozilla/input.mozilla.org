Firefox Input
=============

Firefox Input is a [Django][Django]-based web application to gather user
feedback from the [Mozilla][Mozilla] Firefox beta testing program.

For project goals, specifications, etc., check out the
[Input Wiki Page][wikimo].

[Mozilla]: http://www.mozilla.org
[Django]: http://www.djangoproject.com/
[wikimo]: https://wiki.mozilla.org/Firefox/Input


Getting Started
---------------

### Git

Get the code:

    git clone git@github.com:mozilla/input.mozilla.org.git input-site

Make sure that the directory it creates is not input.mozilla.org because
manage.py adds the top-level directory to the PYTHONPATH and then it
thinks that input.mozilla.org is a Python module and then it all breaks.


### Python

You need Python 2.6.

Also, you probably want to run this application in a
[virtualenv][virtualenv] environment.

Run:

    easy_install pip

followed by:

    pip install -r requirements/prod.txt -r requirements/compiled.txt

to install the required Python libraries.

[virtualenv]: http://pypi.python.org/pypi/virtualenv


### Django

Put your database settings in `settings_local.py`:

        from settings import *

        # ...

        DATABASES = {
            'default': {
                'ENGINE': 'mysql',
                'NAME': 'firefox_input',
                'USER': 'input',
                'PASSWORD': 'input',
                'HOST': 'localhost',
                'PORT': '',
                'OPTIONS': {'init_command': 'SET storage_engine=InnoDB',
                            'charset' : 'utf8',
                            'use_unicode' : True,
                           },
            },
            'website_issues': {
                'ENGINE': 'mysql',
                # ...
            }
        }

Make sure you create the database and user you've specified:

    $ mysql -u root -p
    mysql> CREATE DATABASE firefox_input;
    mysql> GRANT ALL ON firefox_input.* TO input@localhost IDENTIFIED BY 'input';
    mysql> GRANT ALL ON test_firefox_input.* TO input@localhost IDENTIFIED BY 'input';
    mysql> CREATE DATABASE website_issues;
    mysql> GRANT ALL ON website_issues.* TO input@localhost IDENTIFIED BY 'input';
    mysql> GRANT ALL ON test_website_issues.* TO input@localhost IDENTIFIED BY 'input';

To initialize the database, run:

    $ ./manage.py syncdb

To update your schemas from an earlier version:

    $ DJANGO_SETTINGS_MODULE=settings ./vendor/src/schematic/schematic migrations

To initialize the search index, run:

    $ ./manage.py update_index

The Internet has plenty of of documentation on setting up a Django application
with any web server. If you need a wsgi entry point, you can find one in
``wsgi/reporter.wsgi``.


### Highcharts

Due to licensing issues, we can't rightfully bundle highcharts.js.  So you can
go get it using:

    $ ./manage.py cron get_highcharts


### Sphinx

For searching, we use [Sphinx][sphinx]. Set up an instance of it, and adjust
the SPHINX\_\* settings in settings.py to match your setup.

See docs/sphinxsearch.rst for more details.

[sphinx]: http://www.sphinxsearch.com/


### Sites data

The "website\_issues" database has to be a separate SQL-database from
"default", as it uses its own schematic migration version.
It is used to load aggregate website issues (generated by clustering) from a
cron task. To initialize it, run:

    $ ./manage.py syncdb --database=website_issues

To generate site data yourself without getting it pushed from metrics, run:

    $ ./manage.py generate_sites

And for schema updates:

    $ DJANGO_SETTINGS_MODULE=settings ./vendor/src/schematic/schematic migrations/sites


### Cron jobs

There are two jobs you may want to run periodically:

    $ ./manage.py update_product_details  # Mozilla Product Details update
    $ ./manage.py update_index -r         # update and rotate search index

The frequency is up to you, but you probably want to run the search index
updates relatively frequently, while the product details can wait a little
longer.

Note that updating the product details files like this will lead to "local
changes" in your checkout. If you plan on pulling code updates from git
periodically, you should leave ``lib/product_details_json`` untouched, but
create a new directory somewhere else and change the setting
``PROD_DETAILS_DIR`` accordingly.


### Mobile vs. Desktop site

We are using the [Django Sites Framework][sites] to distinguish between the
mobile site and the desktop site. The default is site ID 1 == desktop. If
you create another site using the admin interface, requests for that site's
domain will show the mobile site (set ``settings.MOBILE_SITE_ID`` accordingly,
though the default of 2 is probably correct).

For development, you can create an alias of localhost (``m.localhost``, for
example) in ``/etc/hosts``, and use that as the domain for the second site.
Make sure to include the port (``m.localhost:8000``).

[sites]: http://docs.djangoproject.com/en/dev/ref/contrib/sites/


License
-------

This software is licensed under the [Mozilla Tri-License][MPL]:

    ***** BEGIN LICENSE BLOCK *****
    Version: MPL 1.1/GPL 2.0/LGPL 2.1

    The contents of this file are subject to the Mozilla Public License Version
    1.1 (the "License"); you may not use this file except in compliance with
    the License. You may obtain a copy of the License at
    http://www.mozilla.org/MPL/

    Software distributed under the License is distributed on an "AS IS" basis,
    WITHOUT WARRANTY OF ANY KIND, either express or implied. See the License
    for the specific language governing rights and limitations under the
    License.

    The Original Code is Firefox Input.

    The Initial Developer of the Original Code is Mozilla.
    Portions created by the Initial Developer are Copyright (C) 2010
    the Initial Developer. All Rights Reserved.

    Contributor(s):
      Frederic Wenzel <fwenzel@mozilla.com>

    Alternatively, the contents of this file may be used under the terms of
    either the GNU General Public License Version 2 or later (the "GPL"), or
    the GNU Lesser General Public License Version 2.1 or later (the "LGPL"),
    in which case the provisions of the GPL or the LGPL are applicable instead
    of those above. If you wish to allow use of your version of this file only
    under the terms of either the GPL or the LGPL, and not to allow others to
    use your version of this file under the terms of the MPL, indicate your
    decision by deleting the provisions above and replace them with the notice
    and other provisions required by the GPL or the LGPL. If you do not delete
    the provisions above, a recipient may use your version of this file under
    the terms of any one of the MPL, the GPL or the LGPL.

    ***** END LICENSE BLOCK *****

[MPL]: http://www.mozilla.org/MPL/
