# anaconda.cfg

# HOW TO BOOT
    Set Image ISO
    Boot
    Push Down Esc Key
    linux text ks=https://lazyarea.github.io/anaconda.cfg/kickstart-centos.cfg

# interfaces
    network --bootproto=dhcp   --device=enp0s3 --noipv6 --activate

    network --bootproto=static --device=enp0s8 --ip=192.168.0.20
    --netmask=255.255.255.0 --noipv6 --activate　--nodefroute

# rootpw
    python -c 'import crypt; print(crypt.crypt("SyokiPass@9999",crypt.METHOD_SHA512))'

# phantomjs
    git clone git://github.com/ariya/phantomjs.git
    cd phantomjs
    git checkout 2.1.1
    git submodule init
    git submodule update
    python build.py
    export PATH=$PATH:/path/to/phantomjs/bin

# httpd
    mkdir -p /home/sites/eccube.example.com/{logs,html}
    DOMAIN=example.com
    FQDN=www.${DOMAIN}
    mkdir -p /home/sites/${FQDN}/html/
    mkdir -p /home/sites/${FQDN}/logs/
    echo ${FQDN} > /home/sites/${FQDN}/html/index.html
    chown -R vagrant:vagrant /home/sites/${FQDN}/html
    chown vagrant:vagrant /home/sites/${FQDN}/

    cat > /etc/httpd/conf.d/${FQDN}.conf << "EOF.httpdconf"
    <VirtualHost *:80>
    TABServerAdmin webmaster@DOMAIN
    TABServerName FQDN
    TABServerAlias DOMAIN
    TABDocumentRoot /home/sites/FQDN/html
    TABCustomLog /home/sites/FQDN/logs/access_log combined
    TABErrorLog  /home/sites/FQDN/logs/error_log
    TABTAB<Directory /home/sites/FQDN/html/>
    TABTABOptions ExecCGI Includes FollowSymlinks
    TABTABAllowOverride Options FileInfo AuthConfig Limit
    TABTABRequire all granted
    TAB</Directory>
    </VirtualHost>
    EOF.httpdconf

    sed -i -e "s/DOMAIN/${DOMAIN}/g" /etc/httpd/conf.d/${FQDN}.conf
    sed -i -e "s/FQDN/${FQDN}/g" /etc/httpd/conf.d/${FQDN}.conf
    sed -i -e "s/TAB/    /g" /etc/httpd/conf.d/${FQDN}.conf

    httpd -t
    ## check getenforce
    systemctl status httpd
    systemctl restart httpd

# before mkiso
    mount -t iso9660 -p loop CentOS-7-x86_64-Minimal-1511.iso ./isofs/
    cd /mounted/path/to/base/isolinux/
    curl -O https://lazyarea.github.io/anaconda/centos.cfg
    mv centos.cfg ks.cfg
# edit isolinux.cfg
    - #default vesamenu.c32
    + default setup
    ...
    label check 
    -     menu default
    ...
    + label check
    +      menu label Test this ^media & install CentOS 7
    +      kernel vmlinuz
    +      append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 rd.live.check quiet

# mkiso
    mkisofs -v -r -J -o ../CentOS-7-x86_64-Minimal-1511-01.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table .

