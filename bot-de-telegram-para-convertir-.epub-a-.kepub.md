---
description: Veo mucho potencial en Telegram para automatizar mediante bots ...
---

# Bot de Telegram para convertir .epub a .kepub

> Febrero - 2021

Veo mucho potencial en Telegram para automatizar mediante bots pequeñas tareas que hacemos habitualmente.

Soy usuario habitual de lector de e-book de marca **Kobo**, y utilizo principalmente como forma de cargar libros la conexión wifi y el navegador incluido en el propio lector. De esta forma desde cualquier sitio con Internet y sin necesidad de cables es muy fácil cargar libros en el lector.

En este caso, al tiempo que creo un bot de Telegram por primera vez, intento automatizar la conversión de ebooks desde formato .epub al formato nativo de Kobo, .kepub. Considero que hay ciertas mejoras en el formateo de los libros que hacen más atractivo el formato .kepub

¿Qué necesitamos para la construcción del Bot?

* La propia creación del Bot desde Telegram. Esto nos dará el _**token**_ de acceso a la API HTTP de Telegram. La creación del bot la haremos a partir de una conversación desde Telegram con **BotFather** arrancando con el comando **/newbot.** Seguiremos la conversación en la que nos pedirá nombre del bot que queremos crear, imagen del mismo, descripción ... y tras todo esto BotFather nos devolverá **** el token de acceso a la API HTTP.

![](<.gitbook/assets/Screenshot from 2021-02-23 16-53-27.png>)

