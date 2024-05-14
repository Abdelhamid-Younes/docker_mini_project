FROM python:3.8-buster
LABEL maintainer="Abdelhamid YOUNES abdelhamid.younes21@gmail.com"
COPY student_age.py requirements.txt ./
RUN apt-get update -y && apt-get install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
RUN  pip install --upgrade pip && pip3 install -r /requirements.txt
EXPOSE 5000
ENTRYPOINT [ "python3", "./student_age.py" ]
