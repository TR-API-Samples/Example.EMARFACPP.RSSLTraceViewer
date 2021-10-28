# RSSL XML Trace Viewer Tools

The utility was designed for decoding and displaying the value from RSSL XML Tracing log generated by [RFA C++/.NET](https://developers.refinitiv.com/en/use-cases-catalog/refinitiv-real-time) and [EMA/ETA C/C++](https://developers.refinitiv.com/en/use-cases-catalog/refinitiv-real-time). The main purpose of the project is to create WPF GUI application to display data for [MRN Real-Time News](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-cc/documentation#mrn-data-models-and-refinitiv-real-time-implementation-guide) provided by RIC MRN_STORY. Anyway, the core functionality of the application can be used to decode the value of the field entry data inside the XML element. Therefore, the application can be used to open RSSL tracing log which contains data from other message model type such as Market Price, Market By Price and Market By Order. This utility should be able to help verify data issue or confirm the Market Price data or MRN Real-Time news data that API sends and receive from Provider Server or TREP component. Please note that RSSL Tracing log and this utility was provided for troubleshooting purpose only.

## Prerequisites

Required software components:

* [.NET Framework](https://www.microsoft.com/net/download/dotnet-framework/net461) - .NET Framework 4.6.1 or later version
* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/)- We use Visual Studio 2017 with C# 7.0 to develop the application.

Optional software components:

* [Visual Studio Code](https://code.visualstudio.com/) - IDE for edit source codes and open XML and JSON output from the application.

## Getting Started

The utility can be used on Windows OS only because it uses .NET Framework with WPF library which is not available for another OS. It also required Visual Studio 2017 with .NET framework 4.6.1 to build the application.

In addition to .NET Framework, the project required additional libraries for decoding XML data and parsing the JSON data and it required RDMFieldDictionary and enumtype.def to decode the raw data in Hex string format to the actual value.

A user can update dictionary inside folder “Dict” to the latest version. In case that the application unable to find field definition for some specific fid or custom fid and enum value, it will instead display original Hex string with the fid id instead of showing fid name. Note that you can download the latest version of data dictionary from the [Software Download page](https://my.refinitiv.com/content/mytr/en/downloadcenter.html) and then search for product name "TREP Template service pack".

Anyway, if you want the utility which can run on multiple OS such as macOS and Linux, we have created the [RSSLTraceConverter](https://github.com/Refinitiv-API-Samples/Example.EMARFACPP.Tool.RSSLXMLTraceConverter) which is console application which can decode and convert the data from RSSL tracing log to a new XML file. It shares the core functionality with this project. You can download it from [github](https://github.com/Refinitiv-API-Samples/Example.EMARFACPP.Tool.RSSLXMLTraceConverter).

# Overview

As described earlier, the application was designed for decoding data inside RSSL XML trace log which generated by RFA C++/.NET and EMA/ETA C/C++ and it can be used to display decoded data in WPF GUI. Basically, the RSSL XML trace file was provided for troubleshooting purpose only and there are some scenarios that required data from the RSSL Trace log to confirm data integrity the application received, therefore, there is the requirement to create a tool which can convert field entry value in Hex string format to actual value so that the user can compare the value with data the application received.

The utility can be used to read the XML fragment and decode the Hex string in the field entry of the Market Price and Level 2 Market data to actual value according to the data type from the data dictionary. Moreover, it can be used to decode the [MRN Real-Time News](https://developers.refinitiv.com/en/api-catalog/refinitiv-real-time-opnsrc/rt-sdk-cc/documentation#mrn-data-models-and-refinitiv-real-time-implementation-guide) which provide by using __MRN_STORY__ RIC code. The utility has an internal algorithm to scan the XML fragments and find a series of compressed MRN fragment data inside the tracing log. Then it can unpack the data and shows the MRN JSON data on the GUI.

## How to collect RSSL Tracing log

The RSSL tracing log was provided for troubleshooting purpose and it's not turn on by default. A user may use the following instruction to turn on the log in RFA C++/.NET and EMA C++ application. Please note that currently, this utility does not support the trace from RFA and EMA java because it can't provide a valid XML trace format. There are some garbage or unexpected text inside the XML fragment and there are mixing between simple logger message and XML data inside the trace so that it causes the XML parsing error. If you want to use the trace log from EMA java with the application, the workaround is to manually remove invalid text or string from XML fragment and save it to a valid XML file.

### RFA C++ and .NET application

A user has to add the following configuration to turn on RSSL tracing log. <Connection_RSSL> is a connection name which the user is using in the configuration file.

```
\Connections\<Connection_RSSL>\connectionType = "RSSL"
\Connections\<Connection_RSSL>\rsslPort = "<RSSL Port>"
\Connections\<Connection_RSSL>\hostName = "<ADS/Provider Hostname>"
\Connections\<Connection_RSSL>\traceMsg = false
\Connections\<Connection_RSSL>\traceMsgToFile = true
\Connections\<Connection_RSSL>\tracePing = true
\Connections\<Connection_RSSL>\traceMsgFileName = "RSSLConsumerTrace"
\Connections\<Connection_RSSL>\traceMsgDomains = "all"
\Connections\<Connection_RSSL>\traceRespMsg = true
\Connections\<Connection_RSSL>\traceReqMsg = true
\Connections\<Connection_RSSL>\traceMsgHex = false
\Connections\<Connection_RSSL>\traceMsgMaxMsgSize = 200000000
\Connections\<Connection_RSSL>\traceMsgMultipleFiles = true
```

Note that traceMsgFileName is an output of the XML file used by RFA to generate the log. From the example provided above, the output will be RSSLConsumerTrace_<pid>.xml where <pid> is the process id of the application.

### EMA C++ application

For EMA C++ application, to turn on the RSSL Trace log, a user has to copy EmaConfig.xml which provided in EMA Examples folder to project directory or running directory and then add below XML element to EMA configuration file under section Consumer.Note that you have to set __XmlTraceToFile__ to __1__ to turn on the log and set it to __0__ to turn off the log.

```
<DefaultConsumer value="Consumer_1"/>
<Consumer>
    <Name value="Consumer_1"/>
    ...
    <XmlTraceToFile value="1"/>
    <XmlTraceToStdout value="0"/>
</Consumer>
```

There are applications that setting server name/IP address and the RSSL port from application code and they might need to turn on the RSSL tracing log. This can be achieved by copying EMA configuration file to running directly. Though you are not using the Channel config from the configuration file, you can also copy the configuration file to the running directory and set the config under DefaultConsumer or DefaultProvider section. 

From the above sample, we set DefaultConsumer to "Consumer_1" then EMA will check the value inside the configuration file and then verify option to turn on trace file from Consumer_1. EMA will generate the RSSL Tracing log file name EmaTrace_<id>.xml under the running directory.

## Utility download

The solution project is available from [Github](https://github.com/Refinitiv-API-Samples/Example.EMARFACPP.Tool.RSSLTraceViewer).

## Building and Running Utility

To build the application you just need to clone the solution from GitHub and then open solution file __RSSLTraceViewerGUI.sln__ with Visual Studio 2017. Then building the project with Debug or Release build.

![VisualStudio2017](/images/VS2017.PNG)

### Launching the application

You can build the utility using Visual Studio 2017 and then launch the application by click RSSLTraceViewerGUI.exe which locates under Release or Debug folder like the following sample path "<GitHubRepo>\Example.EMARFACPP.Tool.RSSLTraceViewer\RSSLTraceViewerGUI\bin\Release"

![LaunchingApplication](/images/launch.PNG)

It will show WPF GUI like the following screenshot.

![MainPage](/images/mainpagebullet.png)

There are 4 bullets from the screenshot and the following description is information for each bullet.

1. Start using the application, a user has to click at bullet 1 which is __Load Trace file__ button to load RSSL Trace log XML file.
2. Bullet 2 is a text box for display path to XML file after a user clicked __Load Trace file__ button.
3. Bullet 3 is the area of DataGrid control for displaying XML fragment list ordered by sequence of XML fragment from the XML Tracing log. The DataGrid also have columns for displaying information such as a time stamp, domain type, stream id and GUID in case that DataGrid row represents data for MRN.
4. Bullet 4 is the area of Tree View component for displaying decoded data. The application will generate a Tree View from the structure of the XML fragment.

### Usages

To start using the application, the client just clicks __Load Trace file__ button to load XML file as described in the previous section and then the application will process the XML data and create the internal data structure to keep XML fragment list and then bind the list to DataGrid.

![Load XML Trace file](/images/mainpage5.PNG)

The screenshot below is a sample when the application displaying a list of XML fragment in Data Grid.

![MainPage](/images/mainpage_loaded.png)

There is an additional information from the application shows on the Windows caption. The following sample is details about a number of XML fragments the application read from the XML file.

![Additional Info](/images/AdditionalTab.png)

The user can resize the windows and click at the Grid separator at the position of Bullet 3 from the screenshot below. It allows a user to drag the Grid separator to left or right.

![GridSerator](/images/MovePane.png)

#### Unpack MRN Data

To unpack or display uncompressed JSON data for MRN Real-Time news, the first step, a user has to double click DataGrid row which contains GUID they interest in order to display data in TreeView at the right-hand side.

Actually, the application does not decode XML fragment data immediately after it loads the XML file. Instead, it will decode the data when a user double-clicks one of DataGrid row. This should help reduce XML loading time at the first step.

![ClickRow](/images/MRNLoaded2.png)

The screenshot above represent the data from the RSSL tracing log which contains MRN Real-Times news data from MRN_STORY RIC. After user double click DataGrid row, it will generate a new TreeView control to display XML data inside bullet 2 area. Bullet 3 is button __Unpack MRN Data__ which the user can use to unpack MRN data from selected DataGrid row. Basically, the application has an internal algorithm to find relevance XML fragments and calls the internal function to decompress the data and shows in a new Window like the next sample screenshot.

![Display News](/images/DisplayMRNJson.png)

Note that the __Unpack MRN Data__ button will appear when the selected row containing GUID only. Moreover, if selected GUID contains invalid fragments list, the application will pop up a message box with details of the error. For example, when overall XML Fragments size less than or larger than total size.

#### Display and Save MRN JSON data to a file

From the screenshot to display MRN JSON data from the previous section, there are four bullets on the window.

1. Bullet 1 is text block for display uncompressed JSON data.
2. Bullet 2 is a list of relevant XML fragments that displaying inside the TreeView component.
3. Bullet 3 is a button for saving the raw JSON data to a new file. A user can open the JSON file in a text editor to review data inside XML Fragments.
4. Bullet 4 is the area for display information about the GUID.

#### Display Level 1 and Level 2 Market Price data

The user can just load the RSSL XML trace log which contains Market Price or Level 2 Market data. The step to display data is the same as MRN but the difference is that there is no button __Unpack MRN data__ at the right-hand side. The screenshot below is a sample output when application displaying Market Price data. It will display the decoded value on the area of bullet 2.

Basically, the application needs to verify whether or not the XML element contains the fieldEntry element and then use an internal library to get Fid name and decoding field value before adding the value to the TreeView component. However, if the Fid is not available in the data dictionary, the application will display value as original Hex string instead.

![Display News](/images/MarektPriceBullet.png)

#### Filtering data inside DataGrid

There is an option for the user to filter data by clicking the small button at the DataGrid column. There are some columns such as Msg Type, Key (item name or username in the message), Domain and Stream Id that can be used to filter data. The application also provides an interface to search MRN data by using GUID.

After the user clicks at the small button on the column header, the application will show a Combo box containing a list of available filter values for each column. Then the user can just tick or un-tick a Checkbox inside the Combo box to select the value they interest. The application will apply the filter values with internal XML Fragments data and then display data inside DataGrid based on the filtered result. Sample screenshots below is a result when the user clicks the button on the column header.

![DomainFiltering](/images/filterDomainBullet.png)

![MsgTypeFiltering](/images/filtermsgtye.png)

The following screenshot is a result when the user wants to display refreshMsg only.

![MsgTypeFiltering](/images/msgtyperesult.PNG)

#### Searching GUID

Basically, when the data inside the RSSL Tracing log contains MRN data with GUID list, the application will show GUID search box above the DataGrid area. The user just inputs GUID text in GUID search box and the click __Search__ button. The application will perform searching the GUID based on data inside the DataGrid. The following screenshot is the sample output.

![MsgTypeFiltering](/images/GuidSearchBullet.png)

Bullet 1 is GUID value the user wants to search and Bullet 2 is the result. The functionality should be able to help user confirm whether or not the RFA or EMA application receive some specific GUID or not.

## Limitation of the Utility

1) The current version of the XML parser used by the Utility was designed and tested with the RSSL tracing log from Elektron SDK C++ and the new version of RFA C++ 7.6 or later version. So it may not support the log from the old version of RFA C++ or UPA. This is because the old version uses a different XML format and structure. The XML parser requires additional XML comment come before the XML fragment containing the RSSL message, in order to get a type of message (Incoming or Outgoing) and the time stamp the message sent or receive. It also required some XML attribute such as domainType and streamId in the first XML element inside XML fragment. The supported XML structure should contain XML element and its attribute like the following sample.

A sample of requestMsg.

```XML

<!-- Outgoing Message <string> -->
<!-- Time: <Time String> -->
<!-- rwfMajorVer="<major version>" rwfMinorVer="<minor version>" -->
<requestMsg domainType="<Domain Type>" streamId="<Stream Id>"... >
    <key ... name="<key name>" ...>
    ...

    </key>

    <dataBody>
    </dataBody>
</requestMsg>

```

Sample of updateMsg or refreshMsg.

```XML

<!-- Incoming Message <Message> -->
<!-- Time: <Time> -->
<!-- rwfMajorVer="14" rwfMinorVer="0" -->
<refreshMsg domainType="<Domain Type>" streamId="<Stream Id>" ...>
    <key ... name="VOD.L" .../>
    <dataBody>
        <fieldList flags="0x9 (RSSL_FLF_HAS_FIELD_LIST_INFO|RSSL_FLF_HAS_STANDARD_DATA)" fieldListNum="194" dictionaryId="1">
            <fieldEntry fieldId="1" data="15F9"/>
            <fieldEntry fieldId="2" data="73"/>
            <fieldEntry fieldId="3" data="564F 4441 464F 4E45 2047 524F 5550"/>
            ...
            <fieldEntry fieldId="32741" data=""/>
            <fieldEntry fieldId="32743" data=""/>

        </fieldList>
    </dataBody>
</refreshMsg>

<!-- Incoming Message <Message> -->
<!-- Time: <Time> -->
<!-- rwfMajorVer="<major version>" rwfMinorVer="<minorversion>" -->
<updateMsg domainType="<Domain Type>" streamId="<Stream Id>" ...>
...
    <mapEntry flags="0x0" action="RSSL_MPEA_ADD_ENTRY" key="5@4154DB03" >
        <fieldList flags="0x8 (RSSL_FLF_HAS_STANDARD_DATA)">\
            <fieldEntry fieldId="3427" data="0B01 0842"/>
            <fieldEntry fieldId="3429" data="0E03 20"/>
            <fieldEntry fieldId="3428" data="01"/>
            <fieldEntry fieldId="3426" data="3130 3936 3038 3031 3331"/>
            <fieldEntry fieldId="6520" data="014C E5F9"/>
            <fieldEntry fieldId="6522" data="040A 07E2"/>
        </fieldList>
    </mapEntry>
...
</updateMsg>

```

2) The XML Parser will stop parsing the XML file when it found parsing error in XML such as an invalid character or incomplete XML element. The application will popup message box with additional details about the exception from .NET XML class used by the application. A user may need to manually remove invalid XML tag or XML element from the XML file.

3) The application can just decoding data in XML element name "fieldEntry" and it must contain attributes name "fieldId" and "data". Otherwise the application unable to parse the field data inside RSSL message.

4) As described in the first limitation that the format of XML element must contain the same structure as RSSL Trace from EMA C++ or RFA C++ 7.6 and later version. However, there is an option for the user who wants to use the utility to decode a snapshot of data. There are additional configuration named "__UseRDMStrictMode__"(default is True) in __app.config.json__ locates under the running directory. The user can set the configuration to __False__ to allow the utility parsing the XML fragments inside the XML file without checking the format or structure of the XML message. Anyway, the XML file must contain fieldEntry XML element in order to parse the data. Otherwise, it could lead to an unexpected result.

For example,

    You want to decode fieldEntry inside specific update messages such as Market By Price update and MRN updates messages like the following sample XML data.

```XML
<updateMsg domainType="RSSL_DMT_MARKET_BY_ORDER" streamId="4" containerType="RSSL_DT_MAP" flags="0x10 (RSSL_UPMF_HAS_SEQ_NUM)" updateType="0 (RDM_UPD_EVENT_TYPE_UNSPECIFIED)" seqNum="15536" dataSize="79">
    <dataBody>
        <map flags="0x2 (RSSL_MPF_HAS_SUMMARY_DATA)" countHint="0" keyPrimitiveType="RSSL_DT_BUFFER" containerType="RSSL_DT_FIELD_LIST">
            <summaryData>
                <fieldList flags="0x8 (RSSL_FLF_HAS_STANDARD_DATA)">
                    <fieldEntry fieldId="4148" data="014C E5F9"/>
                </fieldList>
            </summaryData>
            <mapEntry flags="0x0" action="RSSL_MPEA_ADD_ENTRY" key="5@4154DB03">
                <fieldList flags="0x8 (RSSL_FLF_HAS_STANDARD_DATA)">
                    <fieldEntry fieldId="3427" data="0B01 0842"/>
                    <fieldEntry fieldId="3429" data="0E03 20"/>
                    <fieldEntry fieldId="3428" data="01"/>
                    <fieldEntry fieldId="3426" data="3130 3936 3038 3031 3331"/>
                    <fieldEntry fieldId="6520" data="014C E5F9"/>
                    <fieldEntry fieldId="6522" data="040A 07E2"/>
                </fieldList>
            </mapEntry>
        </map>
    </dataBody>
</updateMsg>

<updateMsg domainType="RSSL_DMT_NEWS_TEXT_ANALYTICS" streamId="3" containerType="RSSL_DT_FIELD_LIST" flags="0x1D2 (RSSL_UPMF_HAS_PERM_DATA|RSSL_UPMF_HAS_SEQ_NUM|RSSL_UPMF_DO_NOT_CACHE|RSSL_UPMF_DO_NOT_CONFLATE|RSSL_UPMF_DO_NOT_RIPPLE)" updateType="0 (RDM_UPD_EVENT_TYPE_UNSPECIFIED)" seqNum="3694" permData="0308 4310 154C" dataSize="2717">
    <dataBody>
        <fieldList flags="0x8 (RSSL_FLF_HAS_STANDARD_DATA)">
            <fieldEntry fieldId="4148" data="0226 3EF3"/>
            <fieldEntry fieldId="17" data="0A0A 07E2"/>
            <fieldEntry fieldId="8593" data="5354 4F52 59"/>
            <fieldEntry fieldId="8506" data="32"/>
            <fieldEntry fieldId="11787" data="3130"/>
            <fieldEntry fieldId="32480" data="0AF9"/>
            <fieldEntry fieldId="32479" data="01"/>
            <fieldEntry fieldId="4271" data="5253 4A35 3836 3444 615F 3138 3130 3130 3269 472F 4B42 2B72 456C 6239 6F41 464A
4138 5A53 5373 4831 4C42 6232 2B2F 6F54 6C37 5736 7644"/>
...
        </fieldList>
    </dataBody>
</updateMsg>

<updateMsg domainType="RSSL_DMT_NEWS_TEXT_ANALYTICS" streamId="3" containerType="RSSL_DT_FIELD_LIST" flags="0x1D2 (RSSL_UPMF_HAS_PERM_DATA|RSSL_UPMF_HAS_SEQ_NUM|RSSL_UPMF_DO_NOT_CACHE|RSSL_UPMF_DO_NOT_CONFLATE|RSSL_UPMF_DO_NOT_RIPPLE)" updateType="0 (RDM_UPD_EVENT_TYPE_UNSPECIFIED)" seqNum="3710" permData="0308 4310 154C" dataSize="288">
    <dataBody>
        <fieldList flags="0x8 (RSSL_FLF_HAS_STANDARD_DATA)">
            <fieldEntry fieldId="4271" data="5253 4A35 3836 3444 615F 3138 3130 3130 3269 472F 4B42 2B72 456C 6239 6F41 464A
4138 5A53 5373 4831 4C42 6232 2B2F 6F54 6C37 5736 7644"/>
            <fieldEntry fieldId="12215" data="4844 435F 5052 445F 41"/>
            <fieldEntry fieldId="32479" data="02"/>
            <fieldEntry fieldId="32641" data="4586 490A 8756 9191 83E2 0BD2...>
        </fieldList>
    </dataBody>
</updateMsg>

```

The user can just copy these three XML fragments to a new XML file and then set __UseRDMStrictMode__ to __False__ and re-launch the utility. It should be able to parse the XML data and decode the MRN message if it contains all MRN fragments in the XML file. The application will show the data in the GUI like below screenshot.



![NoRDMStrictMode](/images/StrictMode.png)

Anyway, instead of using this utility to open this kind of XML Trace messages, we strongly suggest the user use [RSSL XML Trace Converter](https://github.com/Refinitiv-API-Samples/Example.EMARFACPP.Tool.RSSLXMLTraceConverter) instead. The command line utility can convert the file to a new XML file contains the decoded value.

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Authors

* **Moragodkrit Chumsri** - Release 1.0 *Initial work*

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

