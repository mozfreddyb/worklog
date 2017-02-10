# logging with MOZ_LOG on the try server

There is a test failure on Mac OS X, that I can hardly debug.
As a first step, I'll  this to the try-server with more logging output enabled.

My test is a mochitest, so I modified testing/mochitest/runtests.py:

```
diff --git a/testing/mochitest/runtests.py b/testing/mochitest/runtests.py
index 45545b4..5afdffd 100644
--- a/testing/mochitest/runtests.py
+++ b/testing/mochitest/runtests.py
@@ -91,7 +91,7 @@ here = os.path.abspath(os.path.dirname(__file__))
 # Try run will then put a download link for all log files
 # on tbpl.mozilla.org.

-MOZ_LOG = ""
+MOZ_LOG = "nsDocShellLogger:4,CSPParser:4,CSPUtils:4,CSPContext:4,CSP:4"
```

And now we play the waiting game.
