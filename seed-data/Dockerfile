FROM python:3.12

RUN apt-get update && apt-get install -y apache2-utils

WORKDIR /app

ADD . /app

RUN chmod +x generate-votes.sh

RUN python make-data.py

## Multistage pour ce container apres qu'il ai genere la data??

CMD ["/bin/bash", "-c", "./generate-votes.sh"]
