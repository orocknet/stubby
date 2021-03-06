## stubby install
```
apt-get update && \
apt-get -y upgrade && \
apt-get -y install build-essential \
 gcc git libunbound-dev net-tools \
 libssl-dev libgetdns-dev libyaml-dev \
 libtool m4 autoconf automake

git clone https://github.com/getdnsapi/getdns.git
cd getdns
git checkout develop
git submodule update --init
libtoolize -ci
autoreconf -fi
mkdir build && cd build

../configure \
--prefix=/usr/local/ \
--without-libidn \
--without-libidn2 \
--enable-stub-only \
--with-ssl \
--with-stubby

make
make install

mv /usr/local/etc/stubby/stubby.yml /usr/local/etc/stubby/stubby.yml.orig
wget -O /usr/local/etc/stubby/stubby.yml https://orocknet.github.io/stubby/stubby.yml
wget -O /usr/local/etc/stubby/stubby.conf https://orocknet.github.io/stubby/stubby.conf
wget -O /lib/systemd/system/stubby.service https://orocknet.github.io/stubby/stubby.service

export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/lib"
ldconfig
useradd -s /usr/sbin/nologin -r -M stubby

chown -R stubby:stubby /usr/local/etc/stubby
mkdir /var/cache/stubby && chown -R stubby:stubby /var/cache/stubby
chown stubby:stubby /usr/local/bin/stubby

systemctl daemon-reload
systemctl enable stubby
systemctl start stubby

netstat -tlpn -c
```
