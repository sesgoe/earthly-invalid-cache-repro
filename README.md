# Invalid Cache Layer Reproduction

In reference to: https://github.com/earthly/earthly/issues/1635

Steps to reproduce:

1. Clone this repo
2. Ensure Earthly is installed (`earthly version v0.6.7 30a39d9b10c429e5a34cdd753652baf9966a3b63 linux/amd64; Ubuntu 20.04`)
3. Change the `SAVE IMAGE` lines to use your own desired container registry, or you can run with mine to see the failure
4. `earthly --ci --push +ubuntu-installs` first to build this cache
5. `earthly --ci --push +min --DOCKER_TAG=value1` `value1` can be whatever you want, just an example tag

I have included the Broken Earthfile as well as a version with a minor change called `Earthfile.maybefix` that changes this:

```
FROM +ubuntu-installs
ARG DOCKER_TAG
RUN echo "$DOCKER_TAG"
WORKDIR /app
COPY requirements_step_1.txt ./requirements_step_1.txt
COPY requirements_step_2.txt ./requirements_step_2.txt
RUN pip install --upgrade pip
#RUN pip install -r requirements_step_1.txt
#RUN pip install -r requirements_step_2.txt
SAVE IMAGE --push sesgoe/earthly-invalid-cache-repro:min_$DOCKER_TAG
```

to this:

```
FROM +ubuntu-installs
# 👈 moved ARG line
# 👈 removed echo line
WORKDIR /app
COPY requirements_step_1.txt ./requirements_step_1.txt
COPY requirements_step_2.txt ./requirements_step_2.txt
RUN pip install --upgrade pip
#RUN pip install -r requirements_step_1.txt
#RUN pip install -r requirements_step_2.txt
ARG DOCKER_TAG # 👈 ARG is now here
SAVE IMAGE --push sesgoe/earthly-invalid-cache-repro:min_$DOCKER_TAG
```

and it _seems_ to work.
