# GRAFICACION-2D

# Unidad 2: Graficación por Computadora

## 2.1. Transformación Bidimensional

Las transformaciones bidimensionales son operaciones geométricas que permiten crear una nueva figura a partir de una previamente dada. Se utilizan para alterar orientación, tamaño y forma de objetos en el plano. Las transformaciones básicas son: traslación, escalamiento, rotación y sesgado.

<img width="712" height="709" alt="cambios" src="https://github.com/user-attachments/assets/ff22fac2-6e10-4168-aa34-8f0f5272294e" />

---

### 2.1.1. Traslación

La traslación es una transformación de *cuerpo rígido* que mueve un objeto en línea recta de una posición a otra, sin rotarlo ni deformarlo.

Se obtiene sumando las distancias de traslación tx y ty a las coordenadas originales (x, y):
x' = x + tx
y' = y + ty
- El par (tx, ty) se llama *vector de traslación* o *vector de cambio*.
- Las distancias pueden ser positivas, negativas o cero.

---

### 2.1.2. Escalamiento

El escalamiento altera el *tamaño* de un objeto multiplicando sus coordenadas por factores de escala:
x' = x · Sx
y' = y · Sy
Tipos de escalamiento:

| Tipo | Descripción |
|------|-------------|
| *Uniforme* | Sx = Sy → cambia el tamaño pero no la forma |
| *Diferencial* | Sx ≠ Sy → produce distorsión en la forma |

> Esta transformación logra el efecto *Zoom*. El escalamiento se hace respecto a un punto fijo de referencia.

---

### 2.1.3. Rotación

La rotación mueve un objeto a lo largo de una trayectoria circular alrededor de un *punto pivote* (xr, yr), dado un ángulo θ:
x' = x·cos(θ) − y·sin(θ)
y' = x·sin(θ) + y·cos(θ)
Para rotar respecto a un punto arbitrario se aplica la secuencia:
1. Trasladar el punto pivote al origen.
2. Rotar respecto al origen.
3. Trasladar de vuelta a la posición original.

> La rotación bidimensional es equivalente a una rotación respecto al eje *Z* en 3D.

---

### 2.1.4. Sesgado (Shear)

El sesgado es una transformación *no rígida* que deforma el objeto original. Existen dos tipos:

- *Sesgo en X (horizontal):* Los valores de y no cambian; los de x se desplazan según y.
- *Sesgo en Y (vertical):* Los valores de x no cambian; los de y se desplazan según x.
Sesgo en X:  x' = x + shx·y,  y' = y
Sesgo en Y:  x' = x,  y' = y + shy·x
> Las líneas oblicuas resultantes tienen longitud mayor a 1.

---

## 2.2. Representación Matricial de las Transformaciones Bidimensionales

Para unificar todas las transformaciones en una sola operación, se utilizan *coordenadas homogéneas, representando cada punto 2D como [x, y, 1] y aplicando matrices de **3×3*.

### Matrices de transformación:

*Traslación:*
| 1  0  tx |
| 0  1  ty |
| 0  0   1 |
*Escalamiento:*
| Sx  0  0 |
|  0 Sy  0 |
|  0  0  1 |
*Rotación:*
| cos(θ)  −sin(θ)  0 |
| sin(θ)   cos(θ)  0 |
|    0        0    1 |
*Sesgado en X:*
| 1  shx  0 |
| 0   1   0 |
| 0   0   1 |
### Composición de transformaciones

La *composición* (concatenación) consiste en multiplicar matrices para aplicar varias transformaciones en una sola operación, ganando eficiencia computacional.
M_compuesta = T · R · S
> La multiplicación de matrices *no es conmutativa*, por lo que el orden de aplicación importa.

Los elementos de la matriz general tienen el siguiente significado:

| Posición | Significado |
|----------|-------------|
| a | Escalado en eje X |
| b | Sesgado en eje Y |
| c | Sesgado en eje X |
| d | Escalado en eje Y |
| tx | Traslación en eje X |
| ty | Traslación en eje Y |

### Ejemplo de ejercicio de control con teclas de direccion en Blender

<img width="910" height="384" alt="imagen mov 1" src="https://github.com/user-attachments/assets/20b27735-4e9f-4343-bd79-b57af3ff9a0d" />

