# immutable operating systems + dev containers = a perfect match

TBD

## fedora silverblue

I've been using linux for work and personal projects since early 2000's, but the concept of immutable operating systems I discovered only recently and it immedialy captured my interest. I stopped counting how many times I've gotten myself in trouble messing up dependencies on my development machines which resulted in either me losing my progress, code or just time. [Fedora Silverblue](https://silverblue.fedoraproject.org) comes to the rescue and gives hope to engineers, who are permanently short on time, to get their experiments started and finalized in predictable time.

## dev containers

If you have ever used visual studio code and containers (with e.g. docker or podman), chances are you have heard about the [dev containers extension](https://code.visualstudio.com/docs/devcontainers/containers). If not - the extension lets you leverage containers to host your engineering toolchain in: locally installed visual studio code uses container-installed e.g. compiler, database, services, etc. so that you don't have to install anything directly on your local machine. This allows for super portable projects which you can simply pull, build and run locally with zero manual setup. The setup/configuration is handled by the dev container's dockerfile + orbiting scripts so that you can focus on the code and not on the infrastructure setup.

## setup

The match is, on paper, made in heaven, but getting both to work together required some head scratching. Here is how I did it
