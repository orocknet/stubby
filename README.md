```
apt-get update && \
apt-get -y upgrade && \
apt-get -y install build-essential \
 gcc git libunbound-dev net-tools \
 libssl-dev libgetdns-dev \
 libyaml-dev libtool m4 autoconf
```
```
cd
git clone https://github.com/getdnsapi/getdns.git
cd getdns
git checkout develop
git submodule update --init
libtoolize -ci
autoreconf -fi
mkdir build && cd build
```
```
../configure --prefix=/usr/local/ \
--without-libidn \
--without-libidn2 \
--enable-stub-only \
--with-ssl \
--with-stubby

make
make install
```
```
rm /usr/local/etc/stubby/stubby.yml
wget -O /usr/local/etc/stubby/stubby.yml https://orocknet.github.io/stubby/stubby.yml
wget -O /usr/local/etc/stubby/stubby.conf https://orocknet.github.io/stubby/stubby.conf
wget -O /lib/systemd/system/stubby.service https://orocknet.github.io/stubby/stubby.service

export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/lib"
ldconfig
d /usr/local/etc/stubby 0750 stubby stubby - -
useradd -s /usr/sbin/nologin -r -M stubby

chown stubby:stubby /usr/local/etc/stubby/stubby.conf
chown stubby:stubby /usr/local/etc/stubby/stubby.yml
chown stubby:stubby /usr/local/bin/stubby
chown stubby:stubby /var/cache/stubby

systemctl daemon-reload
systemctl enable stubby
systemctl start stubby
systemctl status stubby
systemctl restart stubby
```
netstat -tulpen -c