<img width="908" height="386" alt="imagen mov 2" src="https://github.com/user-attachments/assets/d084ca20-27c0-49c1-be16-f734bf60b098" />

Código:

Permite el movimiento por el eje "x" y "z".

```python
import bpy

class SimpleMove(bpy.types.Operator):
    bl_idname = "object.simple_move"
    bl_label = "Mover Cualquier Cosa"

    def modal(self, context, event):
        # Toma el primer objeto de la lista de la escena
        obj = bpy.data.objects[0] 

        if event.value == 'PRESS':
            if event.type == 'LEFT_ARROW':  obj.location.x -= 1
            elif event.type == 'RIGHT_ARROW': obj.location.x += 1
            elif event.type == 'UP_ARROW':    obj.location.z += 1
            elif event.type == 'DOWN_ARROW':  obj.location.z -= 1
            elif event.type == 'ESC': return {'FINISHED'}
            
        return {'RUNNING_MODAL'}

    def invoke(self, context, event):
        context.window_manager.modal_handler_add(self)
        return {'RUNNING_MODAL'}

bpy.utils.register_class(SimpleMove)
bpy.ops.object.simple_move('INVOKE_DEFAULT')
```

---

## 2.3. Trazo de Líneas Curvas

Las curvas paramétricas se describen mediante ecuaciones que dependen de un parámetro auxiliar, permitiendo representar formas complejas con precisión matemática.

---

### 2.3.1. Bézier

Las curvas de Bézier fueron desarrolladas por *Pierre Bézier* en los años 60, originalmente para el diseño de carrocerías de automóviles Renault. Se definen mediante *puntos de control* y polinomios de Bernstein.

<img width="711" height="317" alt="curva" src="https://github.com/user-attachments/assets/ffffe73e-29a5-42f3-9477-fbbfb81526d7" />

*Propiedades principales:*
- La curva *comienza en P₀ y termina en Pₙ* (interpolación de puntos extremos).
- El control es *global*: mover un punto de control modifica toda la curva.
- La curva está contenida dentro de la *envolvente convexa* de los puntos de control.
- Es infinitamente derivable.
- El grado del polinomio es n − 1, donde n es el número de puntos de control.

*Tipos según número de puntos:*

| Puntos de control | Tipo de curva |
|-------------------|---------------|
| 2 | Lineal (recta) |
| 3 | Cuadrática (parabólica) |
| 4 | Cúbica |

*Algoritmo de De Casteljau* (construcción iterativa):
1. Conectar los puntos de control con segmentos.
2. En cada segmento, marcar el punto a distancia proporcional al parámetro t.
3. Conectar esos puntos y repetir recursivamente.
4. El punto final al variar t de 0 a 1 traza la curva.

> Se usa ampliamente en: Adobe Illustrator, Inkscape, Photoshop, CSS animations, PostScript.

<img width="719" height="360" alt="cubico" src="https://github.com/user-attachments/assets/837d8cfa-0f4c-4791-874a-e7daeea76eba" />

---

### 2.3.2. B-Spline

Las B-Splines (Basis Splines) son una *generalización de las curvas de Bézier, desarrolladas formalmente por **Isaac Jacob Schoenberg*. Resuelven la principal desventaja de Bézier: el control global.

*Parámetros principales:*
- P₀, ..., Pₙ: puntos de control
- k: orden de la B-Spline (k = grado + 1)
- d: grado (usualmente d = 3 para cúbicas)
- Ni,k: funciones B-Spline base
  
<img width="717" height="748" alt="spline" src="https://github.com/user-attachments/assets/e9b541b9-144d-43b4-9335-2fd380d9cd96" />

*Ventajas sobre Bézier:*

| Característica | Bézier | B-Spline |
|----------------|--------|----------|
| Control | Global | *Local* |
| Cambio en un punto | Afecta toda la curva | Afecta solo un segmento |
| Grado | Depende del nº de puntos | Independiente del nº de puntos |
| Suavidad | Alta | Alta (Cᵏ⁻²) |

> Las B-Splines pueden evaluarse de forma numéricamente estable mediante el *algoritmo de De Boor*.

---

## 2.4. Fractales

