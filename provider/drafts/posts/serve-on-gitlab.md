---
title: Serving a static website on GitLab
published: March 13, 2017
excerpt: Static website on GitLab
tags: GitLab, Hakyll
---

* toc

Serving a static website on [GitLab] is pretty easy.
Go to [GitLab Pages] and follow the instructions. There are a number of template projects, each using a
different static site generator. The one of my choice is [Hakyll].
I have been using Hakyll for a while for this website and for some other projects.
On GitLab, pages are updated and served _via_ CI jobs, run using [GitLab CI].
For the websites served using GitLab, I have been using the domain names that are automatically assigned.
I wanted to find out how to use a custom domain name pointing to content hosted on GitLab's servers.

## Serving the website

To start the process, I just forked the [example Hakyll site]. This is a great place to start, since
all the deployment configuration is already done.
A couple of weeks ago, however, the CI job described in `.gitlab-ci.yml` started failing due
to missing dependencies. Together with the fact that installing all dependencies ballooned the
update times to ~40 minutes, I decided to create my own [Docker image] [^1] and run the CI job on
my droplet on [DigitalOcean]. [^2]
The `.gitlab-ci.yml` file looks like this:
``` {lang="yml" text="GitLab CI and CD job"}
image: registry.gitlab.com/flaks/flaks.gitlab.io:latest

pages:
  cache:
    paths:
      - _cache
      - .stack
  before_script:
    - stack install --only-dependencies
    - stack build
  script:
    - stack exec site build
  artifacts:
    paths:
      - public
  only:
    - master
```
Since the Docker image has [LTS Haskell] 8.3 baked in, the Cabal file and
`stack.yaml` have to be configured accordingly.

## Redirect & HTTPS

Once the content is up and you have bought your fancy domain name, just follow the [instructions]
to configure the redirect.
[GitLab] + [Namecheap] + [Cloudflare]

[Useful Gist]

[GitLab]: https://gitlab.com/
[GitLab Pages]: https://pages.gitlab.io/
[Hakyll]: https://jaspervdj.be/hakyll/
[GitLab CI]: https://about.gitlab.com/gitlab-ci/
[example Hakyll site]: https://gitlab.com/pages/hakyll
[Docker image]: https://www.docker.com/
[DigitalOcean]: https://www.digitalocean.com/
[LTS Haskell]: https://github.com/fpco/lts-haskell#readme
[instructions]: https://about.gitlab.com/2016/04/07/gitlab-pages-setup/#custom-domains
[Namecheap]: https://www.namecheap.com/
[Cloudflare]: https://www.cloudflare.com/
[Useful Gist]: https://gist.github.com/cvan/8630f847f579f90e0c014dc5199c337b

[^1]: A nice introductory guide to how Docker works can be found [here](https://prakhar.me/docker-curriculum/).
[^2]: By the way, GitLab has [container registries](http://docs.gitlab.com/ce/user/project/container_registry.html).
  You can host custom images for any project on GitLab.
