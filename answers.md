// Command line run command to assign tags to host using Datadog agent
msiexec /qn /i datadog-agent-6-latest.amd64.msi APIKEY="f65e8eadc68633a9188f1d200345a78e" TAGS="env:env_east"

//mycheck.py
import random

try:
    from checks import AgentCheck
except ImportError:
    from datadog_checks.checks import AgentCheck

__version__ = "1.0.0"

class HelloCheck(AgentCheck):
    def check(self, instance):
        self.gauge('my_metric', random.randint(0,1000))
        
//mycheck.yaml
init_config:

instances:
  - min_collection_interval: 45


//Script to define a Timeboard on Datadog and display 3 graphs:
//Timeboard.py

from datadog import initialize,api
options = {
    'api_key': 'f65e8eadc68633a9188f1d200345a78e',
    'app_key': 'ff482d358d44daf6c951012f7171f696e813034e'
}

initialize(**options)

title = "My Timeboard"
description = "An Sample Timeboard."
graphs = [{
    "definition": {
        "events": [],
        "requests": [
            {"q": "avg:my_metric{host:DESKTOP-4KF85Q1}"}
        ],
        "viz": "timeseries"
    },
    "title": "my_metric across my host"
},
{
    "definition": {
        "events": [],
        "requests": [
            {"q": "anomalies(avg:postgresql.disk_read{*},'basic',3)"}
        ],
        "viz": "timeseries"
    },
    "title": "Postgres anomalies"
},
{
    "definition": {
        "events": [],
        "requests": [
            {"q": "avg:my_metric{host:DESKTOP-4KF85Q1}.rollup(sum,3600)"}
        ],
        "viz": "timeseries"
    },"title": "my_metric roll up sum"
}
]

template_variables = [{
    "name": "DESKTOP-4KF85Q1",
    "prefix": "host"
}]

read_only = True
api.Timeboard.create(title=title,
                     description=description,
                     graphs=graphs,
                     template_variables=template_variables,
                     read_only=read_only)
