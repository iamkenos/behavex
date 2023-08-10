# BehaveX

BehaveX is a test wrapper on top of Python Behave that provides additional capabilities to improve testing pipelines.

BehaveX can be used to build testing pipelines from scratch using the same [Behave](https://github.com/behave/behave) framework principles, or to expand the capabilities of Behave-based projects.

## Features provided by BehaveX

* Perform parallel test executions
  * Execute tests using multiple processes, either by feature or by scenario.
* Get additional test execution reports
  * Generate friendly HTML reports and JSON reports that can be exported and integrated with third-party tools
* Provide additional evidence as part of execution reports
  * Include any testing evidence by pasting it to a predefined folder path associated with each scenario. This evidence will then be automatically included as part of the HTML report
* Generate test logs per scenario
  * Any logs generated during test execution using the logging library will automatically be compiled into an individual log report for each scenario
* Mute test scenarios in build servers
  * By just adding the @MUTE tag to test scenarios, they will be executed, but they will not be part of the JUnit reports
* Generate metrics in HTML report for the executed test suite
  * Automation Rate, Pass Rate and Steps executions & duration
* Execute dry runs and see the full list of scenarios into the HTML report
  * This is enhanced implementation of Behave's dry run feature, allowing you to see the full list of scenarios in the HTML report without actually executing the tests
* Re-execute failing test scenarios
  * By just adding the @AUTORETRY tag to test scenarios, so when the first execution fails the scenario is immediately re-executed
  * Additionally, you can provide the wrapper with a list of previously failing scenarios, which will also be re-executed automatically 

![test execution report](https://github.com/hrcorval/behavex/blob/master/img/html_test_report.png?raw=true)

![test execution report](https://github.com/hrcorval/behavex/blob/master/img/html_test_report_2.png?raw=true)

![test execution report](https://github.com/hrcorval/behavex/blob/master/img/html_test_report_3.png?raw=true)


## Installing BehaveX

Execute the following command to install BehaveX with pip:

> pip install behavex

## Executing BehaveX

The execution is performed in the same way as you do when executing Behave from command line, but using the "behavex" command.

Examples:

>Run scenarios tagged as **TAG_1** but not **TAG_2**:
> 
> <pre>behavex -t @TAG_1 -t ~@TAG_2</pre>

>Run scenarios tagged as **TAG_1** or **TAG_2**:
> 
><pre>behavex -t @TAG_1,@TAG_2</pre>

>Run scenarios tagged as **TAG_1**, using **4** parallel processes:
> 
><pre>behavex -t @TAG_1 --parallel-processes 4 --parallel-scheme scenario</pre>

>Run scenarios located at "**features/features_folder_1**" and "**features/features_folder_2**" folders, using **2** parallel processes
> 
><pre>behavex features/features_folder_1 features/features_folder_2 --parallel-processes 2</pre>

>Run scenarios from "**features_folder_1/sample_feature.feature**" feature file, using **2** parallel processes
> 
><pre>behavex features_folder_1/sample_feature.feature --parallel-processes 2</pre>

>Run scenarios tagged as **TAG_1** from "**features_folder_1/sample_feature.feature**" feature file, using **2** parallel processes
> 
><pre>behavex features_folder_1/sample_feature.feature -t @TAG_1 --parallel-processes 2</pre>

>Run scenarios located at "**features/feature_1**" and "**features/feature_2**" folders, using **2** parallel processes
> 
><pre>behavex features/feature_1 features/feature_2 --parallel-processes 2</pre>

>Run scenarios tagged as **TAG_1**, using **5** parallel processes executing a feature on each process:
> 
><pre>behavex -t @TAG_1 --parallel-processes 5 --parallel-scheme feature</pre>

>Perform a dry run of the scenarios tagged as **TAG_1**, and generate the HTML report:
> 
><pre>behavex -t @TAG_1 --dry-run</pre>


## Constraints

* BehaveX is currently implemented on top of Behave **v1.2.6**, and not all Behave arguments are yet supported.
* The parallel execution implementation is based on concurrent Behave processes. Therefore, any code in the **before_all** and **after_all** hooks in the **environment.py** module will be executed in each parallel process. The same applies to the **before_feature** and **after_feature** hooks when the parallel execution is set by scenario.
* The library is provided as is, and no tests have been implemented for the framework yet (initial versions had tests, but they were deprecated). Any contributions to testing would be greatly appreciated.

### Additional Comments

* The **stop** argument does not work when performing parallel test executions.
* The JUnit reports have been replaced by the ones generated by the test wrapper, just to support muting tests

## Supported Behave arguments
The following Behave arguments are currently supported:
* no_color
* color
* define
* exclude
* include
* no_snippets
* no_capture
* name
* capture
* no_capture_stderr
* capture_stderr
* no_logcapture
* logcapture
* logging_level
* summary
* quiet
* stop
* tags
* tags-help

There might be more arguments that can be supported, it is just a matter of adapting the wrapper implementation to use these.

## Additional BehaveX arguments
* output_folder
  * Specifies the output folder where execution reports will be generated (JUnit, HTML and JSon)
* dry-run
  * Overwrites the existing Behave dry-run implementation
  * Performs a dry-run by listing the scenarios as part of the output reports
* parallel_processes
  * Specifies the number of parallel Behave processes
* parallel_scheme
  * Performs the parallel test execution by [scenario|feature]

## Parallel test executions
The implementation for running tests in parallel is based on concurrent executions of Behave instances in multiple processes.

As mentioned as part of the wrapper constraints, this approach implies that whatever you have in the Python Behave hooks in **environment.py** module, it will be re-executed on every parallel process.

BehaveX will be in charge of managing each parallel process, and consolidate all the information into the execution reports

Parallel test executions can be performed by **feature** or by **scenario**.

Examples:
> behavex --parallel-processes 3

> behavex -t @\<TAG\> --parallel-processes 3

> behavex -t @\<TAG\> --parallel-processes 2 --parallel-scheme scenario

> behavex -t @\<TAG\> --parallel-processes 5 --parallel-scheme feature

When the parallel-scheme is set by **feature**, all tests within each feature will be run sequentially.

## Test execution reports
### HTML report
This is a friendly test execution report that contains information related to test scenarios, execution status, execution evidence and metrics. A filters bar is also provided to filter scenarios by name, tag or status.

It should be available at the following path:
> <output_folder>/report.html


### JSON report
Contains information about test scenarios and execution status.

It should be available at the following path:
> <output_folder>/report.json

The report is provided to simplify the integration with third party tools, by providing all test execution data in a format that can be easily parsed.

### JUnit report

The wrapper overwrites the existing Behave JUnit reports, just to enable dealing with parallel executions and muted test scenarios

By default, there will be one JUnit file per feature, unless the parallel execution is performed by scenario, in which there will be one JUnit file per scenario.

Reports are available at the following path:
> <output_folder>/behave/*.xml

## Attaching additional execution evidence to test report

It is considered a good practice to provide as much as evidence as possible in test executions reports to properly identify the root cause of issues.

Any evidence file you generate when executing a test scenario, it can be stored into a folder path that the wrapper provides for each scenario.

The evidence folder path is automatically generated and stored into the **"context.evidence_path"** context variable. This variable is automatically updated by the wrapper before executing each scenario, and all the files you copy into that path will be accessible from the HTML report linked to the executed scenario

## Test logs per scenario

The HTML report provides test execution logs per scenario. Everything that is being logged using the **logging** library will be written into a test execution log file linked to the test scenario.

## Metrics

There are a few metrics that can be easily calculated and reported:
* Automation Rate
* Pass Rate
* Steps execution counter and average execution time

All metrics are provided as part of the HTML report

## Dry runs

The wrapper overwrites the exiting Behave dry run implementation just to be able to provide the outputs into the wrapper reports.

The HTML report generated as part of the dry run can be used to share the scenarios specifications with any stakeholder.

Example:
> behavex -t @TAG --dry-run

## Muting test scenarios

Sometimes it is necessary that failing test scenarios continue being executed in all build server plans, but having them muted until the test or product fix is provided.

Tests can be muted by adding the @MUTE tag to each test scenario. This will cause the wrapper to run the test but the execution will not be notified in the JUnit reports. However, you will see the execution information in the HTML report.

## What to do with failing scenarios?

### @AUTORETRY tag

This tag can be used for flaky scenarios or when the testing infrastructure is not stable at all.

The @AUTORETRY tag can be applied to any scenario or feature, and it is used to automatically re-execute the test scenario when it fails. 

### Rerun all failed scenarios

Whenever you perform an automated test execution and there are failing scenarios, the **failing_scenarios.txt** file will be created into the execution output folder.
This file allows you to run all failing scenarios again. 

This can be done by executing the following command:

> behavex -rf ./<OUTPUT_FOLDER>/failing_scenarios.txt

or

> behavex --rerun-failures ./<OUTPUT_FOLDER>/failing_scenarios.txt

To avoid the re-execution to overwrite the previous test report, we suggest to provide a different output folder, using the **-o** or **--output-folder** argument.

It is important to mention that this argument doesn't work yet with parallel test executions 

## Show Your Support

**If you find this project helpful or interesting, we would appreciate it if you could give it a star** (:star:). It's a simple way to show your support and let us know that you find value in our work.

By starring this repository, you help us gain visibility among other developers and contributors. It also serves as motivation for us to continue improving and maintaining this project.

Thank you in advance for your support! We truly appreciate it.
 
