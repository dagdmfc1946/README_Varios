# Ejecutables creados para pruebas:

## TELSY v01 (--onedir): 
Creado correctamente, ejecuta el servidor **(SE PONE EL LED EN VERDE)** y el navegador **(SE ABRE BIEN)**, se supone que está incluido el driver pero **NO SE EJECUTA**, debe ser por separado.
- **PD:** A veces se presentan problemas con el driver así se ejecute por separado, ya que al realziar una medición se cierra en algunos casos la terminal que ejecuta el driver y en dicho caso toca volver a ejecutarlo nuevamente. 

## TELSY v02 (--onedir): 
Creado correctamente, ejecuta el servidor **(SE PONE EL LED EN VERDE)**, se supone que el navegador debería abrirse, pero NO SE ABRE **(DEBE HACERSE MANUALMENTE)** y el driver también **DEBE EJECUTARSE POR SEPARADO**.
- **PD:** Falta realizar múltiples pruebas para comprobar el funcionamiento.

## TELSY v03 (--onedir):
Debe volver a crearse el ejecutable ya que presentó problemas al iniciar el servidor **(NO SE PONE EL LED EN VERDE)**, el navegador si se abre pero como no se inició el servidor queda en un buble de abrir-cerrar **(TIENE ALGÚN ERROR, IGUAL QUE CON LA EJECUCIÓN DEL SERVIDOR)**.
**(28/08/2025):** Logré por fin crear el ejecutable, Edwin ya revisó, me sugirió revisar el porqué los datos en algunas ocaciones tardan en visualizarse y/o porque en algunas ocasiones no muestra algunos datos (*NIBP y ECG (HR y RR)*).
- **PD:** Ya modifiqué el **autostart** para ejecutar **TELSY v03** automáticamente al iniciar la Raspberry.

## TELSY v04 (--onefile):
- Se logró crear un ejecutable de archivo único, pero tiene problemas ya que pareciera que si ejecuta el servidor y el navegador (porque realmente se observa eso en la aplicación), pero al realizar cualquier medición no toma datos, ni siquiera cuando se esta ejecutando paralelamente el driver.
- Otra forma de comprobar que no quedó bien el ejecutable es que el led NO enciende de color verde (por más de que en la aplicación si se ejecuta como tal la aplicación).

## TELSY v05 (--onedir):
Se creó otro ejecutable correctamente, al percer funciona bien, solo falta verificar con múltiples pruebas de funcionamiento.


---


## Ejecutables presentes en el escritorio:
Archivos **.sh** con acceso directo en el escritorio para ejecutar todo desde ahí.
- **PD:** Actualmente ya se encuentra modificado el archivo de **autostart** para abrir **startserver.sh**, **startsweb.sh** y **run_driver.sh**. Se han hecho varias pruebas y funciona todo correctamente. **De momento esto es lo más fiable.**
