# Feature Specification: Juego del Ahorcado

**Feature Branch**: `001-hangman`

**Created**: 2026-05-31

**Status**: Draft

**Input**: User description: "Crear una especificación funcional para una aplicación web de juego del ahorcado."

## User Scenarios & Testing

### User Story 1 - Iniciar una partida (Priority: P1)

Como jugador
Quiero iniciar una nueva partida de Hangman
Para comenzar a intentar adivinar una palabra oculta

**Why this priority**: Sin una partida activa no existe juego. Esta historia establece el estado inicial necesario para cualquier interacción posterior.

**Independent Test**: Mediante la API. POST /games crea una nueva partida.
GET /games/:id devuelve el estado inicial.

**Acceptance Scenarios**:

1. **Crear una nueva partida**
   **Given** el sistema dispone de al menos una palabra en el diccionario
   **When** el jugador solicita crear una nueva partida
   **Then** el sistema genera una partida con identificador único
   **And** selecciona una palabra aleatoria del diccionario
   **And** devuelve el estado inicial de la partida
   **And** el estado de la partida es "playing"
   **And** la palabra se muestra completamente oculta
   **And** los intentos restantes son 6
   **And** no existen letras acertadas
   **And** no existen letras erróneas

2. **Consultar el estado inicial**
   **Given** existe una partida recién creada
   **When** el jugador consulta la partida
   **Then** el sistema devuelve el mismo estado inicial
   **And** la palabra permanece oculta
   **And** los intentos restantes son 6
   **And** el estado es "playing"

3. **No hay palabras disponibles**
   **Given** el diccionario está vacío
   **When** el jugador solicita crear una partida
   **Then** el sistema rechaza la operación
   **And** informa que no existen palabras disponibles para jugar
---

### User Story 2 - Adivinar letras durante una partida (Priority: P1)

Como jugador  
Quiero ingresar letras durante una partida activa  
Para descubrir progresivamente la palabra oculta

### Why this priority

Esta historia contiene la mecánica principal del juego. Una vez creada la
partida, el jugador debe poder interactuar con ella enviando letras y
recibiendo feedback inmediato sobre su progreso.

### Independent Test

Backend API únicamente.

- POST /games/:id/guess` procesa una letra enviada por el jugador.
- GET /games/:id` permite verificar el estado actualizado de la partida.

Verificable mediante curl, Postman o Supertest sin necesidad de frontend.

### Acceptance Scenarios

#### 1. Adivinar una letra correcta

**Given** una partida activa con la palabra "CASA"  
**And** no existen letras adivinadas previamente
**When** el jugador envía la letra "A"
**Then** el sistema registra la letra como acertada  
**And** revela todas las ocurrencias de la letra en la palabra  
**And** la palabra visible pasa a ser "_ A _ A"  
**And** los intentos restantes no se modifican  
**And** la letra "A" aparece en la lista de letras acertadas  
**And** el estado de la partida permanece como "playing"

---

#### 2. Adivinar una letra incorrecta

**Given** una partida activa con 6 intentos restantes
**When** el jugador envía la letra "X"
**Then** el sistema registra la letra como errónea  
**And** agrega la letra a la lista de letras incorrectas  
**And** reduce los intentos restantes a 5  
**And** el estado de la partida permanece como "playing"

---

#### 3. Adivinar una letra presente varias veces

**Given** una partida activa con la palabra "CASA"
**When** el jugador envía la letra "A"
**Then** el sistema revela todas las posiciones donde aparece la letra  
**And** la palabra visible pasa a ser "_ A _ A"  
**And** la letra se registra una única vez como acertada

---

#### 4. Consultar progreso actualizado

**Given** una partida activa  
**And** el jugador ha realizado jugadas válidas
**When** el jugador consulta el estado de la partida
**Then** el sistema devuelve:
**And** la palabra visible actualizada  
**And** las letras acertadas  
**And** las letras incorrectas  
**And** los intentos restantes  
**And** el estado actual de la partida

---

#### 5. Rechazar una letra ya utilizada

**Given** una partida activa  
**And** la letra "A" ya fue enviada anteriormente
**When** el jugador vuelve a enviar la letra "A"
**Then** el sistema rechaza la solicitud  
**And** informa que la letra ya fue utilizada  
**And** el estado de la partida no se modifica  
**And** los intentos restantes no cambian

---

#### 6. Rechazar entrada vacía

