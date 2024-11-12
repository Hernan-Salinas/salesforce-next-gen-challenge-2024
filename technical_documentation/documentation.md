# Documentación Meta 1 - Sistema de Cobranza Universitaria

## 1. DOCUMENTACIÓN TÉCNICA

### Descripción General
Este documento detalla la configuración y estructura de datos implementada para la Meta 1 del Sistema de Cobranza Universitaria en Salesforce, que incluye la gestión de estudiantes, materias y proceso de inscripción.

### Objetos y Campos Implementados

#### 1. Contact (Estudiante)
Objeto estándar utilizado para almacenar la información de los estudiantes.

**Campos Personalizados:**
* Matrícula
  - Tipo: Auto Number
  - Formato: EST-{00000}
  - Uso: Identificador único del estudiante
  - Requerido: Sí

* Estado
  - Tipo: Picklist
  - Valores: 
    * Guanajuato
    * Nuevo León
    * Jalisco
    * Querétaro
  - Requerido: Sí

* Periodo
  - Tipo: Picklist
  - Valores:
    * Semestral
    * Cuatrimestral
  - Requerido: Sí

* Fecha de Nacimiento
  - Tipo: Date
  - Requerido: Sí

* Promedio
  - Tipo: Number(3,1)
  - Rango válido: 0-10
  - Requerido: Sí

#### 2. Product2 (Materias)
Objeto estándar para gestionar el catálogo de materias disponibles.

**Campos Personalizados:**
* Código Materia
  - Tipo: Text(10)
  - Requerido: Sí

* Periodo
  - Tipo: Picklist
  - Valores:
    * Semestral
    * Cuatrimestral
  - Requerido: Sí

#### 3. Pricebook2 (Listas de Precios)
Configuración de precios por sede mediante listas de precios estándar.

**Listas Configuradas:**
1. Lista de Precios Guanajuato
   - Precio por materia: $17,000

2. Lista de Precios Nuevo León
   - Precio por materia: $18,000

3. Lista de Precios Jalisco
   - Precio por materia: $13,000

4. Lista de Precios Querétaro
   - Precio por materia: $15,000

#### 4. Opportunity (Inscripción)
Objeto estándar adaptado para gestionar el proceso de inscripción.

**Campos Personalizados:**
* Estudiante
  - Tipo: Lookup(Contact)
  - Relación: Uno a muchos
  - Requerido: Sí

* Cantidad_de_Materias
  - Tipo: Number
  - Requerido: Sí

* Forma de Pago
  - Tipo: Picklist
  - Valores:
    * Mensualidades
    * Contado
  - Requerido: Sí

#### 5. Quote (Cotización)
Objeto estándar para gestionar cotizaciones de inscripción.

**Campos Personalizados:**
* Campus
  - Tipo: Picklist
  - Valores:
    * Guanajuato
    * Nuevo León
    * Jalisco
    * Querétaro
  - Requerido: Sí

* Número de Materias
  - Tipo: Number
  - Requerido: Sí

* Tipo de Beca
  - Tipo: Picklist
  - Valores:
    * Ninguna
    * Excelencia
    * Necesidad Económica
    * Familiar Docente
  - Default: Ninguna

* Estado de Cotización
  - Tipo: Picklist
  - Valores:
    * Borrador
    * En Revisión
    * Aprobada
    * Rechazada
    * Enviada
  - Default: Borrador
  - Requerido: Sí

### Relaciones entre Objetos

1. Contact → Opportunity
   * Tipo: Uno a muchos
   * Relación: Un estudiante puede tener múltiples inscripciones
   * Campo de relación: Estudiante (lookup en Opportunity)

2. Product2 → Pricebook2
   * Tipo: Muchos a muchos
   * Relación: Una materia puede tener diferentes precios según la sede
   * Implementación: A través de Price Book Entries

### Reglas de Validación

#### 1. Limite_Materias_Por_Periodo (en Opportunity)
```apex
OR(
    AND(
        ISPICKVAL(Estudiante__r.Periodo__c, "Semestral"),
        Cantidad_de_Materias__c > 7
    ),
    AND(
        ISPICKVAL(Estudiante__r.Periodo__c, "Cuatrimestral"),
        Cantidad_de_Materias__c > 4
    )
)
```

**Descripción:** 
Valida que no se excedan los límites de materias según el periodo del estudiante:
- Periodo Semestral: Máximo 7 materias
- Periodo Cuatrimestral: Máximo 4 materias

**Mensaje de Error:** 
"El número de materias excede el límite permitido para el periodo seleccionado. Semestral: máximo 7 materias. Cuatrimestral: máximo 4 materias."

---
