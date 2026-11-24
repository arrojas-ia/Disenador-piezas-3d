# Diseñador de Piezas 3D — Requisitos del proyecto

Herramienta web (HTML + JavaScript, se ejecuta en el navegador, sin servidor propio)
para diseñar piezas mecánicas ingresando medidas a mano, verlas en 3D, calcularlas
y exportarlas a programas CAD.

## Contexto de uso

- **Se usará en una demostración en vivo en una defensa.** Prioridades:
  - Robustez antes que funciones: ningún dato mal escrito debe romper la página
    (validar entradas, mostrar el error junto al campo y seguir funcionando).
  - Valores por defecto razonables en todos los módulos, para que la pieza se vea
    apenas se abre el módulo.
  - Debe abrir con doble clic en `index.html` (protocolo `file://`). Por eso el
    código propio va dentro de `index.html` o en scripts clásicos; no usar
    `import` de archivos locales (los navegadores lo bloquean en `file://`).
  - Three.js se carga desde CDN con versión fija. Antes de la defensa, comprobar
    que hay internet o dejar una copia local de Three.js como respaldo.
  - Interfaz en español, legible en proyector (letra grande, buen contraste).

## Requisitos funcionales

1. **Ingreso manual de medidas**: cada pieza admite varias medidas independientes
   (piezas irregulares). Las medidas normalizadas (p. ej. tablas ISO) sólo
   rellenan valores por defecto; el usuario siempre puede sobrescribirlas.
2. **Vista 3D** de la pieza con Three.js (rotar, acercar, encuadrar), que se
   regenera al cambiar cualquier medida.
3. **Exportación**:
   - AutoCAD: **DXF** (R12/AC1009, unidades mm, capas, plano 2D con vistas,
     cotas y cajetín).
   - SolidWorks: **STL** (malla 3D en mm). **STEP** queda como mejora futura.
4. **Material**: elegir de una lista filtrada según el trabajo que soporta la
   pieza (carga ligera / media / alta / ambiente corrosivo…), o escribirlo a mano
   junto con sus propiedades (límite elástico Re, resistencia Rm).
5. **Cajetín** con numeración correlativa automática (PZ-0001, PZ-0002…). El
   contador se guarda en el navegador (`localStorage`, siempre dentro de
   try/catch) y aparece en pantalla, en el DXF y en el nombre de los archivos.
6. **Foto + IA** ✅: sacar (cámara) o subir una foto; la IA (Claude, modelo
   `claude-opus-5-5`, SDK oficial `@anthropic-ai/sdk` cargado desde CDN sólo al
   pulsar "Analizar") identifica la pieza y propone qué medidas tomar, con qué
   instrumento y, si hay referencia de escala, un valor estimado. Respuesta en
   JSON con salida estructurada (`output_config.format`). El usuario edita,
   desmarca o agrega medidas y las aplica al módulo; las que no corresponden a un
   campo quedan como "medidas adicionales" y se anotan en el DXF.
   - La clave de API la escribe el usuario en la interfaz (opcionalmente se
     guarda en `localStorage`). Nunca escribir una clave en el código.
   - Botón "Lista estándar sin IA" como respaldo sin internet para la defensa.
7. **Módulos**, todos con sus cálculos:
   - **Pernos y roscas** internas/externas (perno, tuerca, conjunto):
     área resistente, tracción, corte, von Mises, precarga, par de apriete,
     longitud mínima de rosca acoplada. ✅ Implementado.
   - **Escalera / plataforma**: diseñada para una persona de 90 kg más
     herramientas (pendiente).
   - **Piñón de engranaje con chavetero** (pendiente).

## Arquitectura

- `index.html`: interfaz (menú lateral de tarjetas, panel con secciones
  plegables, selectores con iconos/fichas, deslizadores, visor 3D, cajetín,
  vista Foto + IA) y todo el código.
- Cada módulo es un objeto registrado en `MODULOS` con:
  - `selectores`: opciones `[valor, texto, icono]`, `estilo: 'segmentos' | 'fichas'`.
  - `campos`: definición de las medidas (id, etiqueta, unidad, min, max, valor,
    `rmax` del deslizador, `solo` = tipos de pieza donde aplica, `instrumento` y
    `comoMedir` para la lista sin IA).
  - `materiales`: lista de materiales con su uso recomendado.
  - `construir3D(medidas)`: devuelve un `THREE.Group` en milímetros.
  - `calcular(medidas, material)`: devuelve filas de resultados y veredicto.
  - `dibujarDXF(medidas, dxf)`: dibuja las vistas 2D.
- Unidades internas: **mm, N, MPa**. Eje Y del visor = eje de la pieza.

## Convenciones

- Código, comentarios y textos de la interfaz en español.
- Sin dependencias de compilación (nada de npm/bundlers).
- Las fórmulas deben citar su origen en un comentario (norma o libro).
