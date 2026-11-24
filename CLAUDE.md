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
6. **Foto + IA**: descartado por ahora (la API de Claude es de pago). Se
   quitó del código. Para piezas irregulares hay una sección manual de
   "Medidas adicionales" en cada módulo, que se anota en el DXF.
7. **Módulos**, todos con sus cálculos (✅ = implementado). Deben cubrir todas
   las variantes habituales de cada pieza, no sólo la más simple:
   - ✅ **Pernos y roscas**: perno, tuerca o conjunto; cabeza hexagonal (ISO 4017),
     Allen (ISO 4762), avellanada (ISO 10642) o varilla roscada; tuerca normal
     (ISO 4032), contratuerca (ISO 4035) o autoblocante (ISO 7040); rosca métrica
     gruesa, fina (ISO 261) o UNC. Área resistente, tracción, corte, von Mises,
     precarga, par, presión bajo cabeza, rosca acoplada mínima.
   - ✅ **Estructuras** para una persona de 90 kg + herramientas: escalera +
     plataforma, solo escalera, plataforma rectangular, escalera de caracol y
     plataforma circular. Perfil de cualquier forma: tubo rectangular, tubo
     redondo, barra redonda, pletina, angular L, canal UPN, viga IPE/HEB, T, con
     catálogo comercial o medidas a mano (propiedades por composición de
     rectángulos; Imin principal para L). Peso propio, carga repartida extra,
     flexión y flecha (vigas y voladizos), pandeo de Euler con K según apoyos,
     columna del caracol (compresión + flexión), baranda (OSHA 890 N),
     ergonomía (Blondel, inclinación, altura libre del caracol).
   - ✅ **Piñón**: dentado recto o helicoidal (β), plano o con cubo, agujero liso
     o con uno/dos chaveteros DIN 6885. Diámetros, fuerzas Wt/Wr/Wa, Lewis con
     dientes virtuales + Barth, chavetas (DIN 6892 para dos), llanta AGMA,
     socavado, ancho de cara, recubrimiento axial.
   - ✅ **Rodamientos**: rígido de bolas, contacto angular, autoalineable de
     bolas, rodillos cilíndricos, cónicos, a rótula, agujas, axial de bolas y
     axial de rodillos, con series de catálogo (valores aprox. SKF, editables).
     P = X·Fr + Y·Fa según tipo, exponente 3 o 10/3, fiabilidad a1 (ISO 281),
     factor de servicio, C requerida, s0, tipos que no admiten carga axial/radial.
   - Ideas siguientes: eje (flexión + torsión, ASME), soldaduras, poleas,
     engranajes cónicos y tornillo sin fin.

## Arquitectura

- `index.html`: interfaz (menú lateral de tarjetas, panel con secciones
  plegables, selectores con iconos/fichas, deslizadores, visor 3D, cajetín)
  y todo el código.
- Cada módulo es un objeto registrado en `MODULOS` con:
  - `selectores`: opciones `[valor, texto, icono]` (o función `(sel) => opciones`
    si dependen de otro selector), `estilo: 'segmentos' | 'fichas'`, y
    `visible: (sel) => bool` opcional.
  - `campos`: definición de las medidas (id, etiqueta, unidad, min, max, valor,
    `rmax` del deslizador, `solo` = tipos de pieza donde aplica o
    `visible: (sel) => bool`; `etiqueta` puede ser función de `sel`).
  - `materiales`: lista con `Re`, `Rm` (MPa), `E` (MPa) y uso recomendado.
    `usaResistencia: false` si el cálculo no usa Re/Rm (rodamientos).
  - `validar(medidas, sel)`, `alCambiarSelector(id, valor, ponerValores, sel)`,
    `alEditarCampo(id, medidas, ponerValores)`.
  - `construir3D(medidas)`: devuelve un `THREE.Group` en milímetros.
  - `calcular(medidas, material)`: devuelve filas de resultados y veredicto.
  - `dibujarDXF(medidas, dxf)`: dibuja las vistas 2D.
- Unidades internas: **mm, N, MPa**. Eje Y del visor = eje de la pieza.

## Convenciones

- Código, comentarios y textos de la interfaz en español.
- Sin dependencias de compilación (nada de npm/bundlers).
- Las fórmulas deben citar su origen en un comentario (norma o libro).