* _**Kepubify**_ de Patrick Gaskin. Utilidad para la conversión del archivo .epub a .kepub, la podéis encontrar aquí: [https://pgaskin.net/kepubify/](https://pgaskin.net/kepubify/). Recordar dar permisos de ejecución al fichero mediante _**`chmod +x`**_
* Código del bot, en Python en este caso, para hacer la conversión de formato y el envío al repositorio web de libros elegido, pCloud en mi caso. El código lo podemos ver un poco más abajo y está al 95% basado en el bot de **Helia** [https://github.com/Helias/EPUB-to-PDF](https://github.com/Helias/EPUB-to-PDF), utilizando en mi caso Kepubify en lugar de Calibre.
* Un repositorio web al que poder acceder desde el navegador del lector Kobo. Será aquí donde el programa de Python dejará el e-book una vez convertido a .kepub
  * En mi caso utilizo **pCloud**, pero podría ser cualquier otro servicio de disco en nube tipo Google Drive, MS OneDrive, Dropbox, ...
* Requisitos en Python:
  * pip
    * python-telegram , `$pip3 install python-telegram`
    * python-telegram-bot ,`$pip3 install python-telegram-bot`
  * pcloud, librería en Python ([https://pypi.org/project/pcloud/](https://pypi.org/project/pcloud/)) para acceso al almacenamiento en nube de pCloud
    * `$pip3 install pcloud`
* **keyring**, lo utilizo para almacenar las credenciales de pCloud que paso al script de Python. Una vez convertido el archivo .epub a .kepub copio éste a pCloud. _A continuación_ unas líneas en Python de como cargar la identidad en el _keyring_ _no protegido por contraseña._ En el caso de que el keyring esté protegido con contraseña no será posible acceder desde el script principal del bot a la identidad y contraseña guardada en keyring.&#x20;

```python
$pyton3
>>> import keyring
>>> from keyrings.alt.file import PlaintextKeyring
>>> k = PlaintextKeyring()
>>> keyring.set_keyring(k)
>>> keyring.set_password("test", "username", "password")
>>> keyring.get_password("test", "username")
'password'
>>> quit()
```

* Una máquina Linux donde hospedar el script de Python. Opciones económicas pueden ser las de Contabo o Hetzner, o por ejemplo la opción gratuita de Oracle Cloud (OCI).&#x20;
* Creación del servicio en Linux para que el script de Python arranque de forma automática y desatendida con el inicio del servidor Linux.
  * Con campos en archivo _**.service**_ User= y Group= de acuerdo a los permisos sobre el directorio que aloja el script
  * Y yo incluyo como variable de entorno el token de acceso a la API HTTP. Esta variable de entorno la capturará el script del bot para _validarse_ con Telegram

```bash
[Unit]
Description=Telegram bot para conversion a KEPUB
After=multi-user.target

[Service]
User=usuario    
Group=grupo
Type=simple
Environment="TELEGRAM_TOKEN=XXXXXXXXXX:asdasadfs-asdf5cJM-kxxxy7-XCxsseZ3fsdf"
WorkingDirectory=/directoriodetubot
ExecStart=/usr/bin/python3 /directoriodetubot/tubot.py

[Install]
WantedBy=multi-user.target
```

#### El script resultante es el siguiente:

```python
from telegram import InlineKeyboardButton, InlineKeyboardMarkup, Update
from telegram.ext import Updater, CommandHandler, MessageHandler, RegexHandler, CallbackQueryHandler, Filters, CallbackContext

from pcloud import PyCloud
# pc = PyCloud('tuusuario@dominio.com', 'password')
# Esta sería la opción, menos segura, de incluir el usuario y contraseña de acceso
# a pCloud en el propio script

import keyring
import logging
import os
import sys
import subprocess
import glob

# get token from token.conf
# TOKEN = open("token.conf", "r").read().strip()
# Esta sería la opción para incorporar el Token de Telegram obtenido durante la
# creación del mismo a partir de un fichero token.conf existente en el mismo 
# directorio del script
# En mi caso lo importaré de una variable de entorno
TOKEN = os.environ['TELEGRAM_TOKEN']
# Enable logging

logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)
logger = logging.getLogger(__name__)

def start(update: Update, context: CallbackContext):
  update.message.reply_text('Hi! Send me an epub I will convert it into KEPUB!')


def hola(update, context):
  update.message.reply_text('Hello World!')

def list(update, context):
  listing = glob.glob('/directoriodetubot/*.epub')
  for filename in listing:
        update.message.reply_text(filename)

def dir(update, context):
  raiz = "/directoriodetubot/"
  dir_list = os.listdir(raiz)
  update.message.reply_text(dir_list)

def echo(update, context):
    """Echo the user message."""
    update.message.reply_text(update.message.text)

def error(update, context):
    """Log Errors caused by Updates."""
    logger.warning('Update "%s" caused error "%s"', update, context.error)

def conversion(update: Update, context: CallbackContext):

    if update.message.document:
        file_name=update.message.document.file_name
        file = context.bot.getFile(update.message.document.file_id)
        file_path = file_name
        file_extension = os.path.splitext(file_path)[1]
        update.message.reply_text(file_extension)
        if file_extension.lower() == ".epub":
              file.download(file_name)
              update.message.reply_text('Processing...' + file_name)
              file_kepub = file_name.replace(".epub", "_converted.kepub.epub")
              os.system("/directoriodetubot/kepubifylinux '" + file_name + "'") # conversion to KEPUB
              context.bot.send_document(chat_id=update.message.chat_id, document=open(file_kepub, 'rb'), caption="Here your KEPUB!")
              pc = PyCloud('tuusuario@dominio.com', keyring.get_password("pCloud", "tuusuario@dominio.com"))
              pc.uploadfile(files=[file_kepub],path='/directorioenlanubedetuskepubs')
              os.remove(file_name)
              os.remove(file_kepub)
        else:
              update.message.reply_text("No es formato Epub")
    else:
        update.message.reply_text('Send me a file.epub I will convert it into KEPUB')

def main():
  updater = Updater(TOKEN, use_context=True)

  dp = updater.dispatcher

  dp.add_handler(MessageHandler(Filters.document, conversion))
  dp.add_handler(CommandHandler('start', start))
  dp.add_handler(CommandHandler('help', start))
  dp.add_handler(CommandHandler('hola',hola))
  dp.add_handler(MessageHandler(Filters.text, echo))

  # log all errors
  dp.add_error_handler(error)

  updater.start_polling()
  updater.idle()

if __name__ == '__main__':
    main()
```

{% hint style="success" %}
Espero que os sea de utilidad.
{% endhint %}
