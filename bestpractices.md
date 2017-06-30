**This is a reference document created based off various self learnings, articles, conference videos, etc**

### Tips to Reduce Docker Image Sizes
Docker images can easily get to 2–3GB. Here’s some tips that can help reduce their sizes.

#### Tip #1 — Use a smaller base image

`FROM ubuntu`

will set you to 128MB on the outset. Consider using a smaller base image. 

For each `apt-get install` or `yum install` line you add in your Dockerfileyou will be increasing the size of the image by that library size. Realize that you probably don’t need many of those libraries you are installing.

Consider using an alpine base image (only 5MB in size). Most likely, there are alpine tags for the programming language you are using. For example, Python has 2.7-alpine(~50MB) and3.5-alpine(~65MB).

#### Tip #2 — Don’t install debug tools like vim/curl

I notice many developers install `vim` and `curl` in their Dockerfile so that they can more easily debug their application. Unless your application depends on it, don’t install those dependencies. Doing so defeats the purpose of using a small base image.

But how do I debug?

One technique is to have a development Dockerfile and a production Dockerfile. During development, have all of the tools you need, and then when deploying to production remove the development tools.

#### Tip #3 — Minimize Layers
Each line of a Dockerfile is a step in the build process; a layer that takes up size. Combine your RUN statements to reduce the image size. Instead of

```
FROM debian
RUN apt-get install -y <packageA>
RUN apt-get install -y <packageB>
```

Do

```
FROM debian
RUN apt-get install -y <packageA> <packageB>
```

A drawback of this approach is that you’ll have to rebuild the entire image each time you add a new library. 

If you aren’t aware, Docker doesn’t rebuild layers it’s already built, and it caches the Dockerfile line by line Try changing one character of a Dockerfile you’ve already built, and then rebuild. 

You’ll notice that each step above that line will be recognized as already been built, but the line you changed (and each line following) will be rebuilt.

A strategy I recommend is that while in development and testing dependencies, separate out the RUN commands. Once you’re ready to deploy to production, combine the RUN statements into one line.

#### Tip #4 Use — no-install-recommends on apt-get install
Adding — `no-install-recommends` to `apt-get install -y` can help dramatically reduce the size by avoiding installing packages that aren’t technically dependencies but are recommended to be installed alongside packages.

apk add commands should have `--no-cache` added.

#### Tip #5 Add rm -rf /var/lib/apt/lists/* to same layer as apt-get installs
Add `rm -rf /var/lib/apt/lists/*` at the end of the `apt-get -y install` to clean up after install packages.

For yum, add `yum clean all`

Also, if you are install `wget` or `curl`in order to download some package, remember to combine them all in one RUN statement. Then at the end of the run statement, `apt-get remove curl or wget` once you no longer need them. 

This advice goes for any package that you only need temporarily.

#### Tip #6 Use FromLatest.io
FromLatest will Lint your Dockerfile and check for even more steps you can perform to reduce your image size.


