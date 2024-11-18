# Fitosense Clasificador 🦠

<p align="center">
  <img src="https://github.com/user-attachments/assets/6f8c79b2-be89-424b-bce4-22d0bc2abfb1" alt="image" width="300">
</p>


Este proyecto fue realizado con la finalidad de crear un modelo de clasifiación de fitplancton usando conjuntos de datos abiertos en una Universidad usando una red neuronal convolucional.

Para hacer el acceso a este modelo más sencillo. Se hizo una aplicación web en Anvil. 

![image](https://github.com/user-attachments/assets/e23247c6-1811-4071-98cd-fea846157d58)

## ¿Cómo usar?

Para usar la aplicación es un proceso muy sencillo, dirigite a la página de Clasificador:

* Sube tus Imágenes: Haz clic en el botón Subir Imagen ⬆️ para cargar todas las imágenes que quieras clasificar.
* Ejecuta el Clasificador: Una vez que hayas subido las imágenes, presiona el Inicar clasificador ▶️ para que el clasificador comience a trabajar.
* Te regresará el top 10 de los géneros de fitplancton con mayor similitud a tu imagen.

**Nota**: Puedes borrar alguna imagen o todas las imágenes en cualquier momento usando los botones grises debajo.


<p align="center">
  <img src="https://github.com/user-attachments/assets/5f7ed3b1-8be1-4326-bb5b-06406308d721" alt="Recording GIF"  width="800">
</p>

## Chatbot

Fitosense es un chatbot que te ayuda a clasificar tus imágenes de fitoplancton. Puedes preguntarle lo que quieras acerca de los distintos géneros de fitplancton e información relacionada con la biología.

**Consejo**: ¡Pregúntale acerca de tus resultados!


<p align="center">
  <img src="https://github.com/user-attachments/assets/f60d7f8c-5e88-4bf3-b071-09ee5bae1415" alt="image">
</p>

## Biblioteca

Puedes accerder a la información de todos los géneros que nuestro modelo es capaz de clasificar gracias al apartado de biblioteca donde podrás verlos todos o filtrar según su letra inicial.

<p align="center">
  <img src="https://github.com/user-attachments/assets/fa64a7ac-912d-4eab-8870-47f615f9c09f" alt="Recording GIF"  width="800">
</p>

## Manual de Usuario

Este apartado es una guía para que el usuario pueda revisar cómo puede hacer y revisar sus clasificaciones dentro de la aplicaciónu y cómo puede mejorar sus resultados.

![image](https://github.com/user-attachments/assets/b3678599-52f8-492b-9e5b-e117db56a4cb)


# Conjunto de datos

Para el modelo, se consiguieron las imágenes de 3 fuentes y se hizo una distribución en 70% para entrenamiento, 15% validación y 15% prueba. MBLWHOI Library y IFCB Dashboard son imágenes distribuidas por el Instituto Oceanográfico de WoodHole (WHOI) y son tomadas con un Imaging FlowCytobot (IFCB) que hace el proceso de extracción y toma de imágenes de muestras de fitplancton automáticamente. Las imágenes del IFCB son tomadas con un láser de alta precisión, no obstante, las imágenes tomadas no son tan similares a las que se toman en laboratorio. Es por eso que se hizo un conjunto de PlanktonNet, un foro de biología que se dedica a clasificar imágenes de microorganismos y que tiene una biblioteca abierta.

<p align="center">
  <img src="https://github.com/user-attachments/assets/1381e330-ce93-4b97-a6b9-e085d2ee5cca" alt="Segunda imagen">
</p>

Este modelo fue entrenado con un total de 586 159 imágenes distribuidas en 72 géneros de fitplancton. A pesar de tener pocas imágenes en varias clasificaciones. No se eliminaron pues el propósito de este conjunto es obtener una gran cantidad de clasificaciones que sirvan como guía para clasificar géneros de fitplancton.

<p align="center">
  <img src="https://github.com/user-attachments/assets/a34df224-4dc6-434e-a445-9debff6978b8" alt="Primera imagen">
</p>

# Entrenamiento del modelo

Se hicieron 3 propuestas para el modelado:

* **Modelo único:** 1 modelo con las 72 clasificaciones de fitplancton.
* **Propuestas secuencial:** 5 modelos organizados por tamaño el cual contiene una clasificación "unknwon" la cual conteiene las clasificaciones que no pertenecen a ese modelo. En caso de que clasifique como unknwon pasará al siguiente modelo hasta encontrar una clasificación que no se unknwon.
* **Propuesta jerárquica:** 8 modelos en total, 7 submodelos agrupados a partir de sus características visuales. 1 modelo general que predice a que grupo se parece más la imagen y el submodelo predice a qué género de fitplancton tiene mayor similitud.

A partir de las pruebas realziadas, la propuesta seleccionada fue el modelo único con una precisión ponderada de 95% respecto al conjunto de prueba.

![image](https://github.com/user-attachments/assets/7073a052-e15a-4df9-b3f8-91a5ea4ff0b7)
