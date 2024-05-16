FROM python:3.9-slim

RUN apt-get update && apt-get install -y apache2-utils

WORKDIR /app

COPY . /app

RUN chmod +x generate-votes.sh && \
    python make-data.py

CMD ["sh", "/app/generate-votes.sh"]
