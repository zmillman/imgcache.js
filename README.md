imgcache.js
===========

Description
-----------
The purpose of this JS library is to cache images using the new [html5 File API](https://developer.mozilla.org/en-US/docs/Using_files_from_web_applications). If the page is viewed offline and its images are cached through this mechanism, they will be shown instead of an empty image.

This library is especially useful for mobile web applications using PhoneGap/Cordova where the normal browser cache cannot be relied upon and where offline navigation is quite common.

Used with imagesloaded as shown in examples/example2.html, you can see that it can automatically:
* store images in cache
* replace images with cached version if they fail to load (offline / busy server..)
This is the best solution I have found so far to provide easy caching of images within a PhoneGap web app.

This library works with PhoneGap/Cordova 1.7 so the supported platforms should be:
* Android [TESTED]
* iOS [TESTED]
* BlackBerry WebWorks (OS 5.0 and higher)
* Windows Phone 7 ( Mango )

It has also been made to work with the original html5 File API - currently only implemented in recent Chrome versions - to speed up testing of a whole web application from the desktop.

All methods are asynchronous : use callbacks if required.

Using imgcache.js
=================

Requirements
------------
* jQuery (any version from 1.6 should do) or Zepto
* PhoneGap/Cordova (v >= 1.7) *optional*
* [imagesloaded] (http://desandro.github.com/imagesloaded/) *optional*

Installation
----------
To use `imgcache.js`, you need to copy `js/imgcache.js` and add it to your
application's Javascript. You can then load it like so:

    <script src="imgcache.js" type="application/javascript"></script>
    
Using with PhoneGap/Cordova:
* Requires the File API permission in `config.xml`: `<feature name="http://api.phonegap.com/1.0/file"/>`
* Remember to allow access to remote files by adding your domain in config.xml - or all domains using a wildcard: `<access origin="*" />`
    
Using with Chrome (v > 12 I believe...) or future browsers that support the [html5 filesystem API](http://caniuse.com/filesystem):
* Beware of cross domain ajax issue! retrieve image from the same domain or set CORS solutions with the server...
* If page opened locally (file:// ..), chrome needs to be loaded with the following flags: `--allow-file-access-from-files --allow-file-access` otherwise the local filesystem will not be accessible (security error)
* To navigate through the local filesystem open a new tab with filesystem:http://*yourSiteDomain*/persistent/
    
Setup your cache
----------------
Before initializing the cache, you must specify any default options you wish to configure:

    // write log to console
    ImgCache.options.debug = true;
    
    // increase allocated space on Chrome to 50MB, default was 10MB
    ImgCache.options.chromeQuota = 50*1024*1024;
    
See `ImgCache.options` at the top of the source file for more settings.
    
After setting any custom configuration, initialize the cache:

    ImgCache.init(function(){
      alert('cache created successfully!');
    }, function(){
      alert('check the log for errors');
    });
    
If you're using imgcache.js with PhoneGap/Cordova, `ImgCache.init()` must be called in `onDeviceReady`, not before!

Note that in Chrome, the user will be prompted to give permission to the page for accessing the local filesystem (which will run the error callback if they decline).

Storing images
--------------
Images are stored into the local folder specified by `ImgCache.options.localCacheFolder`. To add a file to the cache:

    ImgCache.cacheFile('my-cdn.com/users/2/profile.jpg');

TODO: add documentation for `ImgCache.cacheBackground`

Using cached images
-------------------
Once an image is stored in the cache, you can replace the file displayed by the image tag:

    target = $('img#profile');
    ImgCache.cacheFile(target.attr('src'), function(){
      ImgCache.useCachedFile(target, function(){
        alert('now using local copy');
      }, function(){
        alert('could not load from cache');
      })
    });
    
To check if a file is stored locally:

    ImgCache.isCached(target.attr('src'), function(path, success){
      if(success){
        // already cached
        ImgCache.useCachedFile(target);
      } else {
        // not there, need to cache the image
        ImgCache.cacheFile(target.attr('src'), function(){
          ImgCache.useCachedFile(target);
        });
      }
    });
    
When you no longer want to use the locally cached file:

    target = $('img#profile');
    ImgCache.useOnlineFile(target);

Clearing the cache
------------------
To remove all cached files, clear the local cache folder:

    ImgCache.clearCache(function(){
      // continue cleanup...
    }, function(){
      // something went wrong
    });
    
There is currently no way to invalidate single images from the cache.

Code samples
------------
See html files in the `examples/` folder.

Release Notes
-------------
* v0.5.2 Fixed isCache for Android  + now works with Zepto + using new Chrome Blob
* v0.5.1 Fixed behaviour of isCached method
* v0.5 Added isCached method (thanks to David Novakovic)
* v0.4 Set cache files to not be backed up by iCloud (iOS only - requires Cordova 1.8+)
* v0.3 Added granularity to log entries + callbacks to all asynchronous methods + automated tests page
* v0.2 Cached filenames are now using hash of source url (SHA-1)
* v0.1 Initial release

License
-------
Copyright 2012-2013 (c) Christophe BENOIT

Apache License - see LICENSE.md

Code from http://code.google.com/p/tiny-sha1/ is being used which is under the MIT License.
The copyright for this part belongs to the creator of this work.

Todo
----
* Find a solution for cache invalidation in case an image changes
* When Chrome finally supports canvas.toBlob(), possibly replace download method with new one that draws an Image into a canvas and then retrieves its content using the toBlob() method -- or use [canvas-toBlob.js] (https://github.com/eligrey/canvas-toBlob.js)
