FROM ubuntu:16.04

RUN apt-get update \
	&& apt-get install -y build-essential wget zlib1g-dev


# Install samtools

RUN apt-get install -y python unzip gcc make bzip2 zlib1g-dev ncurses-dev 

RUN apt-get install -y samtools



#install bioawk

#RUN apt-get install -y byacc git \
  #&& git clone https://github.com/lh3/bioawk.git \
  #&& cd bioawk \
  #&& make \

# Install gmap

RUN wget http://research-pub.gene.com/gmap/src/gmap-gsnap-2018-02-12.tar.gz \
	&& tar -zxvf gmap-gsnap-2018-02-12.tar.gz \
	&& cd gmap-2018-02-12 \
	&& ./configure \
	&& make \
	&& make check \
	&& make install


# Install BLAT

#ADD http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/blat/blat /usr/bin/
ADD blat /usr/bin/

RUN chmod +x /usr/bin/blat

# Install FASTA

RUN cd /usr/local/ \
	&& wget http://faculty.virginia.edu/wrpearson/fasta/fasta33-35/fasta-35.4.12.tar.gz \
	&& tar -zxvf fasta-35.4.12.tar.gz \
	&& cd fasta-35.4.12/src \
	&& make -f ../make/Makefile.linux64 \
	&& echo 'PATH=${PATH}:/usr/local/fasta-35.4.12/bin'


# Install PASA

RUN apt-get install -y libdbd-mysql-perl libpng12-dev liburi-perl mysql-client

RUN cd /usr/local/ \
	&& wget https://github.com/PASApipeline/PASApipeline/archive/pasa-v2.2.0.tar.gz \
	&& tar -zxvf pasa-v2.2.0.tar.gz \
	&& cd PASApipeline-pasa-v2.2.0 \
	&& make \
	&& export PASAHOME=$(pwd) \
	&& cd seqclean && tar -zxvf seqclean.tar.gz \
	&& echo "export PASAHOME=${PASAHOME}" >> ~/.bashrc \
	&& echo 'PATH=${PATH}:${PASAHOME}/bin:${PASAHOME}/seqclean/seqclean::${PASAHOME}/seqclean/bin' >> ~/.bashrc

COPY conf.txt /usr/local/PASApipeline-pasa-v2.2.0/pasa_conf/


# Install packages for talking to mysql

#RUN apt-get install -y mysql-client
