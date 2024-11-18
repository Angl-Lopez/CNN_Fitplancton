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

Este modelo fue entrenado usando los siguientes parámetros:

Para  hacer un balanceo de los pesos de clases y combatir el desbalance de clases.

```
from sklearn.utils.class_weight import compute_class_weight

class_weight = compute_class_weight(class_weight='balanced',
                                    classes=np.unique(cls_train),
                                    y=cls_train)
class_weight = dict(enumerate(class_weight))
```

Callbacks para evitar el sobre ajuste y el underfitting.

```
from tensorflow.keras.callbacks import ReduceLROnPlateau, EarlyStopping, TensorBoard, ModelCheckpoint

early_stopping = EarlyStopping(monitor='val_loss', patience=2, restore_best_weights=True)
reduce_lr = ReduceLROnPlateau(monitor='val_loss', factor=0.2, patience=2, min_lr=1e-7)

```

Top 10 de exactitud:

```
def top_10_accuracy(y_true, y_pred):
    return tf.keras.metrics.top_k_categorical_accuracy(y_true, y_pred, k=10)
```

Entrenamiento inicial del modelo:

```
starting_epoch = 0
epochs_top_layers = 10
epochs_fine_tuning = 50

from tensorflow.keras.regularizers import l2
# Crear el modelo base de InceptionV3
base_model = InceptionV3(weights='imagenet', include_top=False)
x = base_model.output
x = GlobalAveragePooling2D()(x)
x = Dense(1024, activation='relu', kernel_regularizer=l2(0.001))(x)  # Regularización L2
x = Dropout(0.3)(x)  # Dropout del 30%


# Actualizar la capa de salida
predictions = Dense(num_classes, activation='softmax')(x)
model = Model(inputs=base_model.input, outputs=predictions)

# Congelar todas las capas de la base del modelo excepto por el primer bloque Inception
for layer in base_model.layers[:-249]:
    layer.trainable = False
for layer in base_model.layers[-249:]:
    layer.trainable = True

# Compilar el modelo
model.compile(optimizer=Adam(1e-4), loss='categorical_crossentropy', metrics=['categorical_accuracy', top_10_accuracy])

# Entrenar el modelo (solo las capas superiores)
history = model.fit(train_generator,
                    epochs=epochs_top_layers+starting_epoch,
                    validation_data=validation_generator,
                    class_weight = class_weight,
                    callbacks = [early_stopping, reduce_lr],
                    shuffle=True)

```

Segunda fase del entrenamiento:

```
from tensorflow.keras.callbacks import ModelCheckpoint, TensorBoard
from datetime import datetime

# Descongelar todas las capas
model.trainable = True

# Recompilar el modelo con una tasa de aprendizaje más baja
model.compile(optimizer=Adam(learning_rate=1e-5), loss='categorical_crossentropy', metrics=['categorical_accuracy', top_10_accuracy])

# Continuar entrenando el modelo (fine-tuning)
history = model.fit(train_generator,
                    initial_epoch=starting_epoch+epochs_top_layers,
                    epochs=starting_epoch+epochs_top_layers+epochs_fine_tuning,
                    validation_data=validation_generator,
                    class_weight = class_weight,
                    callbacks = [early_stopping, reduce_lr],
                    shuffle=True)
```
Tercera y última fase del entrenamiento:

```
from tensorflow.keras.callbacks import ModelCheckpoint, TensorBoard
from datetime import datetime

# Descongelar todas las capas
model.trainable = True

# Recompilar el modelo con una tasa de aprendizaje más baja
model.compile(optimizer=Adam(learning_rate=1e-6), loss='categorical_crossentropy', metrics=['categorical_accuracy'])

# Continuar entrenando el modelo (fine-tuning)
history = model.fit(train_generator,
                    epochs=5,
                    validation_data=validation_generator,
                    class_weight = class_weight,
                    callbacks = [early_stopping, reduce_lr],
                    shuffle=True)
```

Este entrenamiento dio como resultados la siguiente matriz de confusión.

![image](https://github.com/user-attachments/assets/7073a052-e15a-4df9-b3f8-91a5ea4ff0b7)