El término *"fractal"* proviene del latín fractus (fragmentado, roto). Fue acuñado por *Benoît Mandelbrot* en 1975 en su libro The Fractal Geometry of Nature.

### Definición

Un fractal es un objeto geométrico cuya estructura básica, fragmentada o aparentemente irregular, *se repite a diferentes escalas* (autosimilitud). Su dimensión de Hausdorff-Besicovitch es estrictamente mayor que su dimensión topológica.

<img width="713" height="538" alt="fractal" src="https://github.com/user-attachments/assets/14d930e5-e711-4b2e-8e4c-6f18c08d4b89" />

### Características principales

- *Autosimilitud:* Al hacer zoom, se repite el mismo patrón.
- *Dimensión fraccionaria:* No es un entero normal.
- *Generación iterativa:* Se construyen aplicando funciones recursivamente.

### Fractales clásicos

| Fractal | Creador | Descripción |
|---------|---------|-------------|
| Conjunto de Mandelbrot | Benoît Mandelbrot | El más popular; definido en el plano complejo |
| Conjunto de Julia | Gaston Julia | Familia de fractales relacionada con Mandelbrot |
| Triángulo de Sierpinski | Waclaw Sierpinski (1915) | Triángulo con subdivisiones infinitas |
| Copo de nieve de Koch | Helge von Koch (1904) | Curva de perímetro infinito |

### Conjunto de Mandelbrot

Se construye en el plano complejo con la siguiente regla de recurrencia:
z₀ = 0
z_{n+1} = z_n² + c
- Si la sucesión zₙ *permanece acotada* → c pertenece al conjunto (se colorea de negro).
- Si *diverge al infinito* → c queda excluido (se colorea según velocidad de divergencia).

### Fractales en la naturaleza

- Líneas costeras
- Ramas de árboles
- Romanesco (brécol)
- Copos de nieve
- Relámpagos
  
<img width="711" height="528" alt="romanesco" src="https://github.com/user-attachments/assets/ff70a8a5-b664-401b-9d75-73bdca39fd6d" />

---

## 2.5. Uso y Creación de Fuentes de Texto

Las fuentes tipográficas digitales son representaciones gráficas de caracteres almacenadas en archivos con distintos formatos.

### Tipos de fuentes

| Tipo | Descripción |
|------|-------------|
| *Raster (mapa de bits)* | Cada glifo es un bitmap; depende del dispositivo y resolución |
| *Vectoriales* | Glifos definidos por segmentos de línea escalables |
| *TrueType / OpenType* | Glifos definidos por comandos de línea y curva + sugerencias (hints) |

### Formatos principales

#### TrueType (.ttf)
- Desarrollado por *Apple* a finales de los 80 como alternativa al PostScript Type 1 de Adobe.
- Licenciado gratuitamente a Microsoft; compatible con Windows y Mac.
- Usa elementos *vectoriales no-PostScript*.
- Permite escalar caracteres manteniendo su forma.

#### OpenType (.otf)
- Desarrollado conjuntamente por *Adobe y Microsoft* (1996).
- Extensión de TrueType que añade soporte para datos PostScript.
- Puede contener hasta *65,536 glifos*.
- Multiplataforma (Windows, Mac, Linux).
- Soporta características tipográficas avanzadas: ligaduras, versalitas, fracciones, glifos alternativos.
- Usa el estándar *Unicode* para codificación de caracteres.

#### WOFF / WOFF2
- Formatos comprimidos diseñados para la *web*.
- No compatibles con software de diseño gráfico de escritorio.

### Comparativa TTF vs OTF

| Característica | TrueType (.ttf) | OpenType (.otf) |
|----------------|-----------------|-----------------|
| Creado por | Apple | Adobe + Microsoft |
| Glifos máx. | ~256 | 65,536 |
| Plataformas | Windows / Mac | Multiplataforma |
| Características avanzadas | Limitadas | Ligaduras, versalitas, etc. |
| Calidad de impresión | Alta | Buena |
| Uso recomendado | Impresión | Diseño digital y web |

### Herramientas para crear fuentes

- *FontForge* – Editor libre y de código abierto.
- *Glyphs* – Popular en macOS para diseño tipográfico profesional.
- *Adobe Illustrator* – Para bocetar glifos vectoriales.
- *BirdFont* – Editor gratuito y multiplataforma.