**Given** una partida activa
**When** el jugador envía una cadena vacía
**Then** el sistema rechaza la solicitud  
**And** informa que debe ingresarse una única letra válida  
**And** el estado de la partida no se modifica

---

#### 7. Rechazar múltiples caracteres

**Given** una partida activa
**When** el jugador envía "AB"
**Then** el sistema rechaza la solicitud  
**And** informa que solo se permite una letra por intento  
**And** el estado de la partida no se modifica

---

#### 8. Rechazar caracteres no alfabéticos

**Given** una partida activa
**When** el jugador envía "3"
**Then** el sistema rechaza la solicitud  
**And** informa que la entrada debe ser una letra válida  
**And** el estado de la partida no se modifica

---

#### 9. Rechazar jugadas sobre partidas inexistentes

**Given** no existe una partida con el identificador solicitado
**When** el jugador intenta enviar una letra
**Then** el sistema rechaza la solicitud  
**And** informa que la partida no existe

### Notes / Business Rules

- El juego no distingue entre mayúsculas y minúsculas.
- Una letra correcta revela todas sus ocurrencias en la palabra.
- Una letra utilizada previamente no consume intentos.
- Las validaciones no modifican el estado de la partida.
- La detección de victoria o derrota pertenece a la siguiente historia
  (**US3 - Finalizar partida**).

### User Story 3 - Finalizar una partida (Priority: P1)

Como jugador  
Quiero saber cuándo he ganado o perdido una partida  
Para conocer el resultado final del juego

### Why this priority

Una partida no está completa hasta que el sistema determina si el jugador ha
ganado o perdido. Esta historia proporciona el cierre del ciclo principal del
juego y evita que una partida continúe indefinidamente.

### Independent Test

Backend API únicamente.

- Realizar secuencias de jugadas mediante `POST /games/:id/guess`.
- Verificar que el estado de la partida cambia a `won` o `lost` cuando
  corresponda.
- Verificar que no se aceptan nuevas jugadas una vez finalizada la partida.

Verificable mediante curl, Postman o Supertest.

### Acceptance Scenarios

#### 1. Ganar la partida al descubrir todas las letras

**Given** una partida activa con la palabra "SOL"
**And** las letras "S" y "O" ya fueron adivinadas
**When** el jugador envía la letra "L"
**Then** el sistema marca la partida como "won"
**And** revela completamente la palabra "SOL"
**And** mantiene los intentos restantes actuales
**And** devuelve el resultado final de la partida

---

#### 2. Perder la partida al agotar los intentos

**Given** una partida activa con 1 intento restante
**And** la palabra es "SOL"
**When** el jugador envía una letra incorrecta
**Then** el sistema reduce los intentos restantes a 0
**And** marca la partida como "lost"
**And** revela completamente la palabra "SOL"
**And** devuelve el resultado final de la partida

---

#### 3. Consultar una partida ganada

**Given** una partida en estado "won"
**When** el jugador consulta el estado de la partida
**Then** el sistema devuelve el estado "won"
**And** muestra la palabra completa
**And** mantiene el historial de letras acertadas e incorrectas

---

#### 4. Consultar una partida perdida

**Given** una partida en estado "lost"
**When** el jugador consulta el estado de la partida
**Then** el sistema devuelve el estado "lost"
**And** muestra la palabra completa
**And** mantiene el historial de letras acertadas e incorrectas

---

#### 5. Rechazar nuevas jugadas en una partida ganada

**Given** una partida en estado "won"
**When** el jugador intenta enviar una nueva letra
**Then** el sistema rechaza la solicitud
**And** informa que la partida ya ha finalizado
**And** el estado de la partida no se modifica

---

#### 6. Rechazar nuevas jugadas en una partida perdida

**Given** una partida en estado "lost"
**When** el jugador intenta enviar una nueva letra
**Then** el sistema rechaza la solicitud
**And** informa que la partida ya ha finalizado
**And** el estado de la partida no se modifica

### Notes / Business Rules

- Una partida se gana cuando todas las letras únicas de la palabra han sido
  descubiertas.
- Una partida se pierde cuando los intentos restantes llegan a cero.
- Al finalizar la partida, la palabra completa debe quedar visible.
- Una partida finalizada es de solo lectura.
- No se permiten nuevas jugadas una vez alcanzado el estado `won` o `lost`.

### User Story 4 - Jugar una nueva partida (Priority: P2)

