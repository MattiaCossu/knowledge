**ELASTICSEARH** 
ElasticSearch is a distibuted, open source search and analytics engine for all type of data:
* textual 
* numerical
* geospatial
* structured
* unstructured

Elasticsearch is the central component of the Elastic Stack, a set of open source tools for:
* data ingestion
* enrichment
* storage
* analysis
* visualization

[Source:](https://www.elastic.co/what-is/elasticsearch)

**ElastAlert**
It works by combining Elasticsearch with two types of components, rule types and alerts.
Elasticsearch is periodically queried and the data is passed to the rule type, which
determines when a match is found. When a match occurs, it is given to one or more
alerts, which take action based on the match.
This is congured by a set of rules, each of which denes a query, a rule type, and a set
of alerts.

Several rule types with common monitoring paradigms are included with ElastAlert:
* “Match where there are X events in Y time” (**frequency type**)
* “Match when the rate of events increases or decreases” (**spike type**)
* “Match when there are less than X events in Y time” (**flatline type**)
* “Match when a certain eld matches a blacklist/whitelist” (**blacklist and whitelist type**)
* “Match on any event matching a given filter” (**any type**)
* “Match when a field has two dierent values within some time” (**change type**)

[Source:](https://www.elastalert.readthedocs.io)

## Install ElastAlert
### Step 1: install

In order to do so, run the following command:
```bash
python3 -m pip install elastalert
```

For my expirence we need to install setuptools_rust with
```bash
python3 -m pip install setuptools_rust
```

If no error occured, you should be able to test ElastAlert using the following command:
```bash
python3 -m elastalert.elastalert --help
```

### Step 2: setup

Before we create our cong le, let's set up a target directory where we'll save our work.
```bash
mkdir -p ~/elastalert/rules
```

ElastAlert keeps track of its searches by writing data to a separate index on the Elastic stack it
queries. This separate index hasn't been created yet, as the command requires a cong le
rst.
Create the cong le using a text editor:
```bash
nano ~/elastalert/config.yml
```

Simple config file (example)
You can watch file in root of this directory 

We are now ready to create the index by running:
```bash
python3 -m elastalert.create_index --index elastalert_status --config ~/elastalert/config.yml
```

In my case characters /t geneerate an error in yml file, for resolve remove /t characters (tab)

Final output if all works
```bash
ansible@soc:~/elastalert$ python3 -m elastalert.create_index --index elastalert_status --config ~/elastalert/config.yml Elastic Version: 7.10.0                                                                                                 Reading Elastic 6 index mappings:                                                                                       Reading index mapping 'es_mappings/6/silence.json'                                                                      Reading index mapping 'es_mappings/6/elastalert_status.json'                                                            Reading index mapping 'es_mappings/6/elastalert.json'                                                                   Reading index mapping 'es_mappings/6/past_elastalert.json'                                                              Reading index mapping 'es_mappings/6/elastalert_error.json'                                                             New index elastalert_status created                                                                                     Done!
```

**Sigma**
Sigma is a generic and open signature format that allows you to describe relevant log
events in a straight forward manner. The rule format is very exible, easy to write and
applicable to any type of log le. The main purpose of this project is to provide a
structured form in which researchers or analysts can describe their once developed
detection methods and make them shareable with others.

Sigma is for log les what **Snort** is for network trac and **YARA** is for file

[Source:](https://github.com/SigmaHQ/sigma)

## Transforming our Sigma rule into a working ElastAlert rule along with a TheHive integration
For this case i use sigmac tool (sigma roles convert).
Additional documentation on this tool can be found at https://github.com/Neo23x0/sigma/wiki/Converter-Tool-Sigmac.

### Step 1: prepering sigma role 
For convenience' we would separate the sigma rules from those of elastalert to create a new directory 
```bash
mkdir ~/custom_rules
cd ~/custom_rules
```

I will now create in this folder a rule that will go to verify the accesses of a particular user on the RDP service 
```yml
title: 699 Test rule
description: Detect RDP logins
tags:
- Test
status: experimental
author: sec699
logsource:
product: windows
service: security
definition: 'Test rule to detect the existence of RDP logins'
detection:
selection:
EventID: 4624
LogonType: 10
condition: selection
falsepositives:
- everything
level: low
```

### Step 2: Generate a Successfull RDP Login 
To open a new RDP session, start by searching the start-menu for "Remote Desktop Connection"
on Windows or "Remmina" on Ubuntu Linux

### Step 3: Manually Verifying the Logos in **KIBANA**
Let's manually verify if any events for this remote desktop session exist. Open a browser and navigate to your kibana interface

Click the "Discover" icon in the left pane.

with this filter 
```
winlog.event_id: 4624 AND winlog.event_data.LogonType: 10
```
we can watch a hit for this event 

### Step 4: Mapping Sigma to Elastic
I want to rember what it is Sigma, is a freamwork for write rules 

If we retake part of our Sigma rule we see the eld names EventID and LogonType:
```yml
detection:
	selection:
		EventID: 4624
		LogonType: 10
```
if we go back to Kibana and analyze the event's structure, we notice that the eld names need
to be mapped. Sigma's EventID should map to Elasticsearch's winlog.event_id , LogonType
to winlog.event_data.LogonType , and so on...

Luckily, modern Elastic instances and tools are ECS compliant
The main advantage of this standardized naming schema is that Sigma provides an already-
prepared mapping for the ECS, which you can find in their repository at
~/sigma/tools/config/winlogbeat-modules-enabled.yml.

### Step 5: Converting Sigma to Elastalert
We will now convert our Sigma rule using the mapping file:
```bash
sigmac --target elastalert --config ~/sigma/tools/config/winlogbeat-modules-enabled.yml
--output ~/elastalert/rules/test.yml ~/custom_rules/sec699-test.yml
```

Review the newly created rule:
```bash
cat ~/elastalert/rules/test.yml
```
Output:
```yaml
alert:
- debug
description: Detect RDP logins
filter:
- query:
	query_string:
		query: (winlog.channel:"Security" AND winlog.event_id:"4624" AND
winlog.event_data.LogonType:"10")
index: winlogbeat-*
name: 699-Test-rule_0
priority: 4
realert:
	minutes: 0
type: any
```

Before:
```yaml
title: 699 Test rule
description: Detect RDP logins
tags:
  - Test
status: experimental
author: sec699
logsource:
  product: windows
  service: security
  definition: 'Test rule to detect the existence of RDP logins'
detection:
  selection:
    EventID: 4624
    LogonType: 10
  condition: selection
falsepositives:
  - everything
level: low
```

Let's test our rule by using the following command:
```bash
python3 -m elastalert.elastalert --config ~/elastalert/config.yml --rule
~/elastalert/rules/test.yml --verbose --start $(date +"%Y-%m-%d")
```

The nal part of the command forces ElastAlert to start querying for data that has been
ingested today. By default it will start querying from "now". When ElastAlert completed its first run you should receive the message that some alerts were sent:
```
INFO:elastalert:Ran 699-Test-rule_0 from 2020-03-12 00:00 GMT to 2020-03-12 15:45 GMT:
0 query hits (0 already seen), 2 matches, 2 alerts sent
```

### Step 6: Integrated with TheHive
At the moment we have a working ElastAlert rule, but no place to send our alert to. We'll
update our rule to automatically send alerts to TheHive.

**TheHive**
TheHive is a scalable 4-in-1 open source and free security incident response platform
designed to make life easier for SOCs, CSIRTs, CERTs and any information security
practitioner dealing with security incidents that need to be investigated and acted uponswiftly. Thanks to Cortex, our powerful free and open source analysis engine, you can
analyze (and triage) observables at scale using more than 100 analyzers.

[Source](github.com/TheHive-Project)

Once installed and configured we need to make the user alert creator and take its api key

After open your ElastAlert rule with a text editor.
```bash
nano ~/elastalert/rules/sec699-test.yml
```
We'll update the alert section in our rule to have ElastAlert generate alerts in TheHive.
Update our existing rule with the following conguration:
```
alert:
	- 'hivealerter'

hive_connection:
	hive_host: http://192.168.20.106
	hive_port: 9000
	hive_apikey: lAI6EcfZSc4sZmRj6TVXR2zxkoawDXgf # Replace the API key with yours

hive_alert_config:
	title: 'RDP logins detected on host {match[host][name]}'
	type: 'ElastAlert'
	source: '{match[event][provider]}'
	description: 'Detected by {rule[name]}.'
	severity: 2
	tags: ['Sigma', 'Test']
	tlp: 2
	status: 'New'
	follow: True
```

We replaced the default debug alert with a hivealerter . In order for the alerter to work
properly it requires a few mandatory variables:

- hive_host is the IP or hostname where your TheHive instance can be found. This is the
IP of your SOC stack
- hive_port is TheHive's port, which is the default 9000.
- hive_apikey is the API key you previously generated in TheHive.
- hive_alert_config let us customize which info is sent to TheHive. We can substitute certain values by referencing them between {} . The values can be derived from the rule that was triggered, such as '{rule[name]}' or from the event that was matched by the rule. Any field can be used, if we take the hostname as an example we can have a quick
peak at the Kibana event and notice there is a eld called host.name. We can reference this eld through match[host][name].
- source lets you track where your alert was generated from. We will use Kibana's event.provider eld which, for our RDP events, contain the "Microsoft-Windows-Security-Auditing" value.

Below is the full rule for good measurement.
```
alert:
	- 'hivealerter'

hive_connection:
	hive_host: http://192.168.20.106
	hive_port: 9000
	hive_apikey: lAI6EcfZSc4sZmRj6TVXR2zxkoawDXgf # Replace the API key with yours

hive_alert_config:
	title: 'RDP logins detected on host {match[host][name]}'
	type: 'ElastAlert'
	source: '{match[event][provider]}'
	description: 'Detected by {rule[name]}.'
	severity: 2
	tags: ['Sigma', 'Test']
	tlp: 2
	status: 'New'
	follow: True

description: Detect RDP logins
filter:
- query:
	query_string:
		query: (winlog.channel:"Security" AND winlog.event_id:"4624" AND
winlog.event_data.LogonType:"10")
index: winlogbeat-*
name: 699-Test-rule_0
priority: 4
realert:
	minutes: 0
type: any
```

Our integration is ready to be tested, run ElastAlert once more:
```bash
python3 -m elastalert.elastalert --config ~/elastalert/config.yml --rule
~/elastalert/rules/test.yml --verbose --start $(date +"%Y-%m-%d")
```