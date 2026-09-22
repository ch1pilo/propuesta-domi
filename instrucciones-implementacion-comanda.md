# Instrucciones de ajuste — Teléfono y dirección en la creación de comandas (Domi)

Este documento describe **qué** debe cambiar en el formulario de creación de comanda que usan los aliados, no **cómo** implementarlo ni cómo debe verse. Diseño visual, distribución del formulario, componentes y arquitectura técnica quedan a criterio de quien lo construya — esto es solo la lista de reglas de negocio a aplicar.

Contexto: hoy el teléfono y la dirección del cliente final se ingresan sin ninguna validación, lo que causa retrasos en la entrega (dígitos de teléfono incompletos, errores al copiar la ubicación del cliente). Este documento resuelve esos dos puntos.

**Antes de tocar código:** localiza el formulario/componente actual de creación de comanda y los campos existentes de teléfono y dirección. Si algo de lo siguiente no aplica a como está construido el sistema hoy (por ejemplo, si no hay integración con WhatsApp Business API), dilo explícitamente en vez de asumir o inventar la integración.

---

## 1. Teléfono principal — validación de formato (obligatoria, bloqueante)

- Separar en dos partes: **código de operadora** (0412, 0414, 0416, 0422, 0424, 0426) y **número de 7 dígitos**.
- El número debe tener **exactamente 7 dígitos**, solo numéricos — ni más ni menos. No se puede crear la comanda si no cumple esto.
- Esta es la validación más importante: resuelve la causa más común de retrasos (dígito faltante).

## 2. Verificación de WhatsApp (no bloqueante — fase 2 si no hay integración lista)

- Verificar el número contra la API de WhatsApp Business para saber si tiene WhatsApp activo.
- Si **no** tiene, abrir automáticamente un segundo campo de **teléfono alterno** (mismo formato: operadora + 7 dígitos).
- Esto **no debe bloquear** la creación de la comanda si la verificación falla, tarda, o no está disponible — es una advertencia, no un requisito.
- Si el sistema no tiene acceso a esta API todavía, dejar esta parte pendiente y avisar que se necesita gestionar el acceso antes de implementarla.

## 3. Números extranjeros

- Agregar una opción para marcar "cliente en el extranjero".
- Si está activada, pedir: país, código de país + operadora + número extranjero.
- **Obligatorio y bloqueante:** un teléfono nacional de respaldo (mismo formato de 7 dígitos que el punto 1).
- Motivo (para referencia, no para mostrar en pantalla): un número extranjero puede no tener datos/internet local, así que el domi necesita un número nacional al que llamar si WhatsApp no responde.

## 4. Dirección — corrección de la causa real (obligatoria, bloqueante)

**Importante:** el cliente final normalmente ya envía su ubicación correcta al aliado por WhatsApp (comparte "ubicación actual", que genera un link de mapa). El error no está en el cliente — está en que el aliado a veces copia ese link mal, lo cambia sin querer, o mezcla el de otra conversación al transcribirlo a la comanda.

Por eso:
- El campo principal de ubicación debe ser para **pegar el link tal cual** (no reescribir la dirección a mano). Validar que lo pegado tenga formato de URL válida antes de permitir guardar.
- Agregar un botón/enlace "Ver en mapa" junto al campo, que abra ese link pegado, para que el aliado pueda **confirmar visualmente en el momento** que pegó el link correcto, antes de guardar.
- Mantener aparte un campo de texto libre **opcional**, no bloqueante, solo para notas de referencia (torre, piso, apartamento, punto de referencia) — no reemplaza el link.

## 5. Confirmación de dirección antes del despacho (opcional / fase 2)

- Al crear la comanda (botón "Crear comanda" o equivalente), se dispara automáticamente un mensaje al cliente por el chatbot de WhatsApp que ya existe, reenviándole **el mismo link de ubicación** que quedó guardado en el campo del punto 4 — no una dirección aparte — para que confirme que es correcto.
- Mientras se espera respuesta, el botón de crear queda **bloqueado temporalmente** (2 a 3 minutos).
  - Si el cliente confirma dentro de ese lapso → la comanda se crea de inmediato, sin advertencia.
  - Si pasan los 2–3 minutos sin respuesta → el botón se desbloquea solo y la comanda se crea igual, mostrando la advertencia: **"El domi salió sin confirmar la ubicación. Por favor, estar atento."**
- Es un bloqueo con límite de tiempo, no una advertencia sin más ni un bloqueo indefinido: nunca se queda esperando para siempre.

---

## Qué NO cambiar

- Colores, tipografía, distribución visual del formulario — se respeta el sistema de diseño ya existente.
- Cualquier campo de la comanda no mencionado aquí.
- Cómo se implementa cada validación en el código — eso queda a criterio de quien lo construya.

## Por qué importa

Los puntos 1 y 4 son los que más impacto tienen y no dependen de ninguna integración externa — se pueden implementar ya. Atacan directamente las dos causas más comunes de retraso hoy: dígito de teléfono incompleto y error al copiar la ubicación del cliente. El resto (verificación de WhatsApp, confirmación de dirección) suma una capa extra de seguridad, pero no es indispensable para ver una mejora.

## Referencias

- Prototipo interactivo del formulario: https://claude.ai/artifact/1P5vVytbacdgYWtVaMwyei
- Presentación del problema y la propuesta: https://claude.ai/artifact/6WRpq762rafDb4yNGvzrbu
