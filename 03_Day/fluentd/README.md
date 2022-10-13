# Fluentd Simplified
-----
### geoip plugin 실행 방법

  docker run -u root -ti --rm -v /home/me/fluentd/etc:/fluentd/etc -v /home/me/fluentd/log:/var/log/ -v /home/me/fluentd/output:/output fluent/fluentd:v1.10-debian-1 bash -c "apt update && apt install -y build-essential libgeoip-dev libmaxminddb-dev && gem install fluent-plugin-rewrite-tag-filter fluent-plugin-geoip && fluentd -c /fluentd/etc/fluentd.conf -v"
-----
This repo is used as the source code of [this](https://scaleout.ninja/post/fluentd-simplified/) post.

## Run Fluentd
```bash
docker run -ti --rm \
-v $(pwd)/etc:/fluentd/etc \
-v $(pwd)/log:/var/log/ \
-v $(pwd)/output:/output \
fluent/fluentd:v1.10-debian-1 -c /fluentd/etc/fluentd.conf -v
```

## Run Fluentd with plugin
```bash
docker run -u root -ti --rm \
-v $(pwd)/etc:/fluentd/etc \
-v $(pwd)/log:/var/log/ \
-v $(pwd)/output:/output \
fluent/fluentd:v1.10-debian-1 bash -c "gem install fluent-plugin-rewrite-tag-filter && fluentd -c /fluentd/etc/fluentd.conf -v"
```
