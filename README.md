[![Requirements Status](https://requires.io/github/dhondta/malicious-macro-tester/requirements.svg?branch=master)](https://requires.io/github/dhondta/malicious-macro-tester/requirements/?branch=master)
[![Known Vulnerabilities](https://snyk.io/test/github/dhondta/malicious-macro-tester/badge.svg?targetFile=requirements.txt)](https://snyk.io/test/github/dhondta/malicious-macro-tester?targetFile=requirements.txt)


## Table of Contents

   * [Introduction](#introduction)
   * [System Requirements](#system-requirements)
   * [Installation](#installation)
   * [Quick Start](#quick-start)
   * [Issues management](#issues-management)


## Introduction

This CLI tool automates the classification of Office documents with macros using MaliciousMacroBot. It allows to analyze a folder of sample files and to generate a report in multiple output formats.


## System Requirements

This tool was tested on an Ubuntu 16.04 with Python 2.7.

It uses the following libraries:
- `coloredlogs` (because colored logs are better...)
- `elasticsearch` (required if exporting results to ElasticSearch)
- `markdown2` (required if using report generation)
- [`mmbot`](https://github.com/egaus/MaliciousMacroBot) (required for classification)
- `pandas` (required for parsing MaliciousMacroBot results)
- `tinyscript` (required)
- `weasyprint` (required if using PDF report format)
- `xmltodict` (required if using XML report format)


## Installation

1. Clone this repository

 ```session
 $ git clone https://github.com/dhondta/malicious-macro-tester.git
 ```
 
 > **Behind a proxy ?**
 > 
 > Setting: `git config --global http.proxy http://[user]:[pwd]@[host]:[port]`
 > 
 > Unsetting: `git config --global --unset http.proxy`
 > 
 > Getting: `git config --global --get http.proxy`

2. Install Python requirements

 ```session
 $ sudo pip install -r requirements.txt
 ```

 > **Behind a proxy ?**
 > 
 > Do not forget to add option `--proxy=http://[user]:[pwd]@[host]:[port]` to your pip command.
 
3. [Facultative] Copy the Python script to your `bin` folder

 ```session
 $ chmod a+x malicious-macro-tester.py
 $ sudo cp malicious-macro-tester.py /usr/bin/malicious-macro-tester
 ```


## Quick Start

1. Help

 ```session
  $ python malicious-macro-tester.py -h
  usage: malicious-macro-tester [-h] [-d] [-f] [-l] [-q] [-r] [-s] [-u]
                                [--api-key VT_KEY]
                                [--output {es,html,json,md,pdf,xml}] [--send]
                                [-v]
                                FOLDER

  MaliciousMacroTester v2.4
  Author: Alexandre D'Hondt
  Reference: INFOM444 - Machine Learning - Hot Topic

  This tool uses MaliciousMacroBot to classify a list of samples as benign or
   malicious and provides a report. Note that it only works on an input folder
   and list every file to run it against mmbot.

  positional arguments:
    FOLDER                folder with the samples to be tested OR
                          pickle name if results are loaded with -l

  optional arguments:
    -h, --help            show this help message and exit
    -d                    dump the VBA macros (default: false)
    -f                    filter only DOC and XLS files (default: false)
    -l                    load previous pickled results (default: false)
    -q                    do not display results report (default: false)
    -r                    when loading pickle, retry VirusTotal hashes with None results
                           (default: false)
    -s                    pickle results to a file (default: false)
    -u                    when loading pickle, update VirusTotal results (default: false)
    --api-key VT_KEY      VirusTotal API key (default: none)
                            NB: key as a string or file path to the key
    --output {es,html,json,md,pdf,xml}
                          report file format (default: none)
    --send                send the data to ElasticSearch (default: false)
                            NB: only applies to 'es' format
                               the configuration is loaded with the following precedence:
                               1. ./elasticsearch.conf
                               2. /etc/elasticsearch/elasticsearch.conf
    -v                    debug verbose level (default: false)

  Usage examples:
    python malicious-macro-tester.py my_samples_folder
    python malicious-macro-tester.py my_samples_folder --api-key virustotal-key.txt -lr
    python malicious-macro-tester.py my_samples_folder -lsrv --api-key 098fa24...be724a0
    python malicious-macro-tester.py my_samples_folder -lf --output pdf
    python malicious-macro-tester.py my_samples_folder --output es --sent
   
 ```
 
2. Examples of output

 ```session
  $ python malicious-macro-tester.py samples -vfqs --output xml
  17:08:09 [INFO] Instantiating and initializing MaliciousMacroBot...
  17:09:09 [INFO] Processing samples...
  17:09:09 [DEBUG] MMBot: classifying 'file_003.xls'...
  17:09:09 [DEBUG] MMBot: classifying 'file_001.doc'...
  17:09:09 [DEBUG] MMBot: classifying 'file_005.xls'...
  17:09:09 [DEBUG] MMBot: classifying 'file_000.doc'...
  17:09:09 [DEBUG] MMBot: classifying 'file_004.xls'...
  17:09:10 [DEBUG] MMBot: classifying 'file_002.doc'...
  17:09:10 [INFO] Saving results to pickle...
  17:09:10 [INFO] Parsing results...
  17:09:10 [DEBUG] Generating the JSON report (text only)...
  17:09:10 [DEBUG] Generating the XML report...
 ```
 
 This will generate `report.xml`, as shown in the [`examples`](examples) folder, and save the pickled results.


 ```session
  $ python malicious-macro-tester.py samples -vfql --output pdf
  17:11:17 [INFO] Loading previous results from pickle...
  17:11:17 [INFO] Processing samples...
  17:11:17 [INFO] Parsing results...
  17:11:17 [DEBUG] Generating the Markdown report (text only)...
  17:11:17 [DEBUG] Generating the HTML report (text only)...
  17:11:17 [DEBUG] Generating the PDF report...
 ```
 
 This will generate `report.pdf`, as shown in the [`examples`](examples) folder.
 
 ```session
  $ python malicious-macro-tester.py subsamples -vfqls --output html --api-key virustotal-key.txt 
  17:18:22 [INFO] Loading previous results from pickle...
  17:18:22 [DEBUG] Testing VirusTotal API...
  17:18:22 [INFO] Processing samples...
  17:18:22 [DEBUG] > Getting VT information (file_005.xls)...
  17:18:23 [DEBUG] > Getting VT information (file_003.xls)...
  17:18:23 [DEBUG] > Getting VT information (file_004.xls)...
  17:19:23 [DEBUG] > Getting VT information (file_000.doc)...
  17:19:23 [DEBUG] > Getting VT information (file_002.doc)...
  17:19:23 [DEBUG] > Getting VT information (file_001.doc)...
  17:19:24 [WARNING] VT lookup failed for '...'
  17:19:24 [INFO] Saving results to pickle...
  17:19:24 [INFO] Parsing results...
  17:19:24 [DEBUG] Generating the Markdown report (text only)...
  17:19:24 [DEBUG] Generating the HTML report...
 ```
 
 This will load previous pickled results, generate `report.html`, as shown in the [`examples`](examples) folder, and resave the pickled results updated with the information from VirusTotal.
 
 ```session
  $ python malicious-macro-tester.py subsamples -vfql --output json --api-key virustotal-key.txt 
  17:21:33 [INFO] Loading previous results from pickle...
  17:21:33 [DEBUG] Testing VirusTotal API...
  17:21:34 [INFO] Processing samples...
  17:21:34 [INFO] Parsing results...
  17:21:34 [DEBUG] Generating the JSON report...
 ```
 
 This will load previous pickled results and generate `report.json`, as shown in the [`examples`](examples) folder.

## Issues management

Please [open an Issue](https://github.com/dhondta/malicious-macro-tester/issues/new) if you want to contribute or submit suggestions.
