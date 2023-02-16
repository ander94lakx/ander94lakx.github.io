---
title: "Cómo monitorizar dispositivos bluetooth cercanos con un solo comando"
date: 2023-02-15T20:00:00+01:00
tags: ["wireless", "bluetooth", "linux", "info-gathering", "bluetoothctl", "awk"]
draft: true
---

Me encanta Linux. Con la terminal te sientes poderoso. No es que sea un experto en ella, pero cuando te sabes manejar un poco con ella puedes realizar muchas cosas.

El otro día quise hacer un pequeño script para monitorizar los dispositivos bluetooth que tengo a mi alrededor. Lo veía como un ejercicio interesante para obtener datos, como saber cuántos dispositivos tengo cerca, ver que tipo de dispositivos son, cuantos dispositivos vienen y van, y ver que se puede hacer con esa información.

Mi instinto al querer hacer algo del estilo es hacer un script (putos programadores, siempre [reinventando la rueda](https://www.npmjs.com/package/is-odd)). El lenguaje de scripting con el que más comodo me siento es Python, por lo que pensé en usarlo para hacer esto. Después pensé que, en realidad, lo que quería hacer era usa la salida de un comando (`bluetoothctl`), hacer cuatro cosas y volcarlo a un archivo.

Entoces se me ocurrio que lo que deberia hacer es más un shell script que usar algo tan *overkill* como Python. Así, lo podia usar siempre, menos dependencias, etc.

Dandole una vuelta más, me planteé si realmente era necesario un script para esto. Lo mismo entre pipes, greps y transformaciones así podia hacer un *oneline* que pudiera meter en un alias y listo. A veces, cuando busco como obtener cierta info o como hacer ciertas tareas en Linux, los resultados de sitios como *Super User* muestran a gente poniendo locuras de comandos para poder hacer todo tipo de virguerías, por lo que pensé que lo mismo podia hacer yo, o al menos intentarlo. Por ello, hoy vengo a explicar como montar un sistema para generar un log con los dispositivos bluetooth que se van detectando.

Lo primero es lo primero. La herramienta para trabajar con bluetooth que he usado es `bluetoothctl`.

```bash
bluetoothctl
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-1.png)

Si se ejecuta, se ve que funciona en modo interactivo. Para escanear dispositivos simplemente hace falta usar el subcomando `scan on` y, con ello, se pone a detectar dispositivos que aparecen, desaparecen o cambian propiedades.

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-2.png)

Aquí viene el primer problema. Como quiero redirigir la salida de esto, el modo interactivo no me sirve. En Bash (y en otras shells como Zsh, la que uso yo), un `--` indica el fin de comandos, tras lo cual solo se puede pasar parametros. Por lo que, si se pone lo siguiente:

```bash
bluetoothctl -- scan on
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-3.png)

Ya devuelve todo por salida estándar y sin modo interactivo.

El siguiente paso es realizar alguna transformación. De primeras, me interesa cortar algunos parámetros y volcar esto y filtrar algunas lineas en funcion de su contenido. Para esto ya se que lo mejor es usar el avanzado pero intimidante `awk`. He visto algunos [videos](https://www.youtube.com/watch?v=W5kr7X7EG4o) sobre él pero no lo he usado nunca más allá de copiapegas. Simplemente por probar, he probado a usar la opcion para imprimir lo que devuelve, de la siguiente manera.

```bash
bluetoothctl -- scan on | awk '{print}'
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-4.png)

Mierda. Otro problema. No sale nada. Mirando en internet, veo que el motivo es un tema de buffers. Como la salida es continua, hasta que no se libera el buffer de `bluetoothctl` la salida no se redirige a `awk`, por lo que no sirve. También mirando en internet, veo que puedo solucionar esto con `stdbuf` y unos argumentos concretos.

```bash
stdbuf -oL bluetoothctl -- scan on | awk '{print}'
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-5.png)

(No voy a entrar en la expliación detallada de todos los comandos y argumentos, ya que nunca lo voy a poder hacer tan bien como `man`).

Bien, ahora si que funciona, así que podemos empezar a ponernos serios. Para mostrar solo ciertas partes, se puede usar la sintáxis de dólar, que permite seleccionar ciertos parametros en cada línea de entrada.

```bash
stdbuf -oL bluetoothctl -- scan on \ 
	| awk '{print $1 "," $3 "," $4}'
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-6.png)

Seleccionando el primero, el tercero y el cuarto, se muestran solamente esos campos. Por defecto, el separador es el espacio, pero si se quiere explicitar o usar cualquier otro, se puede indicar con `-F`.

```bash
stdbuf -oL bluetoothctl -- scan on \
	| awk -F'[ ]' '{print $1 "," $3 "," $4}'
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-7.png)

Sabiendo como coger campos, ahora lo que me interesa es coger solo ciertas lineas. Se pueden establecer condiciones para ciertas lineas. Se que las lineas que me interesan son las que tienen "Device" en el segundo parámetro. Para poner esto en `awk`, se expresa antes del comando `print` de la siguiente manera:

