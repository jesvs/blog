---
title: Clon Chino de Arduino Nano
author: Jesús Sánchez
type: post
date: 2015-10-02T08:55:34+00:00
url: /clon-de-arduino-nano/
featured_image: /wp-content/uploads/2015/10/wpid-wp-1443775976108-709x510.jpeg
categories:
  - Arduino
tags:
  - Arduino
  - CH340G
  - Drivers
  - FTDI

---
Ahora que hice el post de prueba recordé que este clon chino de Arduino necesita drivers específicos para programarlo.

El chip CH340G se convirtió en el USB a Serial _de facto_ entre los clones de Arduino, luego de que FTDI implementara _&#8220;medidas de protección al usuario&#8221;_ en el driver oficial de Windows para anular los chips FT232RL piratas.

Vaya tiempos que nos tocó vivir, medidas de seguridad en un driver para contrarrestar piratería de microchips.

Drivers para el chip CH34xG: Windows [CH341SER][1] | Mac [CH341SER_MAC][2].

 [1]: https://blog.jesvs.com/wp-content/uploads/2015/10/CH341SER.zip
 [2]: https://blog.jesvs.com/wp-content/uploads/2015/10/CH341SER_MAC.zip