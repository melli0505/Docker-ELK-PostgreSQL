# Docker ELK with PostgreSQL

Monitor PostgreSQL Data Table and logging data to elasticsearch system with composable index template.

## Docker ELK

> This repository based on the Docker ELK implementation by deviantony(https://github.com/deviantony/docker-elk)

Run the latest version(8.8.1) of the Elastic stack with Docker and Docker Compose.

It gives you the ability to analyze any data set by using the searching/aggregation capabilities of Elasticsearch and the visualization power of Kibana.

Based on the official Docker images from Elastic:

- Elasticsearch
- Logstash
- Kibana

See more information -> https://github.com/deviantony/docker-elk

## Custom Settings

### User & Roles

- You can create custom elastic user & role in `setup/entrypoint.sh`, `.env` and `setup/roles/`.
- Make User

```sh
# entrypoint.sh

declare -A users_passwords
users_passwords=(
	[logstash_internal]="${LOGSTASH_INTERNAL_PASSWORD:-}"
	[kibana_system]="${KIBANA_SYSTEM_PASSWORD:-}"
	[monitoring_internal]="${MONITORING_INTERNAL_PASSWORD:-}"
  # add new user
)
```

- Make Role

```json
// setup/roles/custom_role.json example
{
  "cluster": ["manage_index_templates", "monitor", "manage_ilm"],
  "indices": [
    {
      "names": ["logs-generic-default", "logstash-*", "ecs-logstash-*"],
      "privileges": ["write", "create", "create_index", "manage", "manage_ilm"]
    },
    {
      "names": ["logstash", "ecs-logstash"],
      "privileges": ["write", "manage"]
    }
  ]
}
```

```sh
# entrypoint.sh

declare -A users_roles
users_roles=(
	[logstash_internal]='logstash_writer'
	[monitoring_internal]='remote_monitoring_collector'
  # [Username]='{role name}'
)

declare -A roles_files
roles_files=(
	[logstash_writer]='logstash_writer.json'
  # [role name] = 'role file path'
)
```

### Create Index Template

- You have to create your own Index Template which is matched with your Data Table.
- Analyzer / Tokenizer / Filter and Mapping information should be included.
- You can find composable index template examples under `logstash/templates`.

  > If your elastic version > 7.8.x, you have to use composable format index template. If not, legacy index format is okay.
  > For more information, please see documentation(https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html)

### Modify logstash pipeline

- Change `jdbc_connection_string`, `jdbc_user`, `jdbc_password`.
- If you wanna use `sql_last_run` value to prevent duplication of data, you can use `last_run_metadata_path`. It tracks last run data depending on `tracking_column` value in Data Table and save value in `last_run_metadata_path`.
- Of course, you can make your own logstash pipeline under `logstash/pipeline/`.

## Run Docker Compose

### Start Docker Compose

```sh
$ docker-compose up setup
$ docker-compose up
```

### Stop and Reset Docker Compose

```sh
$ docker-compose down -v
```
