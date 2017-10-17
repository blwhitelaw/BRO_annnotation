## Notes on installing PASA

This are for an Ubuntu 14 image

Install mysql

```
sudo apt-get install mysql-server
sudo mysql_secure_installation

mysql root password b28TZKwX

sudo mysql_install_db


sudo apt-get install libdbd-mysql-perl
```




Install GMAP

```
#wget http://research-pub.gene.com/gmap/src/gmap-gsnap-2016-08-24.tar.gz # Doesn't work because --introlength no longer supported

wget http://research-pub.gene.com/gmap/src/gmap-gsnap-2015-12-31.v10.tar.gz

sudo apt-get install build-essential


./configure
make
make check
sudo make install
```

Install BLAT

```
sudo apt-get install unzip
wget http://hgwdev.cse.ucsc.edu/~kent/src/blatSrc35.zip

unzip blatSrc35.zip
cd blatSrc

sudo apt-get install libpng12-dev
mkdir -p ~/bin/$MACHTYPE
mkdir lib/$MACHTYPE

export MACHTYPE
make

```


Install FASTA

```

wget http://faculty.virginia.edu/wrpearson/fasta/fasta33-35/fasta-35.4.12.tar.gz

cd fasta-35.4.12/src
make -f ../make/Makefile.linux64


```


Download and Install PASA

```
wget https://github.com/PASApipeline/PASApipeline/archive/v2.1.0.tar.gz

tar -zxvf v2.1.0.tar.gz

mv PASApipeline-2.1.0 /usr/local/bin/PASA

export PASAHOME=/usr/local/bin/PASA
cd $PASAHOME
make

cd seqclean
tar -zxvf seqclean.tar.gz

```

Create PASA mysql ro_user

```bash
mysql --user root --password
```

```
CREATE USER 'pasa_ro'@'localhost' IDENTIFIED BY 'b28TZKwX';
GRANT SELECT, SHOW VIEW, PROCESS, REPLICATION CLIENT ON *.* TO 'pasa_ro'@'localhost';
FLUSH PRIVILEGES;

CREATE USER 'pasa_rw'@'localhost' IDENTIFIED BY 'b28TZKwX';
GRANT ALL PRIVILEGES ON *.* TO 'pasa_rw'@'localhost';
FLUSH PRIVILEGES;

```

Setup PASA Web portal

```
	sudo apt-get install apache2
	sudo apt-get install libgd-perl
	sudo apt-get install cpanminus
	sudo cpanm URI::Escape

	cd /etc/apache2/mods-enabled/
	ln -s ../mods-available/cgi.load .
	sudo service apache2 reload

	sudo cp -r ${PASAHOME} /usr/lib/cgi-bin/
	cd /usr/lib/cgi-bin/
	chmod -R 755 ./PASA/
```


```

Configure PASA

This is done by configuring settings in `/usr/local/bin/PASA/pasa_conf/conf.txt`

Update PATH

Put the following lines into ~/.bash_profile

```
export PATH=${PATH}:/usr/local/bin/PASA/bin:/usr/local/bin/PASA/seqclean/seqclean/
export PATH=${PATH}:/usr/local/bin/PASA/seqclean/bin/
```



Test Run

```
cd $PASAHOME/sample_data

../scripts/Launch_PASA_pipeline.pl -c alignAssembly.config -C -R -g genome_sample.fasta -t all_transcripts.fasta.clean -T -u all_transcripts.fasta -f FL_accs.txt --ALIGNERS blat,gmap --CPU 2


```

If a test run fails delete the dblike this

```
mysqladmin drop bro_pasa -u root --password=b28TZKwX
```

Since the mysql database can get very large we move it to a larger partition.  This is necessary on Nectar images where the root partition is quite small

```bash
sudo stop mysql
sudo rsync -a /var/lib/mysql /mnt/

```

Edit `/etc/mysql/my.cnf` to update the datadir entry

Backup the old datadir

```bash
sudo mv mysql mysql.bak
```

Update apparmour

```
echo "alias /var/lib/mysql/ -> /mnt/mysql/," >> /etc/apparmor.d/tunables/alias
```







