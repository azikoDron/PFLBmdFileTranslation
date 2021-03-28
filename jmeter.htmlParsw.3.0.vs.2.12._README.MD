# Project description 

## Object of testing

HtmlParsers for **Apache.JMeter 2.13** and **Apache.JMeter 3.0** are being test.

Apache.JMeter 2.13 parsers:
- LagartoBasedHtmlParser
- HtmlParserHTMLParser
- JTidyHTMLParser
- RegexpHTMLParser
- JsoupBasedHtmlParser

Apache.JMeter 3.0 parsers:
- LagartoBasedHtmlParser
- JTidyHTMLParser
- RegexpHTMLParser
- JsoupBasedHtmlParser

The parsers analyse the home pages of various websites:
- stackoverflow.com
- habrahabr.ru
- yandex.ru
- mos.ru
- jmeter.apache.org
- google.ru
- linkedin.com
- github.com

## Reason of testing

The reason for testing was changes in Apache.JMeter 3.0, 
see [http://jmeter.apache.org/changes.html](http://jmeter.apache.org/changes.html).

Excerpts from the list of changes:

> #### Core improvements
> ##### Dependencies refresh
> 
> Deprecated Libraries dropped or replaced by up to date ones:
> 
> - htmllexer, htmlparser removed
> - jdom removed
> 
> > Removed the **htmlparser** and unused **jdom** library.
> 
> #### Protocols and Load Testing improvements
> ##### Parallel Downloads is now realistic and scales much better:
> 
> - Parsing of CSS imported files (through @import) or embedded resources (background, images, …)
> 
> > Added a new parser for CSS files, links to other CSS files (via @import) and links to resources specified in CSS 
> files will be extracted: background images, pictures, ...
> 
> #### Incompatible changes
> 
> - Since version 3.0, the parser for embedded resources (replaced since 2.10 by Lagarto based implementation) which relied on the htmlparser library (HtmlParserHTMLParser) has been dropped along with its dependencies.
> - The following jars have been removed:
>    - htmllexer-2.1.jar (see [Bug 59037](http://bz.apache.org/bugzilla/show_bug.cgi?id=59037))
>    - htmlparser-2.1.jar (see [Bug 59037](http://bz.apache.org/bugzilla/show_bug.cgi?id=59037))
>    - jdom-1.1.3.jar (see [Bug 59156](http://bz.apache.org/bugzilla/show_bug.cgi?id=59156))
> 
> > Removed the **htmlparser** and unused **jdom** library.
> 
> #### Improvements
> ##### HTTP Samplers and Test Script Recorder
> 
> - [Bug 59036](http://bz.apache.org/bugzilla/show_bug.cgi?id=59036) - FormCharSetFinder : Use JSoup instead of deprecated HTMLParser
> - [Bug 59033](http://bz.apache.org/bugzilla/show_bug.cgi?id=59033) - Parallel Download : Rework Parser classes hierarchy to allow plug-in parsers for different mime types
> - [Bug 59140](http://bz.apache.org/bugzilla/show_bug.cgi?id=59140) - Parallel Download : Add CSS Parsing to extract links from CSS files
> 
> >**JSoup** is now used to find the `accept-charset` attribute in the `form` tags instead of the 
> remote **HTMLParser** [Bug 59036]. CSS file parser [Bug 59140] is implemented and this parser is
> used by default [Bug 59033].

## Testing goals

Compare the work of all available parsers. In particular, compare parsers of versions 2.13 and 3.0 
with each other, make sure that loading built-in resources has become more realistic and better.

## Testing strategy

Step 1:

1. Download the home pages of the list of sites using all 5 **Apache.JMeter 2.13** parsers and 
   write down logs.
2. Download the home pages of the list of sites using all 4 **Apache.JMeter 3.0** parsers and 
   write down the logs.
3. Analyse **Apache.JMeter's** logs and compare them with each other. To assess whether the download 
   of built-in resources has become better, whether the list of embedded resources to be loaded has expanded.

Step 2:
1. Download the home pages of the list of popular sites using Google Chrome and **webpagetest.org**.
2. Analyse reports from **webpagetest.org** and compare them with the results of analysing **Apache.JMeter**
   logs. To assess the realism of loading built-in resources.

## Testing approach

To determine exactly how many requests are sent during the opening of the site page from
**Apache.JMeter**, all requests are logged in:
- **View Results Tree** - statistical logger, logging into XML format with subquery logging, XML 
  log will be used to find out the details of requests/answers/errors;

- **CsvLogWriter** is a custom logger
  [https://github.com/pflb/Jmeter.Plugin.CsvLogWriter](https://github.com/pflb/Jmeter.Plugin.CsvLogWriter), 
  logging to CSV format with subquery logging, CSV log will be used for programmatic calculation of statistics 
  on the work of various parsers.

To be able to group queries by Apache.JMeter versions, parsers and sites, additional variables for each query will be written to the log:
- **siteKey** is a site being tested;
- **jmeterVersion** - Apache.JMeter version;
- **htmlParser** is the name of the html-parser currently in use.

# Results

## Summary

### Assessment of the improvement in parsers for version 3.0 compared to version 2.13

There are no dramatic improvements in the completeness of the analysis of html pages, there are deteriorations.

A significant difference is that the parsers for Apache.JMeter 3.0 have recursive loading of the Yandex browser 
promo material page. This manifests itself when downloading https://yandex.ru/.

####Sites with a small amount of content are a good result.

On simple sites like **jmeter.apache.org**, all parsers work the same way. Creating the same number of subrequests that 
the browser creates. The quality of parsers for **jmeter.apache.org** is perfect, 100%.

####Sites with a lot of content are a bad result.

But on a site like *mos.ru*, parsers will find an average of 22 links to built-in resources, while a full page load with 
the load of all built-in resources by the browser - 144 requests. The quality is poor.

Similarly, on *habrahabr.ru*, the *Lagardo* parser from **Apache.JMeter 3.0** will find 55 links, while the browser will 
make 117 subquests. The quality is 47.01%. Satisfactory quality of completeness of retrieve references to embedded 
resources.

## Number of requests when using different parsers

Table on Google Docs: 
[JMeter.HtmlParser.Compare (top table)](https://docs.google.com/spreadsheets/d/16l32BFuUdrJmexTgF155HMAGW7EBaBT-rRjuuLc524c/edit?usp=sharing).

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th rowspan="2">Сайт</th>
      <th colspan="8">Apache.JMeter </th>
      <th colspan="2">Chrome, webpagetest.org, подзапросы</th>
      <th rowspan="2" title="Качество парсинга (Avg / Document&nbsp;Complete)">Качество</th>
    </tr>
    <tr>
      <th>Версия</th>
      <th>Парсер</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th title="Среднее значение по пяти итерациям">Avg</th>
      <th>Document Complete</th>
      <th>Fully Loaded</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="9" valign="top">github.com</th>
      <th rowspan="5" valign="top">2.13</th>
      <th>HtmlParser</th>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td><!--1.AP-->
      <td rowspan="9">17</td><!--4.DC-->
      <td rowspan="9"><b>20</b></td><!--5.FL-->
      <td>60,00%</td>
    </tr>
    <tr>
      <th>JTidy</th>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td><!--1-->
      <td>60,00%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td><!--1-->
      <td>60,00%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>12<b></td>
      <td><b>12<b></td>
      <td><b>12<b></td>
      <td><b>12<b></td>
      <td><b>12<b></td>
      <td><b>12<b></td><!--1-->
      <td><b>60,00%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td><!--1-->
      <td>60,00%</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">3.0</th>
      <th>JTidy</th>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td><!--1-->
      <td>60,00%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td><!--1-->
      <td>60,00%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>12<b></td>
      <td><b>12<b></td>
      <td><b>12<b></td>
      <td><b>12<b></td>
      <td><b>12<b></td>
      <td><b>12<b></td><!--1-->
      <td><b>60,00%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td>
      <td>12</td><!--1-->
      <td>60,00%</td>
    </tr>
    <tr>
      <th rowspan="9" valign="top">google.ru</th>
      <th rowspan="5" valign="top">2.13</th>
      <th>HtmlParser</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td><!--1.AP-->
      <td rowspan="9">9</td><!--4.DC-->
      <td rowspan="9"><b>12</b></td><!--5.FL-->
      <td>33,33%</td>
    </tr>
    <tr>
      <th>JTidy</th>
      <td>3</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>3,8</td><!--1.AP-->
      <td>31,67%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td><!--1.AP-->
      <td>33,33%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>4</b></td>
      <td><b>4</b></td>
      <td><b>4</b></td>
      <td><b>4</b></td>
      <td><b>4</b></td>
      <td><b>4</b></td><!--1.AP-->
      <td><b>33,33%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td><!--1.AP-->
      <td>16,67%</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">3.0</th>
      <th>JTidy</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td><!--1.AP-->
      <td>33,33%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td><!--1.AP-->
      <td>33,33%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>3</b></td>
      <td><b>4</b></td>
      <td><b>4</b></td>
      <td><b>4</b></td>
      <td><b>4</b></td>
      <td><b>3,8</b></td><!--1.AP-->
      <td><b>31,67%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td><!--1.AP-->
      <td>16,67%</td>
    </tr>
    <tr>
      <th rowspan="9" valign="top">habrahabr.ru</th>
      <th rowspan="5" valign="top">2.13</th>
      <th>HtmlParser</th>
      <td>64</td>
      <td>64</td>
      <td>64</td>
      <td>64</td>
      <td>64</td>
      <td>64</td><!--1.AP-->
      <td rowspan="9">112</td><!--4.DC-->
      <td rowspan="9"><b>117</b></td><!--5.FL-->
      <td>54,70%</td>
    </tr>
    <tr>
      <th>JTidy</th>
      <td>64</td>
      <td>64</td>
      <td>64</td>
      <td>64</td>
      <td>64</td>
      <td>64</td><!--1.AP-->
      <td>54,70%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>64</td>
      <td>64</td>
      <td>64</td>
      <td>64</td>
      <td>64</td>
      <td>64</td><!--1.AP-->
      <td>54,70%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>61<b></td>
      <td><b>61<b></td>
      <td><b>61<b></td>
      <td><b>61<b></td>
      <td><b>61<b></td>
      <td><b>61<b></td><!--1.AP-->
      <td><b>52,14%<b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>65</td>
      <td>65</td>
      <td>65</td>
      <td>65</td>
      <td>65</td>
      <td>65</td><!--1.AP-->
      <td>55,56%</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">3.0</th>
      <th>JTidy</th>
      <td>58</td>
      <td>58</td>
      <td>58</td>
      <td>58</td>
      <td>58</td>
      <td>58</td><!--1.AP-->
      <td>49,57%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>58</td>
      <td>58</td>
      <td>58</td>
      <td>58</td>
      <td>58</td>
      <td>58</td><!--1.AP-->
      <td>49,57%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>55<b></td>
      <td><b>55<b></td>
      <td><b>55<b></td>
      <td><b>55<b></td>
      <td><b>55<b></td>
      <td><b>55<b></td><!--1.AP-->
      <td><b>47,01%<b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>59</td>
      <td>59</td>
      <td>59</td>
      <td>59</td>
      <td>59</td>
      <td>59</td><!--1.AP-->
      <td>50,43%</td>
    </tr>
    <tr>
      <th rowspan="9" valign="top">jmeter.apache.org</th>
      <th rowspan="5" valign="top">2.13</th>
      <th>HtmlParser</th>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td><!--1.AP-->
      <td rowspan="9">9</td><!--4.DC-->
      <td rowspan="9"><b>10</b></td><!--5.FL-->
      <td>90,00%</td>
    </tr>
    <tr>
      <th>JTidy</th>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td><!--1.AP-->
      <td>90,00%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td><!--1.AP-->
      <td>90,00%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>9</b></td>
      <td><b>9</b></td>
      <td><b>9</b></td>
      <td><b>9</b></td>
      <td><b>9</b></td>
      <td><b>9</b></td><!--1.AP-->
      <td><b>90,00%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td><!--1.AP-->
      <td>90,00%</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">3.0</th>
      <th>JTidy</th>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td><!--1.AP-->
      <td>100,00%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td><!--1.AP-->
      <td>100,00%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td><!--1.AP-->
      <td><b>100,00%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td><!--1.AP-->
      <td>100,00%</td>
    </tr>
    <tr>
      <th rowspan="9" valign="top">linkedin.com</th>
      <th rowspan="5" valign="top">2.13</th>
      <th>HtmlParser</th>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td><!--1.AP-->
      <td rowspan="9">19</td><!--4.DC-->
      <td rowspan="9"><b>21</b></td><!--5.FL-->
      <td>47,62%</td>
    </tr>
    <tr>
      <th>JTidy</th>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td><!--1.AP-->
      <td>47,62%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td><!--1.AP-->
      <td>47,62%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td><!--1.AP-->
      <td><b>47,62%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>6</td>
      <td>10</td>
      <td>6</td>
      <td>6</td>
      <td>10</td>
      <td>7,6</td><!--1.AP-->
      <td>36,19%</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">3.0</th>
      <th>JTidy</th>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td><!--1.AP-->
      <td>47,62%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td>
      <td>10</td><!--1.AP-->
      <td>47,62%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td>
      <td><b>10</b></td><!--1.AP-->
      <td><b>47,62%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>10</td>
      <td>10</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>7,6</td><!--1.AP-->
      <td>36,19%</td>
    </tr>
    <tr>
      <th rowspan="9" valign="top">mos.ru</th>
      <th rowspan="5" valign="top">2.13</th>
      <th>HtmlParser</th>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td><!--1.AP-->
      <td rowspan="9">119</td><!--4.DC-->
      <td rowspan="9"><b>144</b></td><!--5.FL-->
      <td>16,67%</td>
    </tr>
    <tr>
      <th>JTidy</th>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td><!--1.AP-->
      <td>16,67%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td><!--1.AP-->
      <td>16,67%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>13</b></td>
      <td><b>13</b></td>
      <td><b>13</b></td>
      <td><b>13</b></td>
      <td><b>13</b></td>
      <td><b>13</b></td><!--1.AP-->
      <td><b>9,03%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>24</td>
      <td>24</td>
      <td>0</td>
      <td>24</td>
      <td>24</td>
      <td>19,2</td><!--1.AP-->
      <td>13,33%</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">3.0</th>
      <th>JTidy</th>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td><!--1.AP-->
      <td>17,36%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td><!--1.AP-->
      <td>17,36%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>14</b></td>
      <td><b>14</b></td>
      <td><b>14</b></td>
      <td><b>14</b></td>
      <td><b>14</b></td>
      <td><b>14</b></td><!--1.AP-->
      <td><b>9,72%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td><!--1.AP-->
      <td>17,36%</td>
    </tr>
    <tr>
      <th rowspan="9" valign="top">stackoverflow.com</th>
      <th rowspan="5" valign="top">2.13</th>
      <th>HtmlParser</th>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td><!--1.AP-->
      <td rowspan="9">41</td><!--4.DC-->
      <td rowspan="9"><b>42</b></td><!--5.FL-->
      <td>40,48%</td>
    </tr>
    <tr>
      <th>JTidy</th>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td><!--1.AP-->
      <td>40,48%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td><!--1.AP-->
      <td>38,10%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>16</b></td>
      <td><b>16</b></td>
      <td><b>16</b></td>
      <td><b>16</b></td>
      <td><b>16</b></td>
      <td><b>16</b></td><!--1.AP-->
      <td><b>38,10%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>17</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td><!--1.AP-->
      <td>16,2%</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">3.0</th>
      <th>JTidy</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td><!--1.AP-->
      <td>38,10%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td><!--1.AP-->
      <td>38,10%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>15</b></td>
      <td><b>15</b></td>
      <td><b>15</b></td>
      <td><b>15</b></td>
      <td><b>15</b></td>
      <td><b>15</b></td><!--1.AP-->
      <td><b>35,71%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td><!--1.AP-->
      <td>38,10%</td>
    </tr>
    <tr>
      <th rowspan="9" valign="top">yandex.ru</th>
      <th rowspan="5" valign="top">2.13</th>
      <th>HtmlParser</th>
      <td>70</td>
      <td>68</td>
      <td>72</td>
      <td>67</td>
      <td>69</td>
      <td>69,2</td><!--1.AP-->
      <td rowspan="9">33</td><!--4.DC-->
      <td rowspan="9"><b>35</b></td><!--5.FL-->
      <td>197,71%</td>
    </tr>
    <tr>
      <th>JTidy</th>
      <td>18</td>
      <td>18</td>
      <td>19</td>
      <td>18</td>
      <td>18</td>
      <td>18,2</td><!--1.AP-->
      <td>52,00%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td><!--1.AP-->
      <td>51,43%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>14</b></td>
      <td><b>14</b></td>
      <td><b>14</b></td>
      <td><b>14</b></td>
      <td><b>14</b></td>
      <td><b>14</b></td><!--1.AP-->
      <td><b>40,00%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>19</td>
      <td>18,2</td><!--1.AP-->
      <td>52,00%</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">3.0</th>
      <th>JTidy</th>
      <td>647</td>
      <td>645</td>
      <td>619</td>
      <td>644</td>
      <td>578</td>
      <td>626,6</td><!--1.AP-->
      <td>1790,29%</td>
    </tr>
    <tr>
      <th>Jsoup</th>
      <td>579</td>
      <td>640</td>
      <td>638</td>
      <td>644</td>
      <td>620</td>
      <td>624,2</td><!--1.AP-->
      <td>1783,43%</td>
    </tr>
    <tr>
      <th>Lagarto</th>
      <td><b>48</b></td>
      <td><b>451</b></td>
      <td><b>441</b></td>
      <td><b>450</b></td>
      <td><b>450</b></td>
      <td><b>368</b></td><!--1.AP-->
      <td><b>1051,43%</b></td>
    </tr>
    <tr>
      <th>Regexp</th>
      <td>642</td>
      <td>644</td>
      <td>337</td>
      <td>642</td>
      <td>644</td>
      <td>581,8</td><!--1.AP-->
      <td>1662,29%</td>
    </tr>
  </tbody>
</table>

![JMeter.HtmlParser.Compare](images/JMeter.HtmlParser.Compare.1-2.PNG)
![JMeter.HtmlParser.Compare](images/JMeter.HtmlParser.Compare.3-5.PNG)
![JMeter.HtmlParser.Compare](images/JMeter.HtmlParser.Compare.6-8.PNG)

Column description:

- **Before Start Render** - the number of requests made by the browser before the page content is displayed. 
  These are html markup, basic js and css files, basic images.
- **Document Complete** - the number of requests made by the browser at the time of full download of the document. 
  All the resources of the page have already been loaded here.
- **Fully Loaded** - the number of requests made by the browser at the time when javascript worked out, when 
  everything was loaded.

A good result of parsers will be if there are as many requests as **Google Chrome** does at the time of 
**Document Complete**, and excellent at the time of **Fully Loaded**. When using a particular parser,
**Apache.JMeter** will consider the number of requests close to the number of requests executed by the browser 
at the time of **Fully Loaded** to be a measure of realism.

If we exclude the test results of **yandex.ru**, where:

-   Parsing goes into recursion, making requests to **yandex.ru** over and over again until the recursion depth reaches 
the maximum level and ends with an error:
    > `java.lang.Exception: Maximum frame/iframe nesting depth exceeded`.

and for the measure of the quality of parsers' work, take the number of requests at the time of **Fully Loaded**, 
we get such a table of the average quality of parsers' work.

### Average quality of parser work

Table on Google Docs:
[JMeter.HtmlParser.Compare (bottom table)](https://docs.google.com/spreadsheets/d/16l32BFuUdrJmexTgF155HMAGW7EBaBT-rRjuuLc524c/edit?usp=sharing).

![Average quality of parser work](images/parsers.results.PNG)

The most accurate **HTMLParser** parser in **Apache.JMeter 2.13**. In **Apache.JMeter 3.0**, the **Jsoup** and **JTidy**
parsers showed the same quality. **Lagarto**'s parcer lags behind the leaders. The fulness of parsing for the **Lagarto**
parser in **Apache.JMeter 3.0** has decreased compared to **Apache.JMeter 2.13**.

The quality of the **Lagarto** parser on the current version of **Apache.JMeter 3.0** was 32.73%, only a third of 
all subqueries were sent, two-thirds of the static load was not applied.

# Logs and their processing

## Initial data

All logs are available at the link:  
[https://drive.google.com/drive/folders/0B5nKzHDZ1RIiVkN4dDlFWDR1ZGM](https://drive.google.com/drive/folders/0B5nKzHDZ1RIiVkN4dDlFWDR1ZGM "jmeter.htmlParser.compare.logs").

### WebPageTest.org reports

<table>
	<tr>
		<th>sytekey</th>
		<th>webpagetest.org</th>
		<th>Raw page data (.csv)</th>
		<th>Raw object data (.csv)</th>
		<th>HTTP Archive (.har)</th>
	</tr>
	<tr>
		<td>github.com</td>
		<td><a href="https://www.webpagetest.org/result/160819_VF_FM8/">160819_VF_FM8</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiblRnUzNUVFM3SUk">github.com.summary.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIieGFaaWFXTU9MWlE">github.com.details.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIidGNQRzJYUXprVzQ">github.com.har</a></td>
	</tr>
	<tr>
		<td>google.ru</td>
		<td><a href="https://www.webpagetest.org/result/160819_C9_FQD/">160819_C9_FQD</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiczE1OVdCaUs5Mkk">google.ru.summary.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiTUNjRk1mOHJIMEk">google.ru.details.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiYm9EejNNdHBUWnM">google.ru.har</a></td>
	</tr>
	<tr>
		<td>habrahabr.ru</td>
		<td><a href="https://www.webpagetest.org/result/160819_8N_FRB/">160819_8N_FRB</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiM2FsMnVLLTMwb2M">habrahabr.ru.summary.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiaFZjZ2paTlUzaUE">habrahabr.ru.details.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiMG1wZkVub0NRUlU">habrahabr.ru.har</a></td>
	</tr>
	<tr>
		<td>jmeter.apache.org</td>
		<td><a href="https://www.webpagetest.org/result/160819_CG_FSM/">160819_CG_FSM</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiVzlua0tlTnNXd28">jmeter.apache.org.summary.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIieGk0TjVvZlJVd1U">jmeter.apache.org.details.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIieWdLSGFOeEdZMEE">jmeter.apache.org.har</a></td>
	</tr>
	<tr>
		<td>linkedin.com</td>
		<td><a href="https://www.webpagetest.org/result/160819_K2_FY1/">160819_K2_FY1</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiV19IMjhCUThhTVE">linkedin.com.summary.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiUmdHcWxTc1hGcVk">linkedin.com.details.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiWHVfRU1zNE84ODg">linkedin.com.har</a></td>
	</tr>
	<tr>
		<td>mos.ru</td>
		<td><a href="https://www.webpagetest.org/result/160819_91_G0F/">160819_91_G0F</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIibFJqU0hzX1IxbmM">mos.ru.summary.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIibzRGNWVXNVRoQW8">mos.ru.details.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiSUczcXFSXzMzb0E">mos.ru.har</a></td>
	</tr>
	<tr>
		<td>stackoverflow.com</td>
		<td><a href="https://www.webpagetest.org/result/160819_S0_G18/">160819_S0_G18</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIidlFMbXRObkFyVzA">stackoverflow.com.summary.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiYm0wSUV4YWZzWEk">stackoverflow.com.details.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiWW1nck4tU0tvb2c">stackoverflow.com.har</a></td>
	</tr>
	<tr>
		<td>yandex.ru</td>
		<td><a href="https://www.webpagetest.org/result/160819_MR_G1R/">160819_MR_G1R</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiUW5PSXBWMlU4Skk">yandex.ru.summary.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiWml6R3NNQ0J4bk0">yandex.ru.details.csv</a></td>
		<td><a href="https://drive.google.com/open?id=0B5nKzHDZ1RIiQTNkaWFFN0NhOFU">yandex.ru.har</a></td>
	</tr>
</table>

![github.com via webpagetest.org](images/github.com.PNG)
![google.ru via webpagetest.org](images/google.ru.PNG)
![habrahabr.ru via webpagetest.org](images/habrahabr.ru.PNG)
![jmeter.apache.org via webpagetest.org](images/jmeter.apache.org.PNG)
![linkedin.com via webpagetest.org](images/linkedin.com.PNG)
![mos.ru via webpagetest.org](images/mos.ru.PNG)
![stackoverflow.com via webpagetest.org](images/stackoverflow.com.PNG)
![yandex.ru via webpagetest.org](images/yandex.ru.PNG)


###Apache.JMeter logs

Csv-logs generated by the **CsvLogWriter** plugin are used for processing:
- Description: [https://habrahabr.ru/post/308098/](https://habrahabr.ru/post/308098/ "Плагин CsvLogWriter для JMeter ");
- Project: [https://github.com/pflb/Jmeter.Plugin.CsvLogWriter](https://github.com/pflb/Jmeter.Plugin.CsvLogWriter "Jmeter.Plugin.CsvLogWriter").

As a result of which a log is formed, the list of columns of which includes:
- ***TimeStamp*** is a moment of time;
- ***URL*** is the address of the request;
- ***Elapsed*** - the duration of receiving a response to a request;
- ***Bytes*** - the size of the answer;
- ***siteKey*** - the site used;
- ***htmlParser*** is the name of the one used;
- ***jmeterVersion*** - the version of Apache.JMeter used;
- ***i*** is the iteration number of the test.

**Apache.JMeter** csv logs are aggregated with **python** **pandas**:

```
	import pandas as pd
	import codecs
	from os import listdir
	import numpy as np
	
	
	# Настройки - каталог с логами и настройки считывания логов.
	dirPath = "D:/project/jmeter.htmlParser.3.0.vs.2.13/logs"
	
	read_csv_param = dict( index_col=['timeStamp'],
	                       low_memory=False,
	                       sep = ";",
	                       na_values=[' ','','null'])
	
	# Получение списка csv-файлов в каталоге с логами.
	files = filter(lambda a: '.csv' in a, listdir(dirPath))
	
	
	# Чтение содержимого всех csv-файлов в DataFrame dfs.
	csvfile = dirPath + "/" + files[0]
	print(files[0])
	dfs = pd.read_csv(csvfile,**read_csv_param)
	for csvfile in files[1:]:
	    print(csvfile)
	    tempDfs = pd.read_csv(dirPath + "/" + csvfile, **read_csv_param)
	    dfs = dfs.append(tempDfs)
	
	#dfs.to_excel(dirPath + "/total.xlsx")
	
	# Убрать из выборки все JSR223, по ним статистику строить не надо, оставить только HTTP Request Sampler.
	# У JSR223 URL пустой, у HTTP-запросов URL указан.
	dfs = dfs[(pd.isnull(dfs.URL) == False)]
	
	
	# Сводная таблица по количеству подзапросов, сохраняется в report.subrequests.html - основной результат работы.
	# Из количества запросов удаляется один запрос, чтобы исключить корневой запрос.
	# Цель данного исследования - подсчёт количества подзапросов, поэтому корневой исключается.
	pd.pivot_table(dfs, 
	               index=['siteKey', "jmeterVersion", "htmlParser"], 
	               values="URL", 
	               columns=["i"], 
	               aggfunc=lambda url: url.count()-1).to_html(dirPath + "/report.subrequest.count.html")
```

# Recursive download

![Apache.JMeter recursion care](images/yandex.ru..recursion.PNG)

As you can see:
1. Apache.JMeter finds and goes to 
   `https://yandex.ru/clck/redir/dtype=stred....7004fcb3793e79bb1ac9e&keyno=12`
2. Then finds a new unique link 
   `https://yandex.ru/clck/redir/dtype=stred....cd1c46cad58fbfe2f61&keyno=12`
3. And so it goes into recursion.

In this case, this is a picture inside the link to download Yandex Browser:

![Apache.JMeter уходит в рекурсию](images/yandex.ru..recursion.html.PNG)

And the parser finds this picture. **JMeter** tries to download it, in response he gets an html page, 
there's a link to the picture and other links again. **Apache.JMeter**'s behaviour is correct.

And in Apache.JMeter 2.13 or:
- There is a limit on the length of the links, and due to the cut-off of the unique end of the reference, 
  recursion does not occur;
- Or in **Apache.JMeter 2.13**, something doesn't work properly in parsers;
- Or in **Apache.JMeter 2.13**, something works correctly on the contrary - cookies, something else and the
  Yandex server itself responds to it so that it does not go into recursion, for example, responds with an 
  image request, not with a new html page.

I won't guess. It seems like a desperate situation. But there are no such situations. There's always a solution.

For example, you can try to specify Yandex Browser as User-Agent. Then the server probably won't show the 
picture to download the browser, or the picture will respond to the request with a picture, and there will be 
no recursion. It's a guess I didn't test it.

The script has now listed User-Agent for Google Chrome for synchronisation with webpagetest.org, and the 
server, seeing not its own browser, apparently offers a link to its own.

# Project composition

- **Jmeter.testfile.jmx** is a test script for **Apache.JMeter 2.13** and **Apache.JMeter 3.0** that accepts 
  the following parameters:
- **URL** - the address of the site being tested, for example, https://yandex.ru/;
- `siteKey` - the line by which records will be grouped in logs, for example, yandex.ru;
- `loopCount` - the number of iterations of the test, several iterations are used because websites can be unstable;
- `htmlParser.className` is a parser for extracting links to built-in resources;
- To run the script, you need to download and install an additional 
  [CsvLogWriter](https://github.com/pflb/Jmeter.Plugin.CsvLogWriter/releases) plugin.
- **Jmeter.3.0.bat** is a test startup command file for **Apache.JMeter 3.0**, where you specify the path to 
  the /bin/ Apache.JMeter 3.0 folder, the path to the jmeter.testfile.jmx test script, the test start options,
  and a list of htmlParsers whose work is
- **Jmeter.2.13.bat** is the command file to run the test for **Apache.JMeter 2.13**, where you specify the
  path to the /bin/ Apache.JMeter 2.13 folder, the path to the jmeter.testfile.jmx test script, the test start
  option, and the list of htmlParsers whose
- **Test.bat** is a command file for running the test on two versions of **Apache.JMeter**, 2.13 and 3.0, the file 
  contains the number of iterations of testing and the addresses of the testing sites. The file calls the 
  files **jmeter.2.13.bat** and **jmeter.3.0.bat**;
- **Jmeter.3.0.vs.jmeter.2.13.ipynb** is a notebook for **jupyter** for analysing **Apache.JMeter** logs;
- **Statistics.xlsx** is a table with statistics on the work of parsers, the result of the study.

# Conclusions

There is no particular practical value in the article. But some useful conclusions can be drawn:
- The parser, on average, extracts references to only a third of the resources;
- Parsers work almost the same way, which means you can use any one;
- Parsers are focussed on working with simple sites such as jmeter.apache.org;
- On sites with a lot of content, parsers work much worse than a real browser;
- The full utilisation of built-in resources in the new version of JMeter has decreased slightly, not increased;
- The application use of the CsvLogWriter plugin (see article CsvLogWriter plugin for JMeter) is demonstrated, 
  logging queries to embedded resources in the csv-log made by my colleague Alexandra @Sanchez92;
- With the help of bat files, passing JMeter paramers through the command line, logging variables and processing csv
  logs using pandas, you can test the testing tool itself; the technique has been worked out.