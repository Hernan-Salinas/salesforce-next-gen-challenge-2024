

## 1. DOCUMENTACIÓN TÉCNICA Meta 1 - CREACIÓN DE OBJETOS O CAMPOS PERSONALIZADOS

### Descripción General
Esta sección detalla la configuración y estructura de datos implementada para la Meta 1 del Sistema de Cobranza Universitaria en Salesforce, que incluye la gestión de estudiantes, materias y proceso de inscripción.

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

#### 2. Product (Materias)
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

#### 3. Pricebook (Listas de Precios)
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

2. Product → Pricebook
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

## DOCUMENTACIÓN TÉCNICA Meta 2 - IMPLEMENTACIÓN DE CÁLCULOS DE COSTOS

### Descripción General
Esta sección detalla la implementación de los cálculos de costos y descuentos para el Sistema de Cobranza Universitaria en Salesforce, incluyendo la lógica de precios por sede y los diferentes tipos de descuentos aplicables.

### Campos Implementados en Quote

#### 1. SubtotalSinDescuentos
**Tipo:** Campo de Fórmula (Currency)
**Descripción:** Calcula el costo total de las materias según la sede seleccionada.
**Fórmula:**
```
NumeroMaterias__c * CASE(Campus__c,
    'Guanajuato', 17000,
    'Nuevo León', 18000,
    'Jalisco', 13000,
    'Querétaro', 15000,
    0)
```

#### 2. Descuento_por_Cantidad_de_Materias
**Tipo:** Campo de Fórmula (Currency)
**Descripción:** Calcula el descuento aplicable según la cantidad de materias.
**Reglas de Descuento:**
- 1 materia: Sin descuento
- 2 materias: 10% de descuento en la segunda materia
- 3 o más materias: 15% de descuento en materias adicionales
**Fórmula:**
```
CASE(NumeroMaterias__c,
    1, 0,
    2, (CASE(Campus__c,
          'Guanajuato', 17000,
          'Nuevo León', 18000,
          'Jalisco', 13000,
          'Querétaro', 15000,
          0) * 0.10),
    (CASE(Campus__c,
          'Guanajuato', 17000,
          'Nuevo León', 18000,
          'Jalisco', 13000,
          'Querétaro', 15000,
          0) * 0.15 * (NumeroMaterias__c - 1)))
```

#### 3. Descuento_por_Pago_de_Contado
**Tipo:** Campo de Fórmula (Currency)
**Descripción:** Calcula el 5% de descuento si el pago es de contado.
**Fórmula:**
```
IF(ISPICKVAL(Opportunity.FormaPago__c, 'Contado'),
    SubtotalSinDescuentos__c * 0.05,
    0)
```

#### 4. Total_Final
**Tipo:** Campo de Fórmula (Currency)
**Descripción:** Calcula el monto final considerando todos los descuentos.
**Fórmula:**
```
SubtotalSinDescuentos__c - Descuento_por_Cantidad_de_Materias__c - Descuento_por_Pago_de_Contado__c
```

### Estructura de Precios por Sede
| Sede | Costo por Materia |
|------|------------------|
| Guanajuato | $17,000.00 |
| Nuevo León | $18,000.00 |
| Jalisco | $13,000.00 |
| Querétaro | $15,000.00 |

---
### Campos de Fórmula en Quote

#### 1. SubtotalSinDescuentos
```
NumeroMaterias__c * CASE(Campus__c,
    'Guanajuato', 17000,
    'Nuevo León', 18000,
    'Jalisco', 13000,
    'Querétaro', 15000,
    0)
```

#### 2. Descuento_por_Cantidad_de_Materias
```
CASE(NumeroMaterias__c,
    1, 0,
    2, (CASE(Campus__c,
          'Guanajuato', 17000,
          'Nuevo León', 18000,
          'Jalisco', 13000,
          'Querétaro', 15000,
          0) * 0.10),
    (CASE(Campus__c,
          'Guanajuato', 17000,
          'Nuevo León', 18000,
          'Jalisco', 13000,
          'Querétaro', 15000,
          0) * 0.15 * (NumeroMaterias__c - 1)))
```

