
FROM oraclelinux:8
WORKDIR /function
RUN groupadd --gid 1000 fn && adduser --uid 1000 --gid fn fn

ARG release=19
ARG update=3

RUN dnf -y module install python39
RUN dnf -y install python39-pip 
RUN dnf -y install oracle-instantclient-release-el8
RUN dnf -y install oracle-instantclient-basic
RUN dnf -y install oracle-instantclient-sqlplus
RUN ln -s /usr/lib/oracle/21 /usr/lib/oracle/current
RUN rm -rf /var/cache/yum
     
RUN mkdir /tmp/wallet
RUN mkdir /tmp/work_report_dir
RUN chown -R fn:fn /tmp/wallet
RUN chown -R fn:fn /tmp/work_report_dir
ENV TNS_ADMIN=/tmp/wallet
ENV proxy=
ENV tagspecial=
ENV tagspecial2=
ENV filedate=
ENV fileid=

ADD . /function/
RUN alternatives --set python3 /usr/bin/python3.9 
RUN python3 -m pip install --upgrade pip
RUN python3 -m pip install --no-cache --no-cache-dir -r requirements.txt
RUN rm -fr /function/.pip_cache ~/.cache/pip requirements.txt func.yaml Dockerfile README.md

ENV PYTHONPATH=/python
ENTRYPOINT ["/usr/local/bin/fdk", "/function/func.py", "handler"]