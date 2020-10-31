---
title: Useful CLI Commands
date: 2020-10-31T23:15
---

Following are some CLI commands I find useful and need from time to time but not enough to just remember them.

All of these commands work in my current environment. Some might need dependencies while others should work on most UNIX based systems.

## Search/Replace in Multiple Files

You can use `find` with the `exec` option to do this. The following will find and replace a string in all HTML files within the working directory.

```shell{outputLines: 2, 3}
find . -name '*.html' -exec sed -i -e 's#http://localhost:8008#https://rafhun.github.io/blog#g' {} \;

# Adaption I personally need from sometimes
find . -name '*.html' -exec sed -i -e 's#http://localhost:8080#https://rafhun.github.io/blog#g' {} \; -exec sed -i -e 's#main.js#main_29af687e.js#g' {} \; -exec sed -i -e 's#main.css#main_29af687e.css#g' {} \;
```

Remember that you can use whatever separator character you want within the `sed` function. Here we chose `#` since it does not show up in the search strings we apply.

## MySQL Import Into DB in Docker

Usually necessary when using WP Migrate DB to mirror the current online DB to the local development environment. The command assumes a gzipped myslqdump as import source.

```shell
gzcat dump.sql.gz | docker exec -i container_db_1 mysql -u"username" -p"password" "db_name"
```

## Nameserver Lookup

If you want to check a domains A-Records use the following.

```shell{outputLines: 2-8}
nslookup example.com

Server:         192.168.0.1
Address:        192.168.0.1#53

Non-authoritative answer:
Name:   example.com
Address: 93.184.216.34
```
