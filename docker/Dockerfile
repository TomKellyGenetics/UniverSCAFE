# FROM ubuntu:bionic
FROM tomkellygenetics/cellranger_clean:latest

RUN apt-get update \
 && apt-get upgrade -y \
 && apt-get install -y \
 git \
 git-lfs \
 make \
 gzip \
 pigz \
 rename

RUN apt-get install -y \
  curl \
  zsh \
  nano \
  && sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh) --unattended"

RUN git clone "https://github.com/TomKellyGenetics/universc.git"

RUN cd universc/test/cellranger_reference/cellranger-tiny-ref/ \
 && git lfs pull \
 && rm -rf 3.0.0 1.2.0 \ 
 && cellranger mkref --genome=3.0.0 --fasta=genome-3.0.0.fa --genes=genes-3.0.0.gtf \
 && cellranger mkref --genome=1.2.0 --fasta=genome-1.2.0.fa --genes=genes-1.2.0.gtf 

RUN cd universc \
 && make reference \
 && cd .. 

RUN mkdir -p /cellranger-3.0.2.9001/cellranger-tiny-fastq \
 && ln -s /universc/test/shared/cellranger-tiny-fastq/1.2.0 /cellranger-3.0.2.9001/cellranger-tiny-fastq \
 && ln -s /universc/test/shared/cellranger-tiny-fastq/3.0.0 /cellranger-3.0.2.9001/cellranger-tiny-fastq

RUN mkdir -p /cellranger-3.0.2.9001/cellranger-tiny-ref \
 && ln -s /universc/test/cellranger_reference/cellranger-tiny-ref/1.2.0 /cellranger-3.0.2.9001/cellranger-tiny-ref \ 
 && ln -s /universc/test/cellranger_reference/cellranger-tiny-ref/3.0.0 /cellranger-3.0.2.9001/cellranger-tiny-ref

ENV PATH universc:$PATH

RUN ln -s /universc/launch_universc.sh /cellranger-3.0.2.9001/cellranger-cs/3.0.2.9001/bin/universc

ENV LC_ALL C.UTF-8
ENV LANG C.UTF-8

RUN apt-get install -y \
 python3-pip \
 && python3 -m pip install --upgrade pip setuptools wheel \
 && pip3 install multiqc

RUN git clone https://github.com/linsalrob/fastq-pair.git \
 && cd fastq-pair \
 && mkdir build \
 && cd build \
 && gcc -std=gnu99   ../main.c ../robstr.c ../fastq_pair.c ../is_gzipped.c  -o fastq_pair \
 && cp fastq_pair /bin/fastq_pair

#---[2021/03/22 15:53] update apt-get, grab R 3.6.3 and perl
RUN echo "deb http://cloud.r-project.org/bin/linux/debian stretch-cran35/" >>/etc/apt/sources.list \
 && apt-get update \
 && apt-get upgrade -y \
 && apt-get install -y perl git wget \
 && apt-get install -y --allow-unauthenticated r-base-core

#---[2021/03/22 15:54] install missing libs for compile R packages
RUN apt-get -y install libcurl4-gnutls-dev \
 && apt-get -y install libssl-dev \
 && apt-get -y install libxml2-dev \
 && apt-get -y install libfontconfig1-dev \
 && apt-get -y install libharfbuzz-dev \
 && apt-get -y install libfribidi-dev \
 && apt-get -y install libfreetype6-dev \
 && apt-get -y install libtiff5-dev \
 && apt-get -y install libjpeg-dev \
 && apt-get -y install libpng-dev \
 && apt-get -y install libudunits2-dev \
 && apt-get -y install libgdal-dev

#---[2021/03/22 15:54] install common R packages
RUN Rscript -e 'install.packages("Matrix", repos="http://R-Forge.R-project.org")' \
 && Rscript -e 'install.packages("R.utils")' \
 && Rscript -e 'install.packages("data.table")' \
 && Rscript -e 'install.packages("docopt")' \
 && Rscript -e 'install.packages("ROCR")' \
 && Rscript -e 'install.packages("PRROC")' \
 && Rscript -e 'install.packages("caret")' \
 && Rscript -e 'install.packages("e1071")' \
 && Rscript -e 'install.packages("ggplot2")' \
 && Rscript -e 'install.packages("scales")' \
 && Rscript -e 'install.packages("reshape2")'
 
#---[2021/03/22 15:54] install monocle and cicero
RUN Rscript -e 'install.packages("devtools")' \
 && Rscript -e 'install.packages("BiocManager")' \
 && Rscript -e 'devtools::install_version("RcppAnnoy", "0.0.16")' \
 && Rscript -e 'install.packages("BiocManager")' \
 && Rscript -e 'BiocManager::install(c("BiocGenerics", "DelayedArray", \
    "DelayedMatrixStats","limma", "S4Vectors", "SingleCellExperiment", \
    "SummarizedExperiment", "batchelor", "Matrix.utils"))' \
 && Rscript -e 'devtools::install_github("cole-trapnell-lab/leidenbase")' \
 && Rscript -e 'devtools::install_github("cole-trapnell-lab/monocle3")' \
 && Rscript -e 'BiocManager::install(c("Gviz", "GenomicRanges", "rtracklayer"))' \
 && Rscript -e 'devtools::install_github("cole-trapnell-lab/cicero-release", ref = "monocle3")'

#---[2021/03/22 15:54] clone the git
RUN git clone -v https://github.com/chung-lab/SCAFE 2>2021032308

#---[2021/03/22 15:54] install the paths
WORKDIR ./SCAFE

RUN chmod 755 -R ./scripts/ \
 && chmod 755 -R ./resources/bin/ \
 && echo "export PATH=\$PATH:/SCAFE/scripts" >>~/.bashrc

#---[2021/03/22 15:54] start bash when run
CMD ["/bin/bash"]
