FROM python:3.9

RUN pip install Django==3.2

ADD . /app

EXPOSE 8000

ENTRYPOINT python /app/manage.py runserver 0.0.0.0:8000
