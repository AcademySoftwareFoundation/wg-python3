# Autodesk DCC Support for Python 3
Autodesk is actively working to support Python 3 in its entertainment DCC products. 

3ds Max 2021, which shipped in April 2020, already includes support for Python 3. 

Work to support Python 3 in Maya and MotionBuilder is in active development. Previews of Maya and MotionBuilder supporting Python 3 have been available since late 2019 on the [Autodesk Beta Forum](https://beta.autodesk.com). If you're interesting in providing feedback, please contact us at me.beta.maya@autodesk.com.

Since we realize many studios will required some time to transition to Python 3, 3ds Max 2021 and the upcoming versions of Maya and MotionBuilder after 2020 will support _both_ Python 3 (the new default) and Python 2.

When starting 3ds Max 2021, the next major version of Maya or the next major version of MotionBuiler (after 2020), users can select the version of Python they'd like to use via a “startup switch”. The selected version of Python will then remain fix throughout the session. Users can select a different version of the Python by restarting the application. 

For the next major versions of Maya and MotionBuilder, the startup switch will be available on Windows and Linux. The macOS version of the next major version of Maya will be Python 3 only.

## What version(s) will support both Python 2/3 concurrently?

| Product            | Python 2 Version       | Python 3 Version  |
|--------------------|------------------------|-------------------|
| [3ds Max 2021.x](https://help.autodesk.com/view/MAXDEV/2021/ENU/?guid=Max_Python_API_what_s_new_in_3ds_max_python_api_html)       | 2.7.17                 | 3.7.6             |
| Maya Next          | 2.7.11 (Windows/Linux) | 3.7.8             |
| MotionBuilder Next | 2.7.11                 | 3.7.8             |

## What will be the last version to support only Python 2?
Support for Python 2 will be removed in the next major version of 3ds Max.

For Maya and MotionBuilder, Python 2 will be removed after the next major version. After this, only Python 3 will be supported.
