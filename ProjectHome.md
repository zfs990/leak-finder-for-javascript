<h1>Note: jsleakcheck is no longer supported! It was working against an unofficial and unstable Dev Tools protocol for taking heap snapshots. The protocol is being worked on, and it is not stable enough so that I could keep jsleakcheck working with various Chrome versions. In addition, a lower level compatibility tool, remote_inspector_client.py, which jsleakcheck was using to communicate with Dev Tools, got removed.</h1>



---


In JavaScript you cannot have "memory leaks" in the traditional sense, but you can have objects which are unintentionally kept alive and which in turn keep alive other objects, e.g., large parts of DOM.

Leak Finder for JavaScript works against the Developer tools remote inspecting protocol of Chrome, retrieves heap snapshots, and detects objects which are "memory leaks" according to a given leak definition.

The default configuration of the tool detects goog.Disposable objects which were not dispose()d. goog.EventTarget is a subclass of goog.Disposable, and if an EventTarget is not disposed, event listeners are not discarded properly, and event listeners in turn keep DOM objects alive. It's possible to configure the tool to detect other similar misuses.

Resources:

  * [Blog post](http://google-opensource.blogspot.com/2012/08/leak-finder-new-tool-for-javascript.html) including a screenshot demonstrating the usage.
  * [Slide deck](https://leak-finder-for-javascript.googlecode.com/files/Open%20-%20Finding%20memory%20leaks%20in%20JavaScript%20programs.pdf)


---


Getting the code:

1. Install depot\_tools (see http://dev.chromium.org/developers/how-tos/install-depot-tools )

2. Create a directory for your local repository, and put the following text in a file called .gclient :
```
solutions = [
  { "name"        : "leak-finder",
    "url"         : "git+https://code.google.com/p/leak-finder-for-javascript",
  },
]
```
3. gclient sync


---


Running the code:

Requirements:
  * simplejson (For Ubuntu, the package is python-simplejson, for Mac see http://slackrw.wordpress.com/2010/04/04/simplejson-for-python-on-mac-os-x/ )

Prerequisites:
  * Your web app uses unobfuscated JavaScript.
  * Your web app uses a recent enough Closure version (fetched after Aug 29th 2012).
  * Google Chrome version >= 26.

1. Start Google Chrome with following flags:
```
--remote-debugging-port=9222 --js-flags=--stack_trace_limit=-1 --user-data-dir=some/empty/directory
```
E.g., on Linux:
```
google-chrome --remote-debugging-port=9222 --js-flags=--stack_trace_limit=-1 --user-data-dir=/tmp/jsleakcheck
```
And on Mac OS Lion:
```
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 --js-flags=--stack_trace_limit=-1 --user-data-dir=/tmp/jsleakcheck
```

2. Use the first tab to run your web application. There is a test page in $repo/leak-finder/doc/test-page.html that you can use for trying out jsleakcheck.

Then set goog.Disposable.MONITORING\_MODE=goog.Disposable.MonitoringMode.INTERACTIVE in your web application as early as possible. You can also inject it with Dev Tools (press Ctrl + Shift + i to bring it up).

3. In $your\_repo/leak\_finder/src/, run e.g.,
```
python jsleakcheck.py -d closure-disposable -v
```

4. To display the configuration options, run
```
python jsleakcheck.py -h
```