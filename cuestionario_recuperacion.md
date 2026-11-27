## Responde las siguientes preguntas

---

**1. ¿Qué datos se habrían perdido si solo hubieras usado el respaldo de la Parte A?**

*Respuesta:*

Se habrían perdido los cambios realizados después del respaldo, como los nuevos registros agregados, modificaciones y eliminaciones de datos.

---

**2. ¿Para qué sirve la opción `--master-data=2`?**

*Respuesta:*

Sirve para guardar la información de la posición del binlog en el respaldo, permitiendo saber desde dónde se pueden recuperar los cambios posteriores. El número `2` hace que la instrucción se guarde como comentario en el archivo SQL.

---

**3. ¿Por qué es necesario activar el binlog antes de que ocurra el error?**

*Respuesta:*

Porque el binlog registra los cambios realizados en la base de datos. Si se activa después del error, los cambios anteriores no estarán registrados y no podrán recuperarse.

---

**4. ¿Por qué se esperaron 5 segundos antes y después de anotar la hora?**

*Respuesta:*

Para dar un intervalo de tiempo en el que se registraran los cambios en el binlog y poder identificar con mayor precisión los movimientos realizados antes y después del error.

---