### Uso en programación (Python / Pillow / Matplotlib)

```python
from PIL import ImageFont, ImageDraw, Image

# Cargar una fuente TrueType
fuente = ImageFont.truetype("arial.ttf", size=36)

# Dibujar texto en una imagen
imagen = Image.new("RGB", (300, 100), color="white")
draw = ImageDraw.Draw(imagen)
draw.text((10, 30), "Hola, Graficación", font=fuente, fill="black")
imagen.show()
import matplotlib.pyplot as plt
import matplotlib.font_manager as fm

# Usar fuente personalizada en matplotlib
prop = fm.FontProperties(fname="mi_fuente.ttf")
plt.title("Gráfica con fuente personalizada", fontproperties=prop)
plt.show()
```
### Referencias bibliográficas:

Foley, J. D., van Dam, A., Feiner, S. K., & Hughes, J. F. (1990).
    Computer graphics: Principles and practice (2nd ed.).
    Addison-Wesley.

Hearn, D., & Baker, M. P. (2004). Computer graphics with OpenGL
    (3rd ed.). Pearson Prentice Hall.

Mandelbrot, B. B. (1982). The fractal geometry of nature.
    W. H. Freeman.

Microsoft. (2023). OpenType specification. Microsoft Learn.
    https://learn.microsoft.com/en-us/typography/opentype/spec/

Adobe. (2023). OpenType user guide. Adobe Inc.
    https://helpx.adobe.com/fonts/using/open-type-syntax.html
# Tutorial: Animación 2D en Blender con Grease Pencil
### Personaje caminando paso a paso

> Usa el modo 2D de Blender (Grease Pencil) para dibujar y animar un personaje caminando desde cero.

---

## 📋 Tabla de Contenidos

