FROM owasp/modsecurity:v3-ubuntu-nginx
MAINTAINER Andrea Menin (theMiddle)

ENV PARANOIA=1

#RUN dnf -y update

#RUN dnf -y install python

RUN apt-get update && apt-get install -y \
	git \
	python

RUN mv /etc/nginx/modsecurity.d/modsecurity.conf /etc/nginx/modsecurity.d/modsecurity.conf.old && \
  cd /opt && \
  git clone https://github.com/Rev3rseSecurity/wordpress-modsecurity-ruleset.git && \
  cd wordpress-modsecurity-ruleset && git checkout $COMMIT && \
  echo 'SecAction "id:22000025,phase:1,nolog,pass,t:none,setvar:tx.wprs_allow_xmlrpc=0"' >> /etc/nginx/modsecurity.d/include.conf && \
  echo 'SecAction "id:22000030,phase:1,nolog,pass,t:none,setvar:tx.wprs_allow_user_enumeration=0"' >> /etc/nginx/modsecurity.d/include.conf && \
  echo 'Include /opt/wordpress-modsecurity-ruleset/*.conf' >> /etc/nginx/modsecurity.d/include.conf

COPY modsecurity.conf /etc/nginx/modsecurity.d/
COPY crs-setup.conf /etc/nginx/modsecurity.d/

EXPOSE 80

#ENTRYPOINT ["/docker-entrypoint.sh"]
#CMD ["httpd", "-k", "start", "-D", "FOREGROUND"]
CMD ["/usr/local/nginx/nginx", "-g", "daemon off;"]
