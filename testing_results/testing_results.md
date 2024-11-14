# DOCUMENTACIÓN DE CASOS DE PRUEBA Y RESULTADOS
## Sistema de Cobranza Universidad Liderly

### META 1: CREACIÓN DE OBJETOS Y CAMPOS

#### Caso de Prueba 1: Creación de Estudiante
**Objetivo**: Validar la creación correcta de un estudiante con todos sus campos.

**Pasos de Prueba**:
1. Crear nuevo Contact
2. Llenar campos requeridos
3. Verificar generación de matrícula
4. Guardar registro

**Resultado Esperado**: 
- Matrícula generada automáticamente (EST-00001)
- Campos validados correctamente
- Registro guardado exitosamente

**Caputras en Images:**
Prueba 1.1: Formulario de nuevo estudiante con campos llenos

#### Caso de Prueba 2: Validación de Límites de Materias
**Objetivo**: Verificar regla de validación de materias por periodo.

**Escenarios Probados**:
1. Semestral - 8 materias (debe fallar)
2. Semestral - 7 materias (debe pasar)

**Caputras en Images: Pruebas 2.1 y 2.2**

Prueba 2.1: Mensaje de error al exceder límite de materias
Prueba 2.2: Inscripción exitosa con 7 materias

### META 2: CÁLCULOS DE COSTOS

#### Caso de Prueba 3: Cálculos por Campus
**Objetivo**: Verificar cálculos correctos según campus.

**Capturas en Images: Pruebas 3.1 a 3.4**
Matriz de Pruebas: 
| Campus | Materias | Resultado |
|--------|----------|-----------|
| Guanajuato | 3 | $51,000 |
| Nuevo León | 3 | $54,000 |
| Jalisco | 3 | $39,000 |
| Querétaro | 3 | $45,000 |

Prueba 3.1: Guanajuato 
Prueba 3.2: Nuevo León 
Prueba 3.3: Jalisco 
Prueba 3.4: Querétaro

#### Caso de Prueba 4: Descuentos Progresivos
**Objetivo**: Validar aplicación correcta de descuentos.

**Escenarios**:
- 1 materia: Sin descuento
- 2 materias: 10% en segunda
- 3 materias: 15% en adicionales

**Capturas en Images: Pruebas 4.1 a 4.3**
Prueba 4.1: 1 materia
Prueba 4.2: 2 materias
Prueba 4.3: 4 materias

### META 3: BECAS Y DESCUENTOS

#### Caso de Prueba 5: Aplicación de Becas
**Objetivo**: Verificar cálculo correcto de becas.

**Escenarios Probados**:
1. Beca Excelencia (Promedio ≥ 9.5)
2. Beca Deportiva
3. Beca Familiar Docente
4. Beca Necesidad Económica

**Caputras en Images: Pruebas 5.1 a 5.4**
Prueba 5.1: Excelencia
Prueba 5.2: Deportiva
Prueba 5.3: Familiar Docente
Prueba 5.4: Necesidad Económica

### META 4: AUTOMATIZACIÓN DE EMAILS

#### Caso de Prueba 6: Envío de Cotizaciones
**Objetivo**: Verificar envío automático de emails.

**Pasos**:
1. Aprobar cotización
2. Verificar envío de email
3. Validar contenido del PDF
   
**Captura en Images: Pruebas 6.1 a 6.2**
Prueba 6.1: Cotización Pago de Contado
Prueba 6.2: Cotización Pago Mensualidades

### META 5: PROCESO DE APROBACIÓN

#### Caso de Prueba 7: Flujo de Aprobación
**Objetivo**: Validar proceso completo de aprobación.

**Captura en Images: Pruebas 7.1 a 7.5**
**Escenarios**:
Prueba 7.1: Envío a aprobación
Prueba 7.2: Correo recibido de aprobación
Prueba 7.3: Aprobación o rechazo de la cotización
Prueba 7.4: Comentarios de aprobación
Prueba 7.5: Aprobación completada

