PAV - P5: síntesis musical polifónica
=====================================

synth work/seno.orc work/doremi.sco work/efecto.wav -e work/effects.orc
ha de recordar posar el numero de l efecte a  doremi.sco
el numero esta definit a effects.sco


Obtenga su copia del repositorio de la práctica accediendo a [Práctica 5](https://github.com/albino-pav/P5)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde que
los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A modo de memoria de la práctica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

Ejercicios.
-----------

### Envolvente ADSR.

Tomando como modelo un instrumento sencillo (puede usar el InstrumentDumb), genere cuatro instrumentos que
permitan visualizar el funcionamiento de la curva ADSR.

* Un instrumento con una envolvente ADSR genérica, para el que se aprecie con claridad cada uno de sus
  parámetros: ataque (A), caída (D), mantenimiento (S) y liberación (R).

  <img src="img/1.PNG" width="500" align="center">
  <img src="img/2.PNG" width="500" align="center">

* Un instrumento *percusivo*, como una guitarra o un piano, en el que el sonido tenga un ataque rápido, no haya mantenimiemto y el sonido se apague lentamente.
  - Para un instrumento de este tipo, tenemos dos situaciones posibles:
  En el fichero.orc correspondiente escribimos los siguientes parámetros:
  <img src="img/4.PNG" width="500" align="center">
    * El intérprete mantiene la nota *pulsada* hasta su completa extinción.
    <img src="img/5.PNG" width="500" align="center">
    <img src="img/3.PNG" width="500" align="center">
    * El intérprete da por finalizada la nota antes de su completa extinción, iniciándose una disminución  abrupta del sonido hasta su finalización.
    <img src="img/21.PNG" width="500" align="center">
    <img src="img/6.PNG" width="500" align="center">
  - Debera representar en esta memoria **ambos** posibles finales de la nota.
* Un instrumento *plano*, como los de cuerdas frotadas (violines y semejantes) o algunos de viento. En  ellos, el ataque es relativamente rápido hasta alcanzar el nivel de mantenimiento (sin sobrecarga), y la liberación también es bastante rápida.
<img src="img/8.PNG" width="500" align="center">
<img src="img/9.PNG" width="500" align="center">
<img src="img/7.PNG" width="500" align="center">

Para los cuatro casos, deberá incluir una gráfica en la que se visualice claramente la curva ADSR. Deberá
añadir la información necesaria para su correcta interpretación, aunque esa información puede reducirse a
colocar etiquetas y títulos adecuados en la propia gráfica (se valorará positivamente esta alternativa).

### Instrumentos Dumb y Seno.

Implemente el instrumento `Seno` tomando como modelo el `InstrumentDumb`. La señal **deberá** formarse
mediante búsqueda de los valores en una tabla.

- Incluya, a continuación, el código del fichero `seno.cpp` con los métodos de la clase Seno.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
#include <iostream>
#include <math.h>
#include "seno.h"
#include "keyvalue.h"

#include <stdlib.h>

using namespace upc;
using namespace std;

Seno::Seno(const std::string &param)
  : adsr(SamplingRate, param) {
  bActive = false;
  x.resize(BSIZE);

  /*
    You can use the class keyvalue to parse "param" and configure your instrument.
    Take a Look at keyvalue.h
  */
  KeyValue kv(param);
  int N;

  if (!kv.to_int("N",N))
    N = 40; //default value

  //Create a tbl with one period of a sinusoidal wave
  tbl.resize(N);
  float phase = 0, step = 2 * M_PI /(float) N;
  for (int i=0; i < N ; ++i) {
    tbl[i] = sin(phase);
    phase += step;
  }
}


void Seno::command(long cmd, long note, long vel) {
  if (cmd == 9) {		//'Key' pressed: attack begins
    bActive = true;
    adsr.start();
    phase = 0;
    float f0 = 440.0*pow(2,(((float)note-69.0)/12.0));
    nota = f0/SamplingRate;
	  A = vel / 127.;
    step = tbl.size()*nota;
  }
  else if (cmd == 8) {	//'Key' released: sustain ends, release begins
    adsr.stop();
  }
  else if (cmd == 0) {	//Sound extinguished without waiting for release to end
    adsr.end();
  }
}


const vector<float> & Seno::synthesize() {
  if (not adsr.active()) {
    x.assign(x.size(), 0);
    bActive = false;
    return x;
  }
  else if (not bActive)
    return x;

  for (unsigned int i=0; i<x.size(); ++i) {
    if (round(phase*step) == tbl.size()) {
      phase = 0;
    }
    x[i] = A*tbl[round(phase*step)];
    phase=phase+1;
  }
  adsr(x); //apply envelope to x and update internal status of ADSR

  return x;
}

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


- Explique qué método se ha seguido para asignar un valor a la señal a partir de los contenidos en la tabla,
  e incluya una gráfica en la que se vean claramente (use pelotitas en lugar de líneas) los valores de la
  tabla y los de la señal generada.

  Cada nota, cuando esta es tocada, tiene definida una velocidad. Esta es usada para poder acceder a los valores correspondientes de la tabla (la cual se va recorriendo con la variable step del código). A veces se usa interpolación para poder llegar a dicha velocidad ya que no siempre se tiene un valor en la tabla que corresponda a ese índice.

- Si ha implementado la síntesis por tabla almacenada en fichero externo, incluya a continuación el código
  del método `command()`.
  No se ha implementado.

### Efectos sonoros.

- Incluya dos gráficas en las que se vean, claramente, el efecto del trémolo y el vibrato sobre una señal
  sinusoidal. Deberá explicar detalladamente cómo se manifiestan los parámetros del efecto (frecuencia e
  índice de modulación) en la señal generada (se valorará que la explicación esté contenida en las propias
  gráficas, sin necesidad de *literatura*).

  Efecto vibrato:

  <img src="img/13.PNG" width="500" align="center">
  <img src="img/12.PNG" width="500" align="center">
  <img src="img/11.PNG" width="500" align="center">
  <img src="img/10.PNG" width="500" align="center">

  En la gráfica vemos que aparece una variación periódica en la frecuencia como consecuencia del efecto vibrato. Se han especificado los parámetros en las imágenes anterior, como la I y la frecuencia de muestreo.  

  Efecto tremolo:

  <img src="img/15.PNG" width="500" align="center">
  <img src="img/16.PNG" width="500" align="center">
  <img src="img/17.PNG" width="500" align="center">
  <img src="img/14.PNG" width="500" align="center">

  El efecto de trémolo consiste en una modulación de amplitud de la señal de entrada. La frecuencia se mantiene constante pero por otra parte vemos que la amplitud de la señal fluctua periódicamente. Hemos escuchado el audio para ver que cambios se habian realizado y hemos detectado que la intensidad del sonido variaba pero que el tono era el mismo. Como vemos en la captura de los parámetros del tremolo, aplicamos una amplitud mínima de 0.2. Seguimos visualizando el seno así como sus parámetros de ataque caída, etc.

- Si ha generado algún efecto por su cuenta, explique en qué consiste, cómo lo ha implementado y qué resultado ha producido. Incluya, en el directorio `work/ejemplos`, los ficheros necesarios para apreciar el efecto, e indique, a continuación, la orden necesaria para generar los ficheros de audio usando el programa `synth`.

  No hemos generado nada.

### Síntesis FM.

Construya un instrumento de síntesis FM, según las explicaciones contenidas en el enunciado y el artículo
de [John M. Chowning](https://ccrma.stanford.edu/sites/default/files/user/jc/fm_synthesispaper-2.pdf). El
instrumento usará como parámetros **básicos** los números `N1` y `N2`, y el índice de modulación `I`, que
deberá venir expresado en semitonos.

- Use el instrumento para generar un vibrato de *parámetros razonables* e incluya una gráfica en la que se
  vea, claramente, la correspondencia entre los valores `N1`, `N2` e `I` con la señal obtenida.
- Use el instrumento para generar un sonido tipo clarinete y otro tipo campana. Tome los parámetros del
  sonido (N1, N2 e I) y de la envolvente ADSR del citado artículo. Con estos sonidos, genere sendas escalas
  diatónicas (fichero `doremi.sco`) y ponga el resultado en los ficheros `work/doremi/clarinete.wav` y
  `work/doremi/campana.work`.

  Parámetros clarinete.wav:

  En el documento .orc de campana describimos los parámetros utilizados paraa simular el sonido del clarinete juntamente con el archivo doremi.osc.
  (Enganxar foto del clarinete.orc)
  <img src="img/14.PNG" width="500" align="center">

  (Para simular el sonido del clarinete, usamos los siguientes valores proporcionados en el paper: N1 = 100, N2 = 20, I = 0.5 y para las componentes del ADSR: Attack = 0.04, D = 0, S = 0.5, R = 0.07.)

  Parámetros campana.wav:

  De la misma manera que el clarinete, en la foto vemos que el .orc correspondiente a la campana contiene los parámetros necesarios para simularla.
  (Enganxar foto del clarinete.orc)
  <img src="img/14.PNG" width="500" align="center">



  (Para simular el sonido de la campana, usamos los siguientes valores proporcionados en el paper: N1 = 100, N2 = 140, I = 1 y para las componentes del ADSR: Attack = 0.03, D = 2, S = 0, R = 0.)

  * También puede colgar en el directorio work/doremi otras escalas usando sonidos *interesantes*. Por
    ejemplo, violines, pianos, percusiones, espadas láser de la
	[Guerra de las Galaxias](https://www.starwars.com/), etc.

### Orquestación usando el programa synth.

Use el programa `synth` para generar canciones a partir de su partitura MIDI. Como mínimo, deberá incluir la
*orquestación* de la canción *You've got a friend in me* (fichero `ToyStory_A_Friend_in_me.sco`) del genial
[Randy Newman](https://open.spotify.com/artist/3HQyFCFFfJO3KKBlUfZsyW/about).

- En este triste arreglo, la pista 1 corresponde al instrumento solista (puede ser un piano, flautas,
  violines, etc.), y la 2 al bajo (bajo eléctrico, contrabajo, tuba, etc.).
- Coloque el resultado, junto con los ficheros necesarios para generarlo, en el directorio `work/music`.
- Indique, a continuación, la orden necesaria para generar la señal (suponiendo que todos los archivos
  necesarios están en directorio indicado).

  <img src="img/18.PNG" width="500" align="center">
  <img src="img/19.PNG" width="500" align="center">

  synth Toy_Story.orc -e effects.orc ToyStory_A_Friend_in_meeffects.sco music/ToyStory_A_Friend_in_meeffects.wav

También puede orquestar otros temas más complejos, como la banda sonora de *Hawaii5-0* o el villacinco de
John Lennon *Happy Xmas (War Is Over)* (fichero `The_Christmas_Song_Lennon.sco`), o cualquier otra canción
de su agrado o composición. Se valorará la riqueza instrumental, su modelado y el resultado final.
- Coloque los ficheros generados, junto a sus ficheros `score`, `instruments` y `efffects`, en el directorio `work/music`.
- Indique, a continuación, la orden necesaria para generar cada una de las señales usando los distintos
ficheros.

  Usando la canción Hawaii5-0:

  <img src="img/20.PNG" width="500" align="center">
  synth Hawaii.orc -e effects.orc Hawaiieffects.sco music/Hawaiieffects.wav
