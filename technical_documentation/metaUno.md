## Meta 1: Configuración de Objetos y Campos

### Objetos Utilizados y sus Configuraciones

#### 1. Contact (Objeto Estándar - Estudiante)
Objeto principal para almacenar la información de los estudiantes.

**Campos Personalizados:**
- **Matrícula**
  - Tipo: Auto Number
  - Formato: EST-{00000}
  - Uso: Identificador único del estudiante
  
- **Estado**
  - Tipo: Picklist
  - Valores: Guanajuato, Nuevo León, Jalisco, Querétaro
  - Uso: Sede del estudiante
  
- **Periodo**
  - Tipo: Picklist
  - Valores: Semestral, Cuatrimestral
  - Uso: Define el tipo de periodo académico

- **Fecha de Nacimiento**
  - Tipo: Date
  
- **Promedio**
  - Tipo: Number(3,1)
  - Rango: 0-10
  - Validación: Valor entre 0 y 10 con un decimal

#### 2. Product2 (Objeto Estándar - Materias)
Objeto para gestionar el catálogo de materias.

**Campos Personalizados:**
- **Código Materia**
  - Tipo: Text(10)
  - Uso: Identificador de la materia
  
- **Periodo**
  - Tipo: Picklist
  - Valores: Semestral, Cuatrimestral
  - Uso: Define el periodo al que pertenece la materia

#### 3. Pricebook2 (Objeto Estándar - Listas de Precios)
Gestión de precios por sede.
- Lista de Precios Guanajuato ($17,000)
- Lista de Precios Nuevo León ($18,000)
- Lista de Precios Jalisco ($13,000)
- Lista de Precios Querétaro ($15,000)

#### 4. Opportunity (Objeto Estándar - Inscripción)
Gestión del proceso de inscripción.

**Campos Personalizados:**
- **Estudiante**
  - Tipo: Lookup(Contact)
  - Uso: Relación con el estudiante
  
- **Cantidad_de_Materias**
  - Tipo: Number
  - Uso: Número de materias a inscribir
  
- **Forma de Pago**
  - Tipo: Picklist
  - Valores: Mensualidades, Contado

#### 5. Quote (Objeto Estándar - Cotización)
Gestión de cotizaciones.

**Campos Personalizados:**
- **Campus**
  - Tipo: Picklist
  - Valores: Guanajuato, Nuevo León, Jalisco, Querétaro
  
- **Número de Materias**
  - Tipo: Number
  
- **Tipo de Beca**
  - Tipo: Picklist
  - Valores: Ninguna, Excelencia, Necesidad Económica, Familiar Docente
  
- **Estado de Cotización**
  - Tipo: Picklist
  - Valores: Borrador, En Revisión, Aprobada, Rechazada, Enviada

### Relaciones entre Objetos

1. **Contact → Opportunity**
   - Tipo: Uno a muchos
   - Campo: Estudiante (lookup)
   - Descripción: Un estudiante puede tener múltiples inscripciones

2. **Product2 → Pricebook2**
   - Tipo: Muchos a muchos
   - Mediante: Price Book Entries
   - Descripción: Una materia puede tener diferentes precios según la sede

### Consideraciones Importantes
1. Los estudiantes están vinculados directamente a sus inscripciones mediante el campo lookup
2. Cada sede tiene su propia lista de precios
3. Las materias mantienen el mismo código pero pueden variar en precio según la sede
