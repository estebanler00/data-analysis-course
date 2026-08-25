# Ejercicios — Clase 20: Business Intelligence, Métricas, KPIs e Introducción a Tableau

Estos ejercicios usan `ventas_kiosco.csv` (300 registros: 150 compras de febrero y 150 de marzo). Los primeros ejercicios son de cálculo, sin herramienta. Los últimos dos son el primer contacto con Tableau Public.

## Ejercicio 1 — Ventas totales de marzo

Filtra el dataset para quedarte solo con marzo y calcula las ventas totales del mes. Compara el resultado con el de febrero, que se vio en clase ($424.650). Antes de calcular, revisa si hay algo en los datos que debería excluirse de una venta "neta" (devoluciones, cancelaciones) — si no hay ninguna columna que indique eso, acláralo como supuesto en tu respuesta.

## Ejercicio 2 — Ticket promedio de marzo

Con el resultado del Ejercicio 1 y la cantidad de compras de marzo, calcula el ticket promedio. Compáralo con el de febrero (~$2.831). Calcula también la mediana de las ventas de marzo y compara ambos valores — si son muy distintos, explica qué podría estar causando esa diferencia.

## Ejercicio 3 — Clientes únicos y clientes nuevos

Cuenta cuántos clientes únicos compraron en marzo. Identifica cuáles no habían comprado en febrero (clientes nuevos). Además, calcula cuántas compras hizo cada cliente en el período completo (`groupby("cliente").size()`) e indica si el negocio depende más de clientes que vuelven o de clientes que compran una sola vez.

## Ejercicio 4 — Tasa de churn de febrero a marzo

De los clientes que compraron en febrero, cuenta cuántos no volvieron a comprar en marzo. Calcula la tasa de churn. Antes de calcularla, escribe en una oración qué ventana de tiempo estás asumiendo como "cliente perdido" y por qué te parece razonable para este negocio.

## Ejercicio 5 — Lectura de negocio

Con los cuatro resultados anteriores organizados en una tabla, escribe de tres a cinco oraciones respondiendo: ¿el negocio mejoró de febrero a marzo? La respuesta debe apoyarse en los números, no en una impresión general, y debe mencionar al menos un indicador que contradiga o matice la conclusión principal (como se vio en clase con el ejemplo de churn vs. clientes nuevos).

## Ejercicio 6 — Primer contacto con Tableau Public (edición web)

1. Ingresa a [public.tableau.com](https://public.tableau.com) y crea un perfil gratuito.
2. Buscar la opción de crear una visualización directamente en el navegador (Tableau Public ofrece una edición de autoría web, sin necesidad de descargar nada).
3. Conecta `ventas_kiosco.csv` como origen de datos (subida de archivo).
4. Crea un gráfico de barras con el total de ventas (`cantidad × precio_unitario`) por mes.
5. Guarda el resultado en el perfil.

Una aclaración importante: en Tableau Public, todo lo que se guarda queda **público** —cualquiera con el enlace puede verlo. Para este ejercicio no es un problema (es un dataset de práctica, no datos sensibles), pero es una diferencia real respecto a Power BI Web, donde el trabajo puede quedar privado. Conviene mencionarlo en clase para que quede claro antes de que alguien cargue algo por error.

## Ejercicio 7 — Una vista simple en Tableau

Usando la misma conexión del Ejercicio 6, crea una hoja que muestre el total de ventas de marzo como un único número grande (arrastra el campo de ventas al centro de la vista y cambia el tipo de marca a "Texto"). Confirma que el número coincide con el que se calculó manualmente en el Ejercicio 1.