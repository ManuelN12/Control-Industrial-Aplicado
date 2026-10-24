# Actividad EC1: Sistema masa-muelle-amortiguador

Enunciado: [`Enunciado_Actividad_EC1.pdf`](Enunciado_Actividad_EC1.pdf)

## Archivos

| Archivo | Descripción |
|---|---|
| `MasaMuelle.lvproj` | Proyecto de LabVIEW con el VI principal y el SubVI |
| `Ejercicio3_masa_resorte.vi` | VI principal: simulador del sistema |
| `SubVI_Integrator.vi` | SubVI de integración por Euler, usado dos veces (aceleración → velocidad → posición) |
| `Entrega_EC1_MasaMuelle.zip` | Paquete entregado: `.lvproj` + VI principal + SubVI |
| `imagenes/` | Esquema, cuadro de la masa e icono usados en el panel frontal |
| `Memoria_Actividad_EC1.docx` | Memoria completa en Word, con las respuestas y simulaciones de las tareas 1 a 6 |

---

## Tarea 1. VI que simula el proceso

### Modelo

Balance de fuerzas (segunda ley de Newton), ecuación (4) del enunciado:

$$m\,\ddot{x}(t) = -D\,\dot{x}(t) - K_F\,x(t) + F(t)$$

Despejando la aceleración:

$$\ddot{x} = \frac{F - D\,\dot{x} - K_F\,x}{m}$$

### Implementación en LabVIEW

- **Bucle**: While Loop, detenido con el botón `stop`.
- **Formula Node**: calcula la aceleración `a` y los parámetros estándar:
  ```
  float64 mm;
  mm = max(m, 0.01);
  a  = (F - D*v - Kf*x) / mm;
  K  = 1/Kf;
  z  = D / (2*sqrt(mm*Kf));
  w0 = sqrt(Kf/mm);
  ```
  `max(m, 0.01)` evita la división entre cero, ya que el rango de la masa incluye 0 kg.
- **Integración**: dos copias de `SubVI_Integrator.vi` en cascada (Euler: `y(k+1) = y(k) + Ts·u(k)`).
  - Integrador 1: aceleración `a` → velocidad `v`.
  - Integrador 2: velocidad `v` → posición `x`.
  - `Init` conectado a **First Call?** para reiniciar en la primera iteración; `y_init = 0`, límites ±1000.
- **Realimentación**: `v` y `x` vuelven al Formula Node a través de un **Feedback Node**.
- **Temporización**: `Wait (ms)` = `Ts·1000 / Speed_factor`.
- **Gráficos**: Waveform Charts de posición y fuerza; el multiplicador del eje X (Property Node `XScale.Multiplier`) vale `Ts`, para mostrar el tiempo en segundos.

### Panel frontal

| Elemento | Rango | Valor por defecto |
|---|---|---|
| m [kg] | 0 – 40 | 20 |
| Kf [N/m] | 1 – 10 | 2 |
| D [N/(m/s)] | 0 – 5 | 4 |
| F [N] | –8 – 8 | 0 |
| x [m] | –4 – 4 | — |
| Ts [s] | — | 0,01 |
| Speed_factor | — | 5 |

---

## Tarea 2. Modelización matemática y función de transferencia

### 2.1 Ecuación diferencial

$$m\,\ddot{x}(t) + D\,\dot{x}(t) + K_F\,x(t) = F(t)$$

### 2.2 Transformada de Laplace

Con condiciones iniciales nulas ($x(0)=0$, $\dot{x}(0)=0$):

$$\mathcal{L}\{\dot{x}\} = sX(s), \qquad \mathcal{L}\{\ddot{x}\} = s^2X(s)$$

$$m s^2 X(s) + D s X(s) + K_F X(s) = F(s)$$

$$X(s)\,(m s^2 + D s + K_F) = F(s)$$

$$\boxed{G(s) = \frac{X(s)}{F(s)} = \frac{1}{m s^2 + D s + K_F}}$$

### 2.3 Forma estándar

Dividiendo numerador y denominador entre $K_F$:

$$G(s) = \frac{1/K_F}{\dfrac{m}{K_F}s^2 + \dfrac{D}{K_F}s + 1}$$

Comparando con la forma estándar de segundo orden (ecuación 5):

$$G(s) = \frac{K}{\left(\dfrac{s}{\omega_0}\right)^2 + 2z\dfrac{s}{\omega_0} + 1}$$

| Término | $G(s)$ | Forma estándar |
|---|---|---|
| Numerador | $1/K_F$ | $K$ |
| Coef. de $s^2$ | $m/K_F$ | $1/\omega_0^2$ |
| Coef. de $s$ | $D/K_F$ | $2z/\omega_0$ |

### 2.4 Parámetros

$$K = \frac{1}{K_F} \quad \text{(6)}$$

$$\frac{1}{\omega_0^2} = \frac{m}{K_F} \;\Rightarrow\; \omega_0 = \sqrt{\frac{K_F}{m}} \quad \text{(8)}$$

$$\frac{2z}{\omega_0} = \frac{D}{K_F} \;\Rightarrow\; z = \frac{D\,\omega_0}{2K_F} = \frac{D}{2\sqrt{m K_F}} \quad \text{(7)}$$

### 2.5 Valores por defecto

Con $m = 20$ kg, $K_F = 2$ N/m, $D = 4$ N/(m·s⁻¹):

| Parámetro | Valor |
|---|---|
| $K$ | 0,5 m/N |
| $\omega_0$ | 0,316 rad/s |
| $z$ | 0,316 |

$$G(s) = \frac{1}{20s^2 + 4s + 2} = \frac{0{,}5}{10s^2 + 2s + 1}$$

Polos: $s = -0{,}1 \pm 0{,}3j$. Son complejos con parte real negativa: sistema **estable y subamortiguado** ($z < 1$).

### 2.6 Respuesta a un escalón F = 4 N (valores teóricos)

| Magnitud | Fórmula | Valor |
|---|---|---|
| Valor final | $x(\infty) = K\,F$ | 2 m |
| Sobreoscilación | $M_p = e^{-\pi z/\sqrt{1-z^2}}$ | 35 % |
| Posición máxima | $x(\infty)(1+M_p)$ | ≈ 2,70 m |
| Tiempo de pico | $t_p = \pi / (\omega_0\sqrt{1-z^2})$ | ≈ 10,5 s |
| Tiempo de establecimiento (2 %) | $t_s \approx 4/(z\,\omega_0)$ | ≈ 40 s |

---

## Tarea 3. Influencia de la masa m

Ver `Memoria_Actividad_EC1.docx`.

## Tarea 4. Influencia de la constante del muelle K_F

Ver `Memoria_Actividad_EC1.docx`.

## Tarea 5. Influencia de la constante de amortiguamiento D

Ver `Memoria_Actividad_EC1.docx`.

## Tarea 6. Influencia de la fuerza F

Ver `Memoria_Actividad_EC1.docx`.
