FROM Ubuntu

RUN apt-get-update

RUN pip install flask

COPY . /opt/source_code

ENTRYPOINT FLASK_APP=/opt/source_code/app.py flask run