# howtos
Howto's


Instalação de Dependências Base:

yum -y install yum-utils
yum-config-manager --add-repo https://rpm.kamailio.org/centos/kamailio.repo
yum -y install wget links perl-Config-Tiny-2.14-7.el7.noarch epel-release kernel-devel
Instalação RTPENGINE:
O Processo abaixo é baseado no link: https://blog.kolmisoft.com/rtpengine-install-on-centos-7/

Instalação Iptables:

systemctl stop firewalld
systemctl disable firewalld
yum -y install iptables-services iptables-devel
systemctl enable iptables.service
systemctl start iptables.service
iptables -F
service iptables save
Instalação de Dependências:

yum -y install https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm
yum -y install glib glib-devel gcc zlib zlib-devel openssl openssl-devel pcre pcre-devel libcurl libcurl-devel xmlrpc-c xmlrpc-c-devel
yum -y install libevent-devel glib2-devel json-c-devel json-glib json-glib-devel gperf libpcap-devel git perl-IPC-Cmd libiptcdata-devel libiptcdata-devel hiredis hiredis-devel redis iptables-devel libwebsockets-devel
yum -y install spandsp spandsp-devel
rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
yum -y install http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
yum -y install ffmpeg ffmpeg-devel
yum -y install bcg729 bcg729-devel
yum -y install mariadb-devel
Compilação e instalação do RtpEngine (Daemon):
*** Aqui fazemos um ajuste para não ter que atualizar o Openssl

cd /usr/src
git clone https://github.com/sipwise/rtpengine.git rtpengine
cd /usr/src/rtpengine/daemon/
wget https://github.com/neimaravila/howtos/raw/main/kamailio/rtpengine_daemon_patched_crypto.c -O crypto.c
make
cp -fr rtpengine /usr/sbin/rtpengine
Compilação e instalação do Módulo Iptables

cd /usr/src/rtpengine/iptables-extension
make all
cp -fr libxt_RTPENGINE.so /usr/lib64/xtables/.
Compilação e instalação do Módulo do Kernel:

cd /usr/src/rtpengine/kernel-module
make
cp -fr xt_RTPENGINE.ko /lib/modules/`uname -r`/extra/xt_RTPENGINE.ko
depmod -a
modprobe -v xt_RTPENGINE
Adicionar o Módulo do Kernel para subida no Boot:

echo "# Carregar modulo xt_RTPENGINE"  >> /etc/modules-load.d/rtpengine.conf
echo "xt_RTPENGINE" >> /etc/modules-load.d/rtpengine.conf
Baixar Arquivo de Configuração Padrão:

wget https://raw.githubusercontent.com/neimaravila/howtos/main/kamailio/rtpengine.sysconfig -O /etc/sysconfig/rtpengine
Configuração de Log:

echo "local1.*      -/var/log/rtpengine/rtpengine.log" >> /etc/rsyslog.conf
mkdir -p /var/log/rtpengine
touch /var/log/rtpengine/rtpengine.log
Configuração de LogRotate:

echo "/var/log/rtpengine/rtpengine.log {
daily
rotate 4
missingok
dateext
copytruncate
compress
}" > /etc/logrotate.d/rtpengine
perl -pi.bak -e 's#(\s+)\/var\/log\/messages#;local1.none$1/var/log/messages#' /etc/rsyslog.conf
systemctl restart rsyslog
Configurar o Controle de RTP:

echo 'add 63' > /proc/rtpengine/control
Adicionar Regra de Firewall:

iptables -I INPUT -p udp -j RTPENGINE --id 63
ip6tables -I INPUT -p udp -j RTPENGINE --id 63
iptables-save > /etc/sysconfig/iptables
ip6tables-save > /etc/sysconfig/ip6tables
Configurar Serviço RTPEngine:

wget https://raw.githubusercontent.com/neimaravila/howtos/main/kamailio/rtpengine.service -O /etc/systemd/system/rtpengine.service
systemctl enable rtpengine.service
mkdir -p /var/spool/rtpengine
Instalação do Banco de Dados Postgresql:

yum -y install postgresql-server
postgresql-setup initdb
sed -i 's/ident/trust/g' /var/lib/pgsql/data/pg_hba.conf
systemctl enable postgresql.service
systemctl start postgresql.service
Instalação do Kamailio:

yum -y install kamailio kamailio-websocket kamailio-postgresql kamailio-jansson kamailio-presence kamailio-outbound kamailio-regex kamailio-utils kamailio-json kamailio-uuid kamailio-tcpops kamailio-tls kamailio-dmq_userloc
yum -y install https://github.com/neimaravila/howtos/raw/main/kamailio/kamailio-geoip2-5.5.2-0.el7.centos.x86_64.rpm
yum -y install https://github.com/neimaravila/howtos/raw/main/kamailio/sngrep-1.4.10-0.el7.x86_64.rpm
Download de Configuração Base:

wget https://raw.githubusercontent.com/neimaravila/howtos/main/kamailio/kamctlrc -O /etc/kamailio/kamctlrc
Criação de Banco de Dados Padrão:
OBS: Responder y para as perguntas sobre tabelas de presença e extras.

kamdbctl create
INFO: creating database kamailio …
NOTICE: CREATE TABLE will create implicit sequence “version_id_seq” for serial column “version.id”
NOTICE: CREATE TABLE / PRIMARY KEY will create implicit index “version_pkey” for table “version”
NOTICE: CREATE TABLE / UNIQUE will create implicit index “version_table_name_idx” for table “version”
NOTICE: CREATE TABLE / PRIMARY KEY will create implicit index “topos_t_pkey” for table “topos_t”
INFO: Core Kamailio tables succesfully created.
Install presence related tables? (y/n): y
INFO: creating presence tables into kamailio …
NOTICE: CREATE TABLE will create implicit sequence “presentity_id_seq” for serial column “presentity.id”
…
NOTICE: CREATE TABLE / UNIQUE will create implicit index “rls_watchers_rls_watcher_idx” for table “rls_watchers”
INFO: Presence tables succesfully created.
Install tables for imc cpl siptrace domainpolicy carrierroute
drouting userblacklist htable purple uac pipelimit mtree sca mohqueue
rtpproxy rtpengine secfilter? (y/n): y
INFO: creating extra tables into kamailio …
NOTICE: CREATE TABLE will create implicit sequence “imc_rooms_id_seq” for serial column “imc_rooms.id”
…
NOTICE: CREATE TABLE / PRIMARY KEY will create implicit index “secfilter_pkey” for table “secfilter”
INFO: Extra tables succesfully created.

Criação de Certificado auto-assinado:

openssl req -x509 -newkey rsa:4096 -keyout /etc/kamailio/kamailio-selfsigned.key -out /etc/kamailio/kamailio-selfsigned.pem -days 365 -subj "/C=BR/ST=Minas Gerais/L=Belo Horizonte/O=FAL/OU=FAL/CN=kamailio.pabxip.com.br" -nodes -sha256
Download da Configuração padrão:

wget https://raw.githubusercontent.com/neimaravila/howtos/main/kamailio/kamailio.cfg -O /etc/kamailio/kamailio.cfg
Download e Aplicação de tabelas e views adicionais:

wget https://raw.githubusercontent.com/neimaravila/howtos/main/kamailio/tabelas_view.sql -O /tmp/tabelas_view.sql
Edição de domínio e senha de ramais.

O Arquivo /tmp/tabelas_view.sql contem alguns comandos INSERT INTO que criam a estrutura de empresa/dominio/ramal/servidorVoip.
Após fazer as edições necessárias, importar o arquivo:

su - postgres -c "psql -d kamailio < /tmp/tabelas_view.sql"
Alteração de Endereços IP´s (Substitua pelos seus respectivos IP’s Internos e Externos:

sed -i 's/KAMAILIO_IP_INTERNO/10.128.0.6/g' /etc/kamailio/kamailio.cfg
sed -i 's/KAMAILIO_IP_INTERNO/10.128.0.6/g' /etc/sysconfig/rtpengine
sed -i 's/KAMAILIO_IP_EXTERNO/34.66.60.137/g' /etc/sysconfig/rtpengine
sed -i 's/KAMAILIO_IP_EXTERNO/34.66.60.137/g' /etc/kamailio/kamailio.cfg
Subida de Serviços:

systemctl enable kamailio
service rtpengine start
service kamailio start
Após a inserção de novas informações de dispositivos SIP ou domínios no Kamailio, a informação deverá ser recarregada para a memória do mesmo, através dos seguintes comandos:

kamctl domain reload
kamcmd htable.reload server