#### 3. Descuento_por_Pago_de_Contado
```
IF(ISPICKVAL(Opportunity.FormaPago__c, 'Contado'),
    SubtotalSinDescuentos__c * 0.05,
    0)
```

#### 4. Total_Final
```
SubtotalSinDescuentos__c - Descuento_por_Cantidad_de_Materias__c - Descuento_por_Pago_de_Contado__c
```

## DOCUMENTACIÓN TÉCNICA Meta 3 - CONFIGURACIÓN DE DESCUENTOS Y BECAS

### Descripción General
Este sección detalla la implementación del sistema de becas y descuentos para el Sistema de Cobranza Universitaria en Salesforce, incluyendo los diferentes tipos de becas disponibles y la validación del límite máximo de descuentos.

### Campos Modificados/Implementados

#### 1. TipoBeca (Campo existente modificado en Quote)
**Tipo:** Picklist
**Valores actualizados:**
- Ninguna
- Excelencia
- Deportiva
- Familiar Docente
- Necesidad Económica

#### 2. Descuento_por_Beca
**Tipo:** Campo de Fórmula (Currency)
**Descripción:** Calcula el descuento según el tipo de beca seleccionada
**Porcentajes de Descuento:**
- Beca Excelencia: 10% (si promedio ≥ 9.5)
- Beca Deportiva: 10%
- Beca Familiar Docente: 30%
- Beca Necesidad Económica: 30%

#### 3. Total_Final (Actualizado)
**Tipo:** Campo de Fórmula (Currency)
**Descripción:** Calcula el monto final considerando todos los descuentos incluyendo becas

### Reglas de Validación

#### Limite_Total_Descuentos
**Objetivo:** Asegurar que la suma total de descuentos no exceda el 60%
**Descripción:** Valida la suma de todos los descuentos:
- Descuento por cantidad de materias
- Descuento por pago de contado
- Descuento por beca

### Estructura de Descuentos

#### 1. Descuentos por Beca
| Tipo de Beca | Porcentaje | Condición |
|--------------|------------|-----------|
| Excelencia | 10% | Promedio ≥ 9.5 |
| Deportiva | 10% | N/A |
| Familiar Docente | 30% | N/A |
| Necesidad Económica | 30% | N/A |

#### 2. Descuentos Adicionales (Ya implementados)
| Tipo | Porcentaje |
|------|------------|
| Pago de Contado | 5% |
| Segunda Materia | 10% |
| Materias Adicionales (3+) | 15% |

### Campos de Fórmula

#### 1. Descuento_por_Beca
```
CASE(TipoBeca__c,
    'Excelencia', 
    IF(Opportunity.Estudiante__r.Promedio__c >= 9.5, SubtotalSinDescuentos__c * 0.10, 0),
    'Deportiva', 
    SubtotalSinDescuentos__c * 0.10,
    'Familiar Docente', 
    SubtotalSinDescuentos__c * 0.30,
    'Necesidad Económica', 
    SubtotalSinDescuentos__c * 0.30,
    0)
```

#### 2. Total_Final (Actualizado)
```
SubtotalSinDescuentos__c - Descuento_por_Cantidad_de_Materias__c - 
Descuento_por_Pago_de_Contado__c - Descuento_por_Beca__c
```

### Regla de Validación

#### Limite_Total_Descuentos
```
(Descuento_por_Cantidad_de_Materias__c + Descuento_por_Pago_de_Contado__c + 
Descuento_por_Beca__c) / SubtotalSinDescuentos__c > 0.60
```

# Documentación Meta 4 - AUTOMATIZACIÓN DEL PROCESO DE COTIZACIÓN

### Descripción General
Esta sección detalla la implementación del proceso automatizado de envío de cotizaciones por email en el Sistema de Cobranza Universitaria en Salesforce. La solución evolucionó desde un intento inicial usando Flow hasta una implementación final usando Apex Trigger.

### Evolución de la Solución

#### Fase 1: Implementación con Flow
**Nombre:** Envio_Cotizacion_PDF
**Descripción:** Primera aproximación usando Process Builder y Flow
**Componentes:**
1. Start Element:
   - Trigger: Record Update
   - Object: Quote
   - Condition: EstadoCotizacion__c = 'Aprobada'

2. Get Records:
   - Object: Contact
   - Filter: Id = Opportunity.Estudiante__c
   - Fields: Id, Name, Email

3. Send Email Action:
   - Template ID: Plantilla_Cotizacion
   - Recipient: Get_Student_Information.Id

4. Email Alert Action:
   - Object: Quote
   - Template: Plantilla_Cotizacion
   - Recipients: Quote Contact

**Limitaciones encontradas:**
- Dificultades con permisos y acceso
- Complejidad en el manejo de relaciones entre objetos
- Problemas de deliverability de emails

#### Fase 2: Implementación Final con Apex

### Componentes Implementados

#### 1. Visualforce Email Template
**Nombre:** Plantilla_Cotizacion
**Descripción:** Template que define el formato y contenido del email de cotización
**Configuración:**
- Available For Use: ✓
- Template Type: Visualforce Email Template
- Recipient Type: Contact
- Related To Type: Quote

#### 2. Apex Class
**Nombre:** QuoteEmailHandler
**Descripción:** Clase que maneja la lógica de envío de emails
**Funcionalidades:**
- Detecta cambios en el estado de la cotización
- Obtiene información del estudiante
- Configura y envía el email usando el template

#### 3. Apex Trigger
**Nombre:** QuoteEmailTrigger
**Descripción:** Trigger que inicia el proceso de envío
**Configuración:**
- Objeto: Quote
- Evento: After Update
- Operación: Envía email cuando EstadoCotizacion__c cambia a 'Aprobada'

### Estructura del Email

#### 1. Información General
- Número de cotización
- Fecha
- Campus
- Información del estudiante

#### 2. Detalles Financieros
- Subtotal
- Desglose de descuentos aplicados
- Total final

#### 3. Plan de Pagos
**Pago de Contado:**
- Fecha única: 10 de julio
- Monto total

**Mensualidades:**
- Inscripción: 10 de julio
- 5 mensualidades: agosto a diciembre
- Monto dividido en 6 pagos iguales

---

### Flow (Intento Inicial)
```yaml
Start:
  Type: Record-Triggered Flow
  Object: Quote
  Trigger: When record is updated
  Condition: EstadoCotizacion__c = 'Aprobada'

Get Records:
  Object: Contact
  Filter: Id = Opportunity.Estudiante__c
  Store: Get_Student_Information

Send Email:
  Template: Plantilla_Cotizacion
  RecipientId: Get_Student_Information.Id
  RelatedTo: Quote.Id
```

### Apex Class: QuoteEmailHandler (Solución Final)
```apex
public class QuoteEmailHandler {
    public static void sendQuoteEmail(List<Quote> newQuotes, Map<Id, Quote> oldQuotes) {
        Id templateId = '00Xak000008qMnp';
        
        List<Quote> quotesWithRelations = [
            SELECT Id, Opportunity.Estudiante__c 
            FROM Quote 
            WHERE Id IN :newQuotes
        ];
        
        for(Quote q : quotesWithRelations) {
            Quote newQuote = (Quote)newQuotes.get(0);
            System.debug('EstadoCotizacion__c: ' + newQuote.EstadoCotizacion__c);
            
            if(oldQuotes == null || 
               (newQuote.EstadoCotizacion__c == 'Aprobada' && 
                oldQuotes.get(newQuote.Id).EstadoCotizacion__c != 'Aprobada')) {
                
                if(q.Opportunity.Estudiante__c != null) {
                    Messaging.SingleEmailMessage mail = new Messaging.SingleEmailMessage();
                    mail.setTargetObjectId(q.Opportunity.Estudiante__c);
                    mail.setTemplateId(templateId);
                    mail.setWhatId(q.Id);
                    mail.setSaveAsActivity(false);
                    
                    Messaging.sendEmail(new Messaging.SingleEmailMessage[] { mail });
                }
            }
        }
    }
}
```

