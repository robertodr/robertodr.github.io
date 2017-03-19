---
title: Continuous integration jobs on Travis using Docker
published: March 15, 2017
excerpt: Using Docker on Travis CI
tags: GitHub, Docker, Travis, CI
---

* toc


## Repository

https://github.com/robertodr/travis-docker-example

## Commands

```
docker pull gcc:5.4.0
docker run -v (pwd):(pwd) -w (pwd) -p 127.0.0.1:80:4567 gcc:5.4.0 /bin/sh -c "bash ./fuffa.sh"
```

