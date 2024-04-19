FROM python:3.12

WORKDIR /app

ADD . /app

# Install requirements
RUN pip install --no-cache-dir -r requirements.txt

# Port par défaut mais on peut configuer 5001, 5002, etc. lors du run
EXPOSE 5000

CMD ["python", "app.py"]