Como jugador  
Quiero iniciar una nueva partida después de terminar una anterior  
Para seguir jugando sin abandonar la aplicación

### Why this priority

El MVP es funcional con las historias anteriores: crear una partida, jugarla y
obtener un resultado. Esta historia mejora la experiencia permitiendo encadenar
múltiples partidas de forma sencilla.

### Independent Test

Backend API únicamente.

- Crear una partida.
- Finalizarla con resultado `won` o `lost`.
- Solicitar una nueva partida.
- Verificar que se genera un nuevo juego con estado inicial.

Verificable mediante curl, Postman o Supertest.

### Acceptance Scenarios

#### 1. Crear una nueva partida después de ganar

**Given** una partida en estado "won"
**When** el jugador solicita jugar una nueva partida
**Then** el sistema crea una nueva partida
**And** asigna un identificador único
**And** selecciona una palabra aleatoria del diccionario
**And** devuelve el estado inicial de juego
**And** el estado de la nueva partida es "playing"
**And** los intentos restantes son 6

---

#### 2. Crear una nueva partida después de perder

**Given** una partida en estado "lost"
**When** el jugador solicita jugar una nueva partida
**Then** el sistema crea una nueva partida
**And** asigna un identificador único
**And** selecciona una palabra aleatoria del diccionario
**And** devuelve el estado inicial de juego
**And** el estado de la nueva partida es "playing"
**And** los intentos restantes son 6

---

#### 3. La nueva partida inicia sin información previa

**Given** una partida anterior finalizada
**When** el jugador inicia una nueva partida
**Then** la nueva partida no contiene letras acertadas
**And** no contiene letras incorrectas
**And** la palabra aparece completamente oculta
**And** no conserva información de la partida anterior

---

#### 4. La partida anterior permanece inalterada

**Given** existe una partida finalizada
**And** el jugador inicia una nueva partida
**When** se consulta la partida original
**Then** mantiene su estado final original
**And** no es modificada por la creación de la nueva partida

### Notes / Business Rules

- Una nueva partida siempre comienza desde cero.
- Cada partida posee su propio identificador.
- La creación de una nueva partida no modifica partidas anteriores.
- La palabra utilizada se selecciona aleatoriamente del diccionario disponible.

## Requirements

### Functional Requirements

- **FR-001**: Sistema DEBE iniciar una nueva partida con palabra aleatoria
- **FR-002**: Sistema DEBE mostrar palabra oculta como guiones bajos
- **FR-003**: Sistema DEBE aceptar una letra por turno (a-z, sin tilde)
- **FR-004**: Sistema DEBE mostrar letras acertadas en la palabra
- **FR-005**: Sistema DEBE mostrar letras erróneas en lista separada
- **FR-006**: Sistema DEBE limitar intentos fallidos a 6
- **FR-007**: Sistema DEBE detectar y mostrar victoria
- **FR-008**: Sistema DEBE detectar y mostrar derrota, revel**And**o la palabra
- **FR-009**: Sistema DEBE permitir reiniciar partida
- **FR-010**: Sistema DEBE rechazar letras repetidas con error 400
- **FR-011**: Sistema DEBE rechazar entrada inválida (vacío, múltiple, no letra)
- **FR-012**: Sistema DEBE rechazar jugadas en partida terminada
- **FR-013**: Sistema DEBE devolver 404 para partidas inexistentes

### Key Entities

- **Game**: Representa una partida. Atributos: id, word, correctGuesses[],
  wrongGuesses[], attemptsRemaining, status (playing | won | lost)
- **Guess**: Acción del jugador. Input: letra individual (a-z). Output: estado
  actualizado del Game
- **WordDictionary**: Lista de palabras posibles para selección aleatoria

## Success Criteria

### Measurable Outcomes

- **SC-001**: API responde < 200ms para crear partida y procesar letra
- **SC-002**: Frontend muestra estado actualizado en < 500ms tras cada acción
- **SC-003**: Jugador completa partida (gana o pierde) sin errores del sistema
- **SC-004**: Cobertura de tests ≥ 80%

## Assumptions

- Palabras solo contienen letras a-z (sin tildes, ñ, mayúsculas)
- Una sola partida activa por vez (no multijugador ni sesiones múltiples)
- Jugador usa navegador moderno (Chrome/Firefox/Edge actual)
- Frontend servido estáticamente por Express (mismo origen)
- Diccionario mínimo incluido en código (archivo words.json en backend)
