# seq2scfv-fixes

[Seq2scfv](https://github.com/ngs-ai-org/seq2scfv) is a fantastic tool for scFv analysis, but I ran into issues getting it to run on my machines. Here, I document fixes that make it functional for me. This project is not affiliated with the original authors. Originally, I reported the issue in [Issue #2](https://github.com/ngs-ai-org/seq2scfv/issues/2), but since the official repository is linked to a published paper, I understand that any updates may be limited. I will maintain fixes with any further updates and comments here, in case they help others. I’m not a Docker expert, so I hope this makes sense - feedback and improvements always welcome!

## The issue

I encountered compatibility and permission issues when building the Dockerfile with R 3.6.3, which is the default R version in Ubuntu 20.04 (the base image used in this Dockerfile). Specifically:
1. `ggplot2` installation fails due to the unavailability of `gtable` for R 3.6.3 on CRAN.
2. A permissions issue prevents `install2.r` from executing successfully.

# Solutions

There are two options to modify the code (to do: specify where!).

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

I hope this helps anyone replicating the setup or adapting the environment, especially as CRAN packages evolve. Perhaps changing the R version would be easier, but it would impact reproducibility and I learned not to mess with it in the past!
