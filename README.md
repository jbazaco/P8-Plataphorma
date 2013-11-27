Plataphorma
===========

Meteor pet project created to teach my students the following Meteor functionality: 

* Collection's publish/subscribe 
* Deps.autorun 
* Meteor.methods/call 
* Integration of non-Meteor code in compatibility folder (HTML5 games Alien Invasion and Froot Wars)
* Usage of allow to control client access to collections

![ScreenShot](/screenshot.png)


Plataphorma offers the possibility to run 2 different HTML5 games: Alien Invasion and Froot Wars. 

On the right side of the screen the best players of each game are shown, updated in real time each time a signed in player finishes a game. If no game is selected, the best players overall are shown.

On the left side of the screen a chatroom for the current game is available. Only signed in users can post messages. If no game is selected a general chatroom is shown.

The original code of the two HTML5 games integrated in this project is available here:
* Alien Invasion: https://github.com/cykod/AlienInvasion
* Froot Wars: http://www.wrox.com/WileyCDA/WroxTitle/Professional-HTML5-Mobile-Game-Development.productCd-1118301323,descCd-DOWNLOAD.html

Bootstrap style (file bootstrap.min.css) provided by http://bootswatch.com


Running the project
-------------------

A live version of this code is running here: http://plataphorma.meteor.com

To run the project locally, clone the repo and run ```meteor``` inside it. You can see in .meteor/packages that this Meteor project uses these packages:
* ```meteor remove autopublish```
* ```meteor remove insecure```
* ```meteor add bootstrap```
* ```meteor add accounts-ui```
* ```meteor add accounts-password```

Preguntas
-------------------
1) Click en botón de juego:
En client/client.js línea 112 y sucesivas.
La siguiente línea modifica la variable "current_game" almacenada en session, que es una fuente reactiva, lo que provoca la ejecución del código que utilice esa variable.
```Session.set("current_game", game._id);```


2) Se escribe mensaje en chat sin estar autenticado
En client/client.js línea 85 y sucesivas.
Como en la línea 87 se comprueba si hay un usuario registrado, viendo si tiene un ID ```if (Meteor.userId()){ ```, al no estar registrado entra en el else y se ejecuta ```$("#login-error").show();``` que muestra un mensaje de error.

3) Se escribe mensaje en chat estando autenticado
En client/client.js línea 85 y sucesivas.
En este caso, en la líne 87 cumple la condición por lo que entra al "if". Con ```Messages.insert``` almacena en la colección Messages el mensaje que ha escrito el usuario, el mensaje contiene el ID del usuario, la fecha en la que se escribió, para que chat es ( ```Session.get("current_game")``` ) y el contenido del mensaje.
Como se cambia el contenido de la colección Messages se ejecuta el código en client/client.js línea 71 y sucesivas ya que Messages es una fuente reactiva, y en la línea 73 se está buscando en dicha colección ( ```var messagesColl =  Messages.find({}, { sort: { time: -1 }});``` )


4) Se termina partida con puntuación máxima sin estar autenticado
En client/compatibility/alien-invasion/game.js línea 104 y en la línea 114 (para el caso del alienInvasion)
```Meteor.call("matchFinish", Session.get("current_game"), Game.points);```
Invoca matchFinish pasándole como argumentos Session.get("current_game") y Game.points (la puntuación).
El código de matchFInish está en server/server.js en la línea 47 y sucesivas, y en la línea 50 no entra al "if" al no estar autenticado y no hace nada.

5) Se termina partida con puntuación máxima estando autenticado
Igual que el 4 con la diferencia de que ahora sí entra al "if" al estar autenticado, y con Matches.insert almacena en la colección Matches la puntuación de la partida. Eso provoca que al cambiar la colección Matches se ejecute el código en client/client.js línea 157 y sucesivas, donde con ```var matches =  Matches.find({}, {limit:5, sort: {points:-1}});``` se obtienen las 5 mejores puntuaciones y se muestran en el navegador al insertarlos en el index.html

6) ¿Qué sale en consola cuando te autenticas?
Muestra el current_user que tenías(null) y el que tienes ahora.
El código se ejecuta en client/clien.js en la línea 57 y sucesivas. Debido al Deps.autorun se ejecuta el código si se produce alguna cambio en una fuente de datos reactiva que se evalúe dentro, en este caso debido a Meteor.userId(); se ejecuta el código.

7) ¿Qué sale en consola cuando te sales?
Muestra el current_user que tenías y el que tienes ahora(null).
Por lo mismo que en el 6.
