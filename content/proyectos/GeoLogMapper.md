+++
date = '2024-12-08T18:13:04+01:00'
draft = false
title = 'GeoLogMapper'
+++
GeoLogMapper es un proyecto que realice hace un tiempo, ya que se me ocurrió tener un mapa cloropleto basándome en el trafico que accede a una página web. Es un proyecto útil para observar si los países que acceden a un servidor web que puede ser privado y no quieres que se acceda desde todo el mundo.
![Mapa cloropleto](/images/mapa.jpg "Mapa Cloropleto")
El proyecto consiste en varios apartados:
## Recopilación de datos:
Para recopilar los datos cree un pequeño script que apache ejecuta al escribir el log y guarda en un base de datos.
``` [python]
#!/usr/bin/python3
import sys
import signal
import mariadb
import os
from datetime import datetime
from IPcheck import IPchecker

# Handle SIGTERM signal
def signalHandler(signum,frame):
    connection.commit()
    cursor.close()
    connection.close()

# Connection params
conn_params= {
    "user" : os.getenv("DB_USER"),
    "password" : os.getenv("DB_PASS"),
    "host" : os.getenv("DB_HOST"),
    "database" : os.getenv("DB_NAME")
}

# Creating connection object
connection = mariadb.connect(**conn_params)
cursor = connection.cursor()

# Creating IPchecker object
checker = IPchecker()

# Handling SIGTERM signal
signal.signal(signal.SIGTERM,signalHandler)

# stdin loop
for line in sys.stdin:
    if not line == '':
        try:
            line_clean = line.split("[")
            ip = line_clean[0].split(" ")[0]
            line_clean = line_clean[1].split("]")
            time = datetime.strptime(line_clean[0],"%d/%b/%Y:%H:%M:%S %z")
            msg = line_clean[1]
            data = (ip,time,msg)
            cursor.execute("INSERT INTO Apache2Logs(ip,datetime,msg) VALUES(%s,%s,%s)",data)
            if checker.check(ip):
                r = checker.geolocate(ip)
                checkdata = (ip,r["country"],r["lat"],r["lon"],r["countryCode"])
                cursor.execute("INSERT INTO geolocatedips VALUES (%s,%s,%s,%s,%s)",checkdata) 
            connection.commit()
        except Exception as e:
            text = "Error " + str(e)
            open("error.log","a").write(text + "\n")
            print(status.text)
```
Para que se ejecute esto cree una imagen de Docker con el fichero de configuración de apache modificado.
``` [config]
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
	CustomLog "|/usr/bin/logscrapper.py" combined
</VirtualHost>
```
## Presentación de datos:
Para presentar los datos de una manera intuitiva he usado unas librerías de Python que me permitieron crear una capa encima del mapa con el mapa de calor de los países que mas acceden al servidor.
``` [python]
from flask import Flask, render_template_string
from sqlalchemy import create_engine
import pandas
import folium
import requests
import os

dbuser = os.getenv("DB_USER")
dbpass = os.getenv("DB_PASS")
dbhost = os.getenv("DB_HOST")
dbname = os.getenv("DB_NAME")
constr = "mariadb+pymysql://" + dbuser + ":" + dbpass + "@" + dbhost + "/" + dbname + "?charset=utf8mb4"
con = create_engine(constr)
app = Flask(__name__)

@app.route("/")
def fullscreen():
    """Simple example of a fullscreen map."""
    mapa = folium.Map(tiles="cartodb positron",zoom_start=3, location=[15.00,0.00])
    geojson = requests.get("http://geojsonAPI/get110m").json()
    datos = pandas.read_sql(sql="select country, timesFromCountry from countries",con=con)
    folium.Choropleth(geo_data=geojson,data=datos,columns=["country","timesFromCountry"], key_on="feature.properties.ISO_A3_EH").add_to(mapa)
    return mapa.get_root().render()
```
## Interactividad:
Para poder tener la información de una manera interactiva use un Bot de telegram que usando otra librería de Python sacaba una captura del mapa y la envía por telegram al recibir un comando.
``` [python]
from flask import Flask, request
from telegram import TelegramAPI
from remoteGetter import RemoteGetter
from enviroment import Enviroment
from base64 import b64decode

env = Enviroment()
Api = TelegramAPI(env.bot_id,env.chat_id)
Remote = RemoteGetter(env.dataserver,env.dataport)
app = Flask(__name__)

@app.route("/", methods=['POST'])
def get_message():
    datos = request.get_json()
    message_txt = datos["message"]["text"]
    match message_txt:
        case "/ping@mariol03_bot":
            Api.sendMessage("pong")
        case "/geologmapper@mariol03_bot":
            Api.sendImageBytes(b64decode(Remote.getImage().encode("UTF-8")))
        case "/iplist@mariol03_bot":
            Api.sendTable(Remote.getTable())
    return {"result":"ok"}
```

# Resultados
![Imagen mostrando chat de telegram](/images/TelegramChat.png "Imagen mostrando chat de telegram")