```bash
stdbuf -oL bluetoothctl -- scan on \
	| awk -F'[ ]' '$2 ~ /Device/  {print $1 "," $3 "," $4}'
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-8.png)

Incluso se pueden poner varias condiciones. En mi caso las lineas que me interesan son las que tengan un `[NEW]` o un `[DEL]`, que es lo que determina cuando se encuentra un dispositivo y cuando se deja de detectar. Esto junto a lo de "Device" limita perfectamente lo que intento monitorizar. Para poner esto, se pone con `&&` y limitando entre parentesis. Como lo que se usan son regex, se usa el single pipe (`|`) para decir que vale cualquiera de esos dos valores, quedando así:

```bash
stdbuf -oL bluetoothctl -- scan on \
	| awk -F'[ ]' '($2 ~ /Device/ && $1 ~ /[NEW]|[DEL]/) {print $1 "," $3 "," $4}'
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-11.png)

De momento ya va bastante bien, pero de poco me sirve un log si esto no tiene timestamps. `awk` es tan potente que puedes usar algunas fuciones dentro de él (no sé si esto es built-in de `awk` o no, pero alucinante). En este caso, añado a mi `print` una llamada a `strftime()` con el formato que me gusta, de la siguiente manera:

```bash
stdbuf -oL bluetoothctl -- scan on \
	| awk -F'[ ]' '($2 ~ /Device/ && $1 ~ /[NEW]|[DEL]/)  {print strftime("%Y/%m/%d-%H:%M:%S-%Z", systime()) "," $1 "," $3 "," $4}'
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-10.png)

A estas alturas ya estoy flipando, poque la de lineas de script de Python que me he ahorrado con esto. Ya, lo ultimo que me falta es volcarlo a un archivo.

Algo que me interesaba era volcarlo a un archivo pero ir viendo al mismo tiempo la salida. Para esto `tee` viene perfecto. Pasamos la salida de `awk` a `tee` con un pipe y listo: 

```bash
stdbuf -oL bluetoothctl -- scan on \
	| awk -F'[ ]' '($2 ~ /Device/ && $1 ~ /[NEW]|[DEL]/)  {print strftime("%Y/%m/%d-%H:%M:%S-%Z", systime()) "," $1 "," $3 "," $4}' | tee -a bluetooth_scan_log.txt
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-12.png)

Mierda. No funciona. Pero espera, esto ya nos suena. Tenemos entre `awk` y `tee` el mismo problema que teniamos con `bluetoothctl` y `awk`, así que probamos a solucionarlo de la misma manera y listo:

```bash
stdbuf -oL bluetoothctl -- scan on \
	| stdbuf -oL awk -F'[ ]' '($2 ~ /Device/ && $1 ~ /[NEW]|[DEL]/)  {print strftime("%Y/%m/%d-%H:%M:%S-%Z", systime()) "," $1 "," $3 "," $4}' \
	| tee -a bluetooth_scan_log.txt
```

![Bluetooth oneliner](/static/images/bt-oneline/bt-oneline-13.png)

Y listo. Ya tenemos todo el sistema montado. Una vez hecho, he visto que hay varios detalles que corregir, como que quizas coger justo esos parametros no es buena idea ya que el cuarto parametro puede cortarse con espacios, o que igual no me interesa filtrar tanta información.

Pero, de todas maneras, eso no es lo importante. Lo esencial es que, con unos comandos, un poco de prueba error y nuestros amigos Google y `man`, se puede montar muchas cosas, sin tener que hacer scripts ni nada. Esto tiene varias ventajas:

- Usar herramientas estándar, que estan disponibles en la mayoría de distribuciones Linux, lo que asegura poder usarlo en cualquier lado.

- Menos scripting. Que no me malinterpretéis, me encanta programar, pero para que hacerlo si ya hay comandos solidos y probados que permiten hacerlo. No hay que reinventar la rueda, *Keep It Simple, Stupid!*

- Poder ponerlo como alias (o función *oneline* si las comillas dan muchos problemas) en nuestros `.bashrc` (o `.zshrc` en mi caso). Ejecutar esto escribiendo una sola palabra te hace sentir muy poderoso.

Y esto es todo. Sé que, y me he dado cuenta mientras lo hacía, de que hay muchas maneras de mejorar esto. Algunas las he visto, de otras no me habré dado ni cuenta. Mi intención con esto es simplemente mostrar la belleza y la magia de usar herramientas estándar y la terminal, y lo potentes que son este tipo de herramientas para hacer muchas cosas que, al final, no necesitan ser programadas.

Siempre que vayáis hacer alguna cosa del estilo, plantearos una cosa: alguien sería capaz de hacer esto con un comando? Si la respuesta es sí, buscadlo y, si no lo encontráis, abrid la terminal!

*Happy Hacking!*
