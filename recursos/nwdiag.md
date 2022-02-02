# ​nwdiag

> Enero - 2022

****[**nwdiag**](https://github.com/blockdiag/nwdiag) es una utilidad de línea de comando para generar diagramas básicos de red, de ocupación de racks y de paquetes.

Para su instalación, desde la shell correspondiente a nuestro sistema operativo ejecutamos:

**`sudo pip install nwdiag`**

Una vez instalado necesitaremos un fichero de texto en el que incluiremos las sentencias con las instrucciones para general el diagrama que queremos.

Por ejemplo, para crear el diagrama de red que vemos a continuación utilizaremos un fichero <mark style="color:blue;">simple.diag</mark> con el contenido siguiente:

{% code title="simple.diag" %}
```
nwdiag {
  network dmz {
      address = "210.x.x.x/24"

      web01 [address = "210.x.x.1"];
      web02 [address = "210.x.x.2"];
  }
  network internal {
      address = "172.x.x.x/24";

      web01 [address = "172.x.x.1"];
      web02 [address = "172.x.x.2"];
      db01;
      db02;
  }
}
```
{% endcode %}

Una vez generado el fichero, por ejemplo ejecutando `touch simple.diag` para su creación y editándolo para incluir contenido con nano `simple.diag`; ejecutaremos

**`nwdiag simple.diag`**

y el resultado será un fichero <mark style="color:blue;">simple.png</mark> en la misma ruta con la imagen resultado.

![Resultado de ejecutar nwdiag simple.diag](<../.gitbook/assets/image (3).png>)

Como resultado de la instalación de nwdiag tendremos dos comandos adicionales, **rackdiag** y **packetdiag**, para generar respectivamente diagramas de racks con sus elementos y diagramas de paquetes.

A continuación un ejemplo con el diagrama de tipo Rack.



{% tabs %}
{% tab title="Diagrama Rack" %}
![Resultado de ejecutar rackdiag simplerack.diag](<../.gitbook/assets/image (5).png>)
{% endtab %}

{% tab title="Fichero Rack" %}
{% code title="simplerack.diag" %}
```
rackdiag {
  16U;
  1: UPS [2U];
  3: DB Server;
  4: Web Server;
  5: Web Server;
  6: Web Server;
  7: Load Balancer;
  8: L3 Switch;
}
```
{% endcode %}
{% endtab %}
{% endtabs %}
