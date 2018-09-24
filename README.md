apt-get update && apt-get -y upgrade
apt-get -y install build-essential \
 gcc git libunbound-dev net-tools \
 libssl-dev libgetdns-dev \
 libyaml-dev libtool m4 autoconf

cd
git clone https://github.com/getdnsapi/getdns.git
cd getdns
git checkout develop
git submodule update --init
libtoolize -ci
autoreconf -fi
mkdir build && cd build

../configure --prefix=/usr/local/ \
--without-libidn \
--without-libidn2 \
--enable-stub-only \
--with-ssl \
--with-stubby

make
make install

## edit disini listen port dll
nano /usr/local/etc/stubby/stubby.yml

export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/lib"
ldconfig

## service
nano /usr/local/etc/stubby/stubby.conf

# tmpfiles.d (5) for use with stubby.service
d /usr/local/etc/stubby 0750 stubby stubby - -

mkdir /var/cache/stubby
nano /lib/systemd/system/stubby.service

[Unit]
Description=stubby DNS resolver

[Service]
User=stubby
DynamicUser=yes
CacheDirectory=/var/cache/stubby
WorkingDirectory=/usr/local/etc/stubby
ExecStart=/usr/local/bin/stubby
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target

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

netstat -tulpen -c
