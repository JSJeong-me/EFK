https://docs.fluentd.org/configuration/config-file


### Config File Location

    $ sudo vi /etc/td-agent/td-agent.conf


### List of Directives

    1. source directives determine the input sources
    2. match directives determine the output destinations
    3. filter directives determine the event processing pipelines
    4. system directives set system-wide configuration
    5. label directives group the output and filter for internal routing
    6. worker directives limit to the specific workers
    7. @include directives include other files