**You can download the plugin [here][1]:**

[1]: (https://github.com/pflb/Jmeter.Plugin.CsvLogWriter/blob/master/ru.pflb.jmeter.listener.CsvLogWriter.jar?raw=true)

There is Jmeter-plugins.org, с **jp@gc - Flexible File Writer**:

- standard\src\kg\apc\jmeter\reporters:
  - FlexibleFileWriter.java
  - FlexibleFileWriterGui.java

Tasks - to do:

- display error texts;
- output to csv format;
- rotation.


D:\Project\jmeter-plugins\standard\src\kg\apc\jmeter\reporters\\**FlexibleFileWriter.java**, the key method is here
``public void sampleOccurred(SampleEvent evt)``:

	
	@Override
    public void sampleOccurred(SampleEvent evt) {
        if (fileChannel == null || !fileChannel.isOpen()) {
            if (log.isWarnEnabled()) {
                log.warn("File writer is closed! Maybe test has already been stopped");
            }
            return;
        }

        ByteBuffer buf = ByteBuffer.allocateDirect(writeBufferSize);
        for (int n = 0; n < compiledConsts.length; n++) {
            if (compiledConsts[n] != null) {
                //noinspection SynchronizeOnNonFinalField
                synchronized (compiledConsts) {
                    buf.put(compiledConsts[n].duplicate());
                }
            } else {
                if (!appendSampleResultField(buf, evt.getResult(), compiledFields[n])) {
                    appendSampleVariable(buf, evt, compiledVars[n]);
                }
            }
        }

        buf.flip();

        try {
            syncWrite(buf);
        } catch (IOException ex) {
            log.error("Problems writing to file", ex);
        }
    }


It's convenient to implement subrequest processing in method ``sampleOccurred``.
To get error texts. it's convenient to display first 200-300 response characters in the log, for requests with the 
response code == 500.

Log rotation can be implemented in method ``syncWrite`` by performing rotation by the number of bytes written per file.


**Description of plugin pflb@CsvLogWriter functionality.**

Plugin allows you to record a detailed log file without prior settings. 

Log file records th following data: 
- timeStamp - start of processing, millisecond;  
- elapsed - duration of processing, millisecond;
- label - JMeter component names; 
- responseCode - request response code; 
- responseMessage - Contents of the response to the request;	
- threadName - thread name;	
- dataType - data type;
- success - response status;	
- failureMessage - error message;
- bytes - the amount of server response data;	
- grpThreads - number of active virtual users of the current group;	
- allThreads - total number of active virtual users of all groups; 
- URL - links;	
- Filename - the name of the file where the responses are recorded;	
- Latency - time until the first server response is received;	
- Encoding - encoding;	
- SampleCount - number of samples;	
- ErrorCount - number of errors;	
- Hostname - host name;	
- IdleTime - downtime, millisecond;
- Connect - time spent installing the connection;	
- headersSize - header size;
- bodySize - body size;
- contentType - content type from the answer header;
- endTime - request completion time;
- threadName_label - thread and JMeter component names;
- parent_threadName_label - the names of the parent's JMeter component and thread;
- startTime - the start time of the request;
- stopTest - a sign whether the test is stopped - Stop button;
- stopTestNow - a sign whether the test is stopped - Shutdown button;
- stopThread - a sign whether the current thread has been stopped;
- startNextThreadLoop - will the reply start;
- isTransactionSampleEvent - a sign that the current event is an (TransactionController) transaction;
- transactionLevel - request nesting level;
- responseDataAsString - the full content of the error in the string format in case of its occurrence;
- requestHeaders - request headers;
- responseData - the full content of the error in case of its occurrence;
- responseHeaders - response headers;
- <Имя_переменной> - JMeter variables.



The plugin can record the result of subrequests and record the full error text in case of its occurrence, in plain text
, not XML format.

Example of the plugin's result:
```
    timeStamp;elapsed;label;responseCode;responseMessage;threadName;dataType;success;failureMessage;bytes;grpThreads;allThreads;URL;Filename;Latency;Encoding;SampleCount;ErrorCount;Hostname;IdleTime;Connect;"responseData";"transactionLevel"
    1454498796125;6759;"Transaction Controller1";"200";"Number of samples in transaction : 11, number of failing samples : 0";"Thread Group 1-1";"";true;"";1289;1;1;"null";"";0;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";0
    1454498796125;540;"Transaction Controller 2";"200";"Number of samples in transaction : 1, number of failing samples : 0";"Thread Group 1-1";"";true;"";114;1;1;"null";"";0;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";1
    1454498796126;540;"jp@gc - Dummy Sampler";"200";"OK";"Thread Group 1-1";"text";true;"";114;1;1;"null";"";4;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";2
    1454498796667;667;"Transaction Controller 2";"200";"Number of samples in transaction : 1, number of failing samples : 0";"Thread Group 1-1";"";true;"";114;1;1;"null";"";0;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";1
    1454498796668;667;"jp@gc - Dummy Sampler";"200";"OK";"Thread Group 1-1";"text";true;"";114;1;1;"null";"";9;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";2
    1454498797336;666;"Transaction Controller 2";"200";"Number of samples in transaction : 1, number of failing samples : 0";"Thread Group 1-1";"";true;"";114;1;1;"null";"";0;"ISO-8859-1";1;0;"aperevozchikova";9;0;"";1
    1454498802041;853;"jp@gc - Error Sampler";"500";"Internal Server Error";"Thread Group 1-1";"text";true;"";149;1;1;"null";"";87;"ISO-8859-1";1;0;"aperevozchikova";9;0;"Error text."
```

To start the plugin, you need to fill in the `Filename` field. 
The `Filename` field contains a path to the file where the result will be fixed. You can prescribe the directory manually
or select file path using the `Browse` button.  
In case where the specified file exists, a new file is created with the addition of a postfix. 
There are also checkboxes on the form. The checkboxes can be used to manipulate the data fixed in the log.
`Additional parameters` checkbox is responsible for fixing additional parameters.
`Response` is responsible for fixing the error text, `User variables` is responsible for fixing custom variabless.
