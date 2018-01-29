FROM zabbix/zabbix-server-mysql
COPY sendEmail-v1.56.tar.gz /tmp/
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories && \
apk add --no-cache perl-crypt-ssleay perl-io-socket-ssl ca-certificates tzdata && \
tar xzf /tmp/sendEmail-v1.56.tar.gz -C /tmp/ && \
rm -f /tmp/sendEmail-v1.56.tar.gz && \
sed -i "s/SSL_version => 'SSLv3 TLSv1'/SSL_version => 'TLSv1'/g" /tmp/sendEmail-v1.56/sendEmail && \
mv /tmp/sendEmail-v1.56/sendEmail /usr/bin && \
chmod +x /usr/bin/sendEmail && \
rm -rf /tmp/* && \
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
echo "Asia/Shanghai" > /etc/timezone 
EXPOSE 10051/TCP 162/UDP
WORKDIR /var/lib/zabbix
ENTRYPOINT ["/bin/bash"]
CMD ["/run_zabbix_component.sh", "server", "mysql"]
