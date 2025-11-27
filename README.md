# seq2scfv-fixes

## About

[Seq2scfv](https://github.com/ngs-ai-org/seq2scfv) is a fantastic tool for scFv analysis, but I had trouble getting it to run on my machines. This repository documents fixes and workarounds that made it functional for me. It's not affiliated with the original authors. I submitted a related [issue](https://github.com/ngs-ai-org/seq2scfv/issues/2) and pull requests, but given that the official repo is tied to a published paper, I understand that updates may be limited. Hence, I will maintain any further updates or tweaks here in case they're useful to others. I'm not a Docker expert - feedback always welcome!

## The Issue

I encountered compatibility and permission issues when building the Dockerfile with R 3.6.3, which is the default R version in Ubuntu 20.04 (the base image used in this Dockerfile). Specifically:
1. `ggplot2` installation fails due to the unavailability of `gtable` for R 3.6.3 on CRAN.
2. A permissions issue prevents `install2.r` from executing successfully.

## The Solutions

There are two versions of the solution; both modify a chunk of the `Dockerfile` code (make it look like below).

**Simple Solution:** This solution installs gtable version 0.3.0 from the CRAN archive, which is compatible with R 3.6.3, and sets the correct permissions for install2.r. This approach ensures the build completes successfully without many changes.

```dockerfile
# Ensure install2.r has executable permissions
RUN chmod +x /usr/bin/install2.r

# Install gtable version 0.3.0 from the CRAN archive
RUN Rscript -e "install.packages('https://cran.r-project.org/src/contrib/Archive/gtable/gtable_0.3.0.tar.gz', repos = NULL, type = 'source')"

# Install other necessary R packages, including ggplot2 and gridExtra
RUN /usr/bin/install2.r --libloc /home/R --error --repos "https://stat.ethz.ch/CRAN/" \
    data.table \
    reshape2 \ 
    dplyr \
    tidyr \
    ggplot2 \
    gridExtra \
    stringr
```

**Alternative Solution:** If you prefer the newest acceptable gtable version (0.3.5), this solution seems to work for me, but requires separating the installation of the libraries into a few steps.

```dockerfile
# Set executable permission for install2.r
RUN chmod +x /usr/bin/install2.r

# Install core libraries first to cache this layer
RUN /usr/bin/install2.r --libloc /home/R --error --repos "https://stat.ethz.ch/CRAN/" \
    data.table \
    reshape2 \
    dplyr \
    tidyr \
    stringr

# Install gtable version 0.3.5 from the archive
RUN Rscript -e "install.packages('https://cran.r-project.org/src/contrib/Archive/gtable/gtable_0.3.5.tar.gz', repos = NULL, type = 'source')"

# Install ggplot2 and gridExtra separately to prevent conflicts
RUN /usr/bin/install2.r --libloc /home/R --error --repos "https://stat.ethz.ch/CRAN/" \
    ggplot2 \
    gridExtra
```

## Summary

I hope this helps anyone trying to replicate the setup or adapting the environment, especially as CRAN packages evolve over time. Perhaps switching the R version would be easier, but it would impact reproducibility and I learned not to mess with R in the past!

Internally, `seq2scfv` depends on `IgBlast`, which can be tricky to configure. The documentation is limited, and errors are common - especially when working with non-default species that require custom-prepared reference files. I've seen issues raised about it without clear solutions. 

After a fair bit of trial, error, and reverse engineering, I'm preparing a tutorial on such IgBlast setup - stay tuned. 

I also plan to release tools for analyzing and visualizing seq2scfv output - links coming. 

If you're interested in these, please get in touch, so we can discuss the needs, features, or collaborate on it! 
