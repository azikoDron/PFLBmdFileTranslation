# Jmeter.Plugin.TailSampler

_Sampler_, performing parallel loading of specified resources.

Plugin makes it easy to load embedded resources, 
making the test as close as possible to the browser operation.
**HTTP Request Tail** converts the list of links into an HTML document, 
GET request for each of these links, downloads embedded resources

[Link][1] to download.

[1]: (https://github.com/pflb/Jmeter.Plugin.TailSampler/blob/master/ru.pflb.jmeter.samplers.TailSampler.jar?raw=true)

This version of the plugin designed to work with JMeter 3.0, JMeter 2.13. 
Not tested on earlier versions.

### Installation

1. Download the plugin (see links above) ru.pflb.jmeter.samplers.TailSampler.jar.
2. Copy the plugin to `lib/ext` directory JMeter 3.0.
3. Restart JMeter 3.0.

Example directory:
```
D:\TOOLS\apache-jmeter-3.0\lib\ext\
D:\TOOLS\apache-jmeter-3.0\lib\ext\ru.pflb.jmeter.samplers.TailSampler.jar
```
### Description
Default settings:

- [v] **Retrieve All Embedded Resources** - *the checkbox is set by default, it can be removed,
but then subrequests will not be executed, and the HTTP Request Tail will become useless*.
- [v] **Use concurrent pool** -  *the checkbox is set by default, 
  on a large number of embedded resources, multi-threaded download increases the download speed*.
- **Use concurrent pool Size**: `4` - *the default value is 4, the value is used as the JMeter base value*.
   - HttpClient4 *when configuring* **Use concurrent pool Size**: 4 will send up to 4 requests parallel,
     each thread will use 1 persistent connection per domai.