1. [Requisitos previos](#requisitos-previos)
2. [Configurar la escena 2D](#1-configurar-la-escena-2d)
3. [Dibujar el personaje con Grease Pencil](#2-dibujar-el-personaje-con-grease-pencil)
4. [Entender la animación por poses](#3-entender-la-animación-por-poses)
5. [Crear las poses del ciclo de caminata](#4-crear-las-poses-del-ciclo-de-caminata)
6. [Insertar keyframes de dibujo](#5-insertar-keyframes-de-dibujo)
7. [Previsualizar el ciclo](#6-previsualizar-el-ciclo)

---

## Requisitos previos

- *Blender 3.0 o superior* instalado → https://www.blender.org/download/
- Un mouse (opcional pero muy recomendado para dibujar)
- Una tableta gráfica (no obligatoria, pero ayuda mucho al dibujar)

---

## 1. Configurar la escena 2D

### 1.1 Crear una escena 2D desde la pantalla de inicio

1. Abre Blender.
2. En la pantalla de inicio (Splash Screen), selecciona *2D Animation*.
   - Esto crea automáticamente una escena optimizada para Grease Pencil.
   - Verás un lienzo blanco en lugar del típico cubo 3D.
     
<img width="765" height="476" alt="menu blender" src="https://github.com/user-attachments/assets/8e3c5a65-dcc0-4e95-a013-3802c578bec0" />

>  Si ya tenías Blender abierto: *File → New → 2D Animation*.

### 1.2 Configurar FPS y duración

1. Panel derecho → ícono de *escena*  → sección *Frame Rate*.
2. Selecciona 24 fps.
3. Define el rango de frames:
   - *Start:* 1
   - *End:* 25
   
>  Un ciclo de caminata básico usa *8 poses. A 3 fps de animación (en 24 fps), el ciclo completo dura **25 frames* = ~0.6 segundos. Repetiremos ese ciclo varias veces.

### 1.3 Configurar la resolución

1. Panel derecho → ícono de *cámara* → *Format*.
2. Resolución: 1920 x 1080 al 100%.

---

## 2. Dibujar el personaje con Grease Pencil

### 2.1 ¿Qué es Grease Pencil?

Grease Pencil es la herramienta de dibujo 2D dentro de Blender. Funciona como una capa de dibujo donde puedes:
- Dibujar con trazos a mano alzada
- Colorear con rellenos
- Animar cada trazo individualmente

### 2.2 Seleccionar el objeto Grease Pencil

1. En la escena 2D, ya hay un objeto Grease Pencil vacío llamado *Stroke*.
2. Haz clic sobre él para seleccionarlo (borde naranja).
3. Presiona Tab para entrar en *Edit Mode* o ve a la barra superior y selecciona *Draw Mode*.

### 2.3 Elegir el pincel de dibujo

1. En la barra superior izquierda del viewport, selecciona el modo *Draw* (lápiz).
2. En la barra de herramientas izquierda verás los pinceles disponibles:
   - *Ink* → trazo limpio y firme (recomendado para personajes)
   - *Pencil* → trazo con textura
   - *Fill* → para rellenar áreas cerradas

### 2.4 Dibujar el personaje (pose de referencia)

Para este tutorial usaremos un personaje de anime (Pochita).

Dibuja las siguientes partes en capas separadas:

![pochita referencia](https://github.com/user-attachments/assets/1d82a34e-0f90-40cf-8725-5525219207ab)


*Cómo crear capas:*
1. Panel derecho → ícono de *líneas verdes* (Grease Pencil Properties).
2. Sección *Layers* → haz clic en + para agregar una capa.
3. Renómbrala haciendo doble clic sobre el nombre.

>  Dibujar en capas separadas te permite mover cada parte del cuerpo de forma independiente, lo cual es clave para la animación.

### 2.5 Dibujar cada parte

1. Selecciona la capa correspondiente en el panel de capas.
2. Asegúrate de estar en *Draw Mode*.
3. Dibuja con clic sostenido sobre el lienzo.
4. Presiona Ctrl + Z para deshacer si te equivocas.

---

## 3. Entender la animación por poses

La caminata se anima dibujando *poses clave* en fotogramas distintos. Blender NO interpola los dibujos automáticamente — cada pose debe dibujarse a mano.

---

## 4. Crear las poses del ciclo de caminata

### 4.1 Activar el modo de animación de Grease Pencil

1. Asegúrate de estar en *Draw Mode*.
2. En la Timeline, activa el botón *Auto Keyframe* (ícono de rombo rojo 🔴 en la Timeline).

### 4.2 Dibujar Pose 1 — Frame 1 

1. Ve al *Frame 1* en la Timeline.
2. Selecciona la capa Pierna_Der.
3. Dibuja la primera imagen de referencia de Pochita.
   
<img width="1847" height="1302" alt="frame 1" src="https://github.com/user-attachments/assets/fc59b106-e536-4069-99d0-e402cf850624" />

*Guardar el keyframe de dibujo:*
1. En la Timeline, presiona I estando sobre el área de Grease Pencil.
2. Selecciona *Active Layer* (guarda solo la capa activa) o *All Layers* (guarda todo).

### 4.3 Dibujar las poses siguientes

Repite el mismo proceso para cada pose:

*Frame 4*
- Dibuja la pose 2.
- Presiona I → *All Layers*.
  
<img width="1855" height="1308" alt="frame 2" src="https://github.com/user-attachments/assets/8b7be5c6-3ff2-49b0-9cc0-393b1e376715" />

*Frame 7*
- Dibuja la pose 3.
- Presiona I → *All Layers*.
  
<img width="1931" height="1274" alt="frame 3" src="https://github.com/user-attachments/assets/64cd89ac-2633-42f4-9c16-4918dd78ec12" />

*Frame 10*
- Dibuja la pose 4.
- Presiona I → *All Layers*.
  
<img width="1892" height="1139" alt="frame 4" src="https://github.com/user-attachments/assets/39755eca-2319-4ac9-8acf-9c56013f3baa" />

*Frame 13*
- Dibuja la pose 5.
- Presiona I → *All Layers*.
  
<img width="1889" height="1242" alt="frame 5" src="https://github.com/user-attachments/assets/d80357bb-a53f-478f-8f63-1861103034d4" />


*Frames 6, 7, 8:*
- Repite el mismo patrón espejado de los frames 4, 7 y 10.

>  Tip: Puedes usar Shift + clic en la Timeline para ver el dibujo del frame anterior como referencia fantasma (onion skinning).

### 4.4 Activar Onion Skinning (piel de cebolla)

El *Onion Skinning* muestra el dibujo del frame anterior y siguiente en transparencia, para que puedas trazar con mayor precisión.

1. Panel derecho → ícono de *Grease Pencil* → sección *Onion Skinning*.
2. Activa el toggle *Onion Skinning*.
3. Ajusta *Before* y *After* a 2 para ver 2 frames de contexto.

---

## 5. Insertar keyframes de dibujo

En Grease Pencil, los keyframes funcionan diferente al 3D. Cada frame con un dibujo es un keyframe independiente.

### Tipos de keyframe en Grease Pencil

| Tipo | Color en Timeline | Descripción |
|---|---|---|
| *Keyframe normal* | 🟡 Amarillo | Frame con un dibujo nuevo |
| *Hold* | 🟠 Naranja | El dibujo anterior se mantiene sin cambios |

*Para insertar un keyframe de dibujo:*
- Método 1: Dibuja algo en un frame → se crea automáticamente si Auto Keyframe está activo.
- Método 2: I → *Active Layer* o *All Layers* en el frame deseado.

*Para que un dibujo se mantenga varios frames (Hold):*
1. Haz clic derecho sobre el keyframe en la Timeline.
2. Selecciona *Make Frame a Hold*.

---

## 6. Previsualizar el ciclo

1. Ve al *Frame 1* con Shift + ←.
2. Presiona *Spacebar* para reproducir.
3. Observa si las poses se ven fluidas.
4. Pausa con *Spacebar* y ajusta los dibujos que no se vean bien.

---

## 7. Agregar movimiento horizontal

Para que el personaje camine atravesando la pantalla (no solo en el mismo lugar), mueve el objeto Grease Pencil completo con keyframes de posición.

### 7.1 Volver al Object Mode

1. Presiona Tab para salir de Draw Mode y volver a *Object Mode*.
2. Selecciona el objeto Grease Pencil completo.

### 7.2 Keyframe de posición inicial

1. Ve al *Frame 1*.
2. Mueve el personaje al lado izquierdo de la pantalla: G → X → -8 → Enter.
3. Presiona I → *Location*.

### 7.3 Keyframe de posición final

1. Ve al *Frame 25*.
2. Mueve el personaje al lado derecho: G → X → 8 → Enter.
3. Presiona I → *Location*.

Ahora el personaje caminará de izquierda a derecha mientras el ciclo de caminata se repite.

---

##  Errores comunes

| Error | Causa | Solución |
|---|---|---|
| El dibujo desaparece en otros frames | Cada frame es independiente | Usa *Hold* si quieres mantener el dibujo varios frames |
| El personaje no se mueve horizontalmente | Keyframes de posición no insertados | Ve a Object Mode y agrega keyframes de Location |
| Las poses se ven muy bruscas | Pocas poses intermedias | Agrega poses de follow-through entre las poses principales |
| El ciclo tiene un salto al repetirse | Frame 1 y frame 17 son distintos | Asegúrate de que la pose 9 (frame 17) sea idéntica a la pose 1 |
| No se ve el dibujo anterior como referencia | Onion Skinning desactivado | Actívalo en Grease Pencil Properties |
| Auto Keyframe no guarda los dibujos | Solo guarda propiedades 3D, no trazos | Presiona I manualmente para guardar cada dibujo |

---

##  Glosario

| Término | Significado |
|---|---|
| *Grease Pencil* | Herramienta de dibujo 2D dentro de Blender |
| *Keyframe de dibujo* | Frame donde guardas un dibujo específico |
| *Onion Skinning* | Ver frames anteriores/siguientes en transparencia |
| *Ciclo de caminata* | Secuencia de poses que se repite para simular caminar |
| *Hold* | Mantener el mismo dibujo durante varios frames |
| *Draw Mode* | Modo de Blender para dibujar con Grease Pencil |
| *Auto Keyframe* | Guarda automáticamente propiedades al moverlas |

---

##  Recursos adicionales

- Documentación Grease Pencil: https://docs.blender.org/manual/en/latest/grease_pencil/
- Comunidad Blender: https://www.reddit.com/r/blender/
- Referencia visual de ciclo de caminata: busca "Richard Williams walk cycle" en YouTube

---

Tutorial · Blender 3.x+ · Grease Pencil 2D


