
# PHP-FPM Active Process Counting Issue with fastcgi_keep_conn

## Problem Description

This repository demonstrates a bug in PHP-FPM active process counting that occurs with a specific nginx and PHP-FPM configuration. The issue involves incorrect counting of active processes when the `fastcgi_keep_conn on` option is enabled in nginx configuration.

## Bug Details

- **Cause**: `fastcgi_keep_conn on` parameter in nginx configuration
- **Effect**: Incorrect counting of active processes in PHP-FPM status
- **Scope**: This bug has been present for a long time across different PHP versions

## Minimal Configuration Reproducing the Bug

### Requirements
- Docker
- Docker Compose

### Running
```shell
docker-compose up -d
```

After starting the containers, PHP-FPM status will be available at [http://localhost:8080/status](http://localhost:8080/status)


## Reproducing the Bug

1. Start containers: `docker-compose up -d`
2. Observe incorrect active process counting: `watch -n 1 curl -s http://localhost:8080/status`
3. Run some benchmark tool `wrk -c 150 http://127.0.0.1:8080/status`
4. Look for insane total processes numbers

```text
pool: www
process manager: static
start time: 03/Jun/2025:13:11:48 +0000
start since: 23914
accepted conn: 158110
listen queue: 0
max listen queue: 20
listen queue len: 4096
idle processes: 0
active processes: 698 <-------
total processes: 698 <------- 
max active processes: 1597 <-------
max children reached: 0
slow requests: 0
memory peak: 14680064
```

## HOT-FIX Solution

To hit-fix the issue, remove or comment out the line:
```
# fastcgi_keep_conn on;
```

After this change, active process counting will return to normal.

## Testing Notes

- Default nginx and PHP-FPM images from dockerhub **do not** reproduce this issue
- The problem only occurs with this specific configuration combination
- The bug was tested across different PHP versions and has existed for a long time

## Remarks

This issue demonstrates how specific configuration combinations can lead to unexpected application behavior bugs.