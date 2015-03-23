Install ruby + cucumber on Windows


Install ruby for windows using the [RubyInstaller for Windows]
(http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.1.5-x64.exe?direct)
(Note that as of writing, 2.2.0 didn''t work with cucumber so if you want to play safe go with 2.1.0)

Copy this [file]
(https://raw.githubusercontent.com/rubygems/rubygems/master/lib/rubygems/ssl_certs/AddTrustExternalCARoot-2048.pem) to `%RUBY_HOME%\lib\ruby\2.1.0\rubygems\ssl_certs`

Install DevKit from [RubyInstaller for Windows](http://dl.bintray.com/oneclick/rubyinstaller/DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe?direct)

Then open a new cmd (or other console emulator) and test out the install

```shell
C:\> ruby -v
ruby 2.1.5p273 (2014-11-13 revision 48405) [x64-mingw32]

C:\> gem install cucumber
Fetching: builder-3.2.2.gem (100%)
Successfully installed builder-3.2.2
Fetching: diff-lcs-1.2.5.gem (100%)
Successfully installed diff-lcs-1.2.5
Fetching: multi_json-1.11.0.gem (100%)
...
Parsing documentation for multi_test-0.1.2
Installing ri documentation for multi_test-0.1.2
Done installing documentation for builder, cucumber, diff-lcs, gherkin, multi_js
on, multi_test after 34 seconds
6 gems installed
```

When running cucumber, you may see a warning about ANSICON: ```** WARNING: You must use ANSICON 1.31 or higher 
(http://adoxa.110mb.com/ansicon) to get coloured output on Windows```

If you are running in vanilla CMD.exe, install ANSICON to see colorized output. If you are running in a 
modern console that already supports ANSI, you can supress this warning by adding an environment variable `ANSICON=1`.

Happy cuking!

