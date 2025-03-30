# Doakyz Lancache logstash docker-compsoe setup

Forked from :https://github.com/hauslan/logstash


This fork assumes that you'll use docker compose to run logstash; while not required, changes have been made to use the lancache domains list here: https://github.com/doakyz/domains

Changes included
* Changed folder structure, moving all required files and folders to the root directory
* Minor change to the kibana.yml file to remove server.publicBaseUrl warnings when running
* Changes to logstash.conf to reflect doakyz/domains


## Making changes
To add additional domain groups to the graphing `logstash.conf` must be edidted.


This is done be adding a new else if statement after the existing set, but before the UNKNOWN block

Example of the Arch Linux block in `logstash.conf` 
```bash
        # Arch Linux
        } else if [source] =~ "arch" or [log][file][path] =~ "arch" or [cache_tag] == "arch" {
            mutate { replace => { upstream => "arch" } }
```


The source has to match what is being loaded by lancache in its `cache_domains.json` file

```bash
		{
			"name": "arch",
			"description": "Arch Linux based repositories",
			"domain_files": ["arch.txt"]
		}
```

Due to the nature of how the containers work, it's sometimes required to completely remove all of the container volumes and rerun the dashboard import. Otherwise the fields will be treated as UNKOWN

## It's woth keeping the original README
## Example of ELK stack with Filebeat for collecting stats from LanCache

This is an example set up of the ELK stack with a filebeat instance for reading the logs from LanCache. This should be a very quick way to get a working monitoring stack however should not be used in a production enviroment!

Please make sure to change the password of the default elastic user, this can done by searching for "changeme" and replacing. 

This assumes you already have a running LanCache setup, for instructions on how to do that please see the [LanCache website](https://lancache.net/docs).

# Quickstart

1. Clone this repository
2. Update the required .env fields in the such as `ELASTIC_PASSWORD`, `IMPORT_DASHBOARD_ONLY_ONCE` & `CACHE_ROOT`,
3. Execute `docker-compose up -d`
4. Visit hostname:5601 (hostname will be localhost if running locally), then login with elastic and the password you changed.
5. You can now view the dashboards through the left sidebar, it might take some time for all of the data to be pulled into the ELK stack


---
## Things to note

* Keep an eye on ElasticSearch memory usage, production clusters of ElasticSearch have a minimum recommendation of 16GB of memory, we have in this case only assigned 1G, you may need to increase this for your setup. You can do this through the `ES_JAVA_OPTS` variable inside the docker compose file. You can find Elastics architecture guidance here: https://www.elastic.co/guide/en/elasticsearch/guide/current/hardware.html
* This uses the logstash.conf from the root directory, you may want to change this, or change the file entirely.
* By default, the default dashboards will only be uploaded to Kibana once, this is configured through the `IMPORT_DASHBOARD_ONLY_ONCE` env, `true`, `false` & `disabled` are valid options here.
> * true = Will run 1 time (makes a file in `./dashboard-importer/tmp/import_done`)
>   * If you wish to have it run again, you can delete this file on the host & re-run `docker-compose up -d`.
> * false = Will run every time the `dashboard-importer` container is ran
> * disabled = Will never import (the container will still boot, if you want to stop this, you can remove the `dashboard-importer` service from your `docker-compose.yml`)




## Monocache Logstash Config

This config is used to interpret monocache logs in elastic search. It defines a filebeat endpoint through logstash and the groks required to separate out key game infomation for some of the CDNs. Also included is an export of our dashboards and visualisations to display results.

#### Configuring your logstash.conf if not using the example deployment

In the output section of the logstash.conf file you will need to change the hosts array to match the names of your elastic search hosts, and you might need to update the user/password rows:

    hosts    => [ 'elasticsearch' ] 
    user => 'elastic'
    password => '${ELASTIC_PASSWORD}'

### Reference

These configs are provided to help give you a start for your elastic instance. We will be happy to help if we can but we do not intend to provide full support on your elk stack.

This was originally created following advice from [ilumos](https://github.com/ilumos) at [zeroping heros](https://github.com/zeropingheroes/lancache-elk). The [example deployment](https://github.com/lancachenet/logstash/tree/master/example) uses the previously mentioned config, which has been further developed on by [Ewan](https://github.com/ewancolyer) & [Rushmead](https://github.com/rushmead) at [Hauslan](https://github.com/hauslan) to create a quickstart example.