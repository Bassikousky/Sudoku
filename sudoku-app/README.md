# Sudoku Master

Una aplicación interactiva de Sudoku construida con **Vue 3** y **Vite**. El juego permite generar tableros con diferentes niveles de dificultad, validar números en tiempo real, y resolver automáticamente el puzzle.

## 🎮 Características

- ✅ Generación automática de tableros de Sudoku válidos
- ✅ Tres niveles de dificultad (Fácil, Medio, Difícil)
- ✅ Validación en tiempo real de conflictos
- ✅ Soporte para teclado (números 1-9, Backspace/Delete)
- ✅ Interfaz numérica interactiva
- ✅ Botón para resolver automáticamente
- ✅ Detección de victoria

---

## 📋 Estructura del Código

### **1. Datos Reactivos (ref)**

```javascript
const grid = ref(initGrid());
```
Matriz 9x9 que almacena los números actuales del tablero. Se inicializa con ceros.

```javascript
const initialGrid = ref(Array.from({ length: 9 }, () => Array(9).fill(false)));
```
Matriz 9x9 que marca cuáles celdas son fijas (generadas por el juego) y cuáles pueden editarse. Los `true` indican celdas que no pueden modificarse.

```javascript
const difficulty = ref(30);
```
Número de celdas que se vacían al generar el tablero (20=Fácil, 40=Medio, 60=Difícil).

```javascript
const gameStarted = ref(false);
```
Controla si el juego ha comenzado. Muestra/oculta la pantalla de juego.

```javascript
const selectedCell = ref({ r: null, c: null });
```
Almacena la posición (fila, columna) de la celda actualmente seleccionada.

---

### **2. Funciones de Validación**

#### `hasConflict(r, c)`
Verifica si existe un conflicto en la posición (r, c):
- Busca duplicados en la **fila**
- Busca duplicados en la **columna**
- Busca duplicados en el **bloque 3x3**

Retorna `true` si hay conflicto, `false` si es válido.

#### `isSafe(board, row, col, num)`
Función auxiliar para la generación de tableros. Valida que un número pueda colocarse en una posición sin violar las reglas del Sudoku.

---

### **3. Generación del Tablero**

#### `fillGrid(board)`
Algoritmo **backtracking** que rellena el tablero de forma recursiva:
1. Busca la primera celda vacía (0)
2. Intenta números del 1-9 en orden aleatorio
3. Si es seguro colocar el número, continúa recursivamente
4. Si no hay solución, retrocede y prueba otro número

Retorna `true` cuando el tablero está completo y válido.

#### `generateSudoku()`
Genera un nuevo juego:
1. Inicializa un tablero vacío
2. Llena el tablero completamente con `fillGrid()`
3. Vacía celdas según el nivel de dificultad
4. Marca las celdas originales como fijas en `initialGrid`

---

### **4. Resolución del Juego**

#### `solve()`
1. Borra todos los números ingresados por el usuario (mantiene los fijos)
2. Llena el tablero con la solución correcta usando `fillGrid()`

---

### **5. Interacción del Usuario**

#### `selectCell(r, c)`
Selecciona una celda si no es fija. Actualiza `selectedCell.value`.

#### `setNumber(num)`
Coloca un número (1-9) en la celda seleccionada.

#### `eraseCell()`
Borra el contenido de la celda seleccionada (la pone en 0).

#### `handleKeyPress(event)`
Escucha el teclado del usuario:
- Teclas **1-9**: Colocan números
- **Backspace/Delete**: Borran la celda

---

### **6. Condición de Victoria**

#### `gameWon` (computed)
Propiedad computada que verifica si se ha ganado:
- El tablero debe estar **completamente lleno** (sin ceros)
- **No debe haber conflictos** en ninguna celda

Si ambas condiciones se cumplen, muestra el mensaje de victoria con animación.

---

## 🎨 Interfaz de Usuario

### **Estructura en Vue**

```vue
<div class="container">
  <!-- Selector de dificultad (siempre visible) -->
  <div class="settings">
    <select v-model="difficulty">
      <option :value="20">Fácil</option>
      <option :value="40">Medio</option>
      <option :value="60">Difícil</option>
    </select>
  </div>

  <!-- Botón para empezar (solo antes de comenzar) -->
  <button v-if="!gameStarted" @click="generateSudoku">Jugar</button>

  <!-- Pantalla de juego (solo cuando ha comenzado) -->
  <div v-if="gameStarted" class="game-screen">
    <!-- Mensaje de victoria -->
    <div v-if="gameWon" class="win-message">
      🎉 ¡Felicidades! Has resuelto el Sudoku 🎉
    </div>

    <!-- Tablero y controles -->
    <div class="game-container">
      <!-- Tablero 9x9 -->
      <div class="sudoku-board">
        <div v-for="(row, rowIndex) in grid" :key="rowIndex" class="row">
          <div v-for="(cell, colIndex) in row" :key="colIndex"
               :class="[...]"
               @click="selectCell(rowIndex, colIndex)">
            {{ cell !== 0 ? cell : '' }}
          </div>
        </div>
      </div>

      <!-- Teclado numérico -->
      <div class="keypad">
        <button v-for="n in 1-9" @click="setNumber(n)">{{ n }}</button>
        <button @click="eraseCell" class="btn-erase">Borrar</button>
      </div>
    </div>

    <!-- Botones de control -->
    <div class="controls">
      <button @click="generateSudoku">Nuevo Juego</button>
      <button @click="solve" class="btn-solve">Resolver</button>
    </div>
  </div>
</div>
```

---

## 🎨 Estilos CSS Importantes

### **Celdas del Tablero**
- `.cell`: Celda base (45x45px, borde gris)
- `.is-fixed`: Números originales (fondo gris, texto bold)
- `.is-selected`: Celda seleccionada (fondo azul claro, outline)
- `.is-invalid`: Número con conflicto (fondo rojo, texto rojo)

### **Bordes del Tablero**
- Filas cada 3 divisiones: `border-bottom: 3px`
- Columnas cada 3 divisiones: `border-right: 3px`
- Borde exterior: `3px solid #000`

### **Animaciones**
```css
@keyframes bounce {
  from { transform: scale(1); }
  to { transform: scale(1.05); }
}
```
La animación de bounce se aplica al mensaje de victoria.

---

## 🔄 Flujo de la Aplicación

1. **Inicio**: Usuario ve selector de dificultad y botón "Jugar"
2. **Selecciona dificultad** y hace clic en "Jugar"
3. **Se genera un tablero** random usando backtracking
4. **Usuario interactúa**:
   - Hace clic en celdas o usa teclado para ingresar números
   - Los números se validan automáticamente (se resaltan en rojo si hay conflicto)
5. **Victoria**: Cuando el tablero está completo sin errores, muestra mensaje celebración
6. **Nuevo juego**: Puede empezar de nuevo o resolver automáticamente

---

## 📦 Instalación y Ejecución

```bash
# Instalar dependencias
npm install

# Ejecutar servidor de desarrollo
npm run dev

# Construir para producción
npm run build
```

---

## 🛠️ Tecnologías Utilizadas

- **Vue 3**: Framework progresivo de JavaScript
- **Vite**: Herramienta de construcción rápida
- **Script Setup**: Sintaxis moderna de componentes Vue
- **Reactivity API**: `ref`, `computed`, `onMounted`, `onBeforeUnmount`

---

## 📝 Notas Técnicas

- El algoritmo de generación usa **backtracking recursivo** para garantizar tableros válidos
- Se barajan los números (1-9) antes de intentar colocarlos para mayor aleatoriedad
- Los listeners de teclado se limpian correctamente en `onBeforeUnmount` para evitar memory leaks
- La validación de conflictos se ejecuta en tiempo real al seleccionar celdas