### Apex Trigger: QuoteEmailTrigger
```apex
trigger QuoteEmailTrigger on Quote (after update) {
    QuoteEmailHandler.sendQuoteEmail(Trigger.new, Trigger.oldMap);
}
```

# Documentación Meta 5 - VALIDACIÓN Y REGLAS DE APROBACIÓN

### Descripción General
Esta sección detalla la implementación del proceso de aprobación y reglas de validación para el Sistema de Cobranza Universitaria en Salesforce, asegurando que todas las cotizaciones sean revisadas por el coordinador de carrera antes de finalizarlas.

### Componentes Implementados

#### 1. Usuario Coordinador
**Tipo:** Usuario de Salesforce
**Configuración:**
- Profile: Chatter External User
- Role: None
- Permisos: Acceso a Quotes y procesos de aprobación

#### 2. Campo de Aprobador
**Nombre:** Coordinador
**Objeto:** Quote
**Configuración:**
- Tipo: Lookup (User)
- Required: ✓
- Descripción: Usuario coordinador que aprobará la cotización

#### 3. Proceso de Aprobación
**Nombre:** Aprobacion_Cotizacion
**Objeto:** Quote
**Configuración:**

**Entry Criteria:**
- EstadoCotizacion__c equals "En Revisión"

**Approval Steps:**
1. Paso único de aprobación:
   - Approver: Coordinador de Carrera
   - Actions:
     * On Submit: Bloquear registro
     * On Approve: Actualizar estado a "Aprobada"
     * On Reject: Actualizar estado a "Rechazada"

**Initial Submission Actions:**
- Field Update: EstadoCotizacion__c = "En Revisión"

**Final Approval Actions:**
- Field Update: EstadoCotizacion__c = "Aprobada"

**Final Rejection Actions:**
- Field Update: EstadoCotizacion__c = "Rechazada"

#### 4. Page Layout Modifications
**Tipo:** Lightning Record Page
**Configuración:**
- Botón "Submit for Approval" agregado al Highlight Panel
- Sección de Approval History visible
- Campos relevantes organizados para fácil acceso

### Flujo del Proceso
1. Usuario edita Quote y cambia estado a "En Revisión"
2. Usuario envía Quote para aprobación
3. Coordinador recibe notificación
4. Coordinador revisa y aprueba/rechaza
5. Sistema actualiza estado automáticamente

---

### Configuración del Proceso de Aprobación
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApprovalProcess xmlns="http://soap.sforce.com/2006/04/metadata">
    <active>true</active>
    <allowRecall>false</allowRecall>
    <name>Aprobacion_Cotizacion</name>
    <recordEditability>AdminOnly</recordEditability>
    <showApprovalHistory>true</showApprovalHistory>
    
    <entryCriteria>
        <formula>ISPICKVAL(EstadoCotizacion__c, 'En Revisión')</formula>
    </entryCriteria>
    
    <finalApprovalActions>
        <action>
            <name>UpdateToApproved</name>
            <type>FieldUpdate</type>
            <field>EstadoCotizacion__c</field>
            <literalValue>Aprobada</literalValue>
        </action>
    </finalApprovalActions>
    
    <finalRejectionActions>
        <action>
            <name>UpdateToRejected</name>
            <type>FieldUpdate</type>
            <field>EstadoCotizacion__c</field>
            <literalValue>Rechazada</literalValue>
        </action>
    </finalRejectionActions>
</ApprovalProcess>
```

