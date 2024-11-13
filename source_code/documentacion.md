# Código Fuente

## Meta 1: Configuración de Objetos y Campos

### Reglas de Validación

1. Límite de Materias por Periodo (Opportunity / Inscripción)
```
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
**Descripción**: Valida que no se excedan los límites de materias según el periodo.
- Periodo Semestral: Máximo 7
- Periodo Cuatrimestral: Máximo 4

**Lógica del código**:
- Usa OR para evaluar cualquiera de las dos condiciones
- Cada AND verifica:
  * Primero el tipo de periodo usando ISPICKVAL
  * Luego compara la cantidad contra el límite
- Si cualquier AND es verdadero, la validación falla

## Meta 2: Implementación de Cálculos de Costos

### Campos de Fórmula en Quote

1. Subtotal Sin Descuentos
```
NumeroMaterias__c * 
CASE(Campus__c,
    'Guanajuato', 17000,
    'Nuevo León', 18000,
    'Jalisco', 13000,
    'Querétaro', 15000,
    0)
```
**Descripción**: Calcula el costo base según campus.

**Lógica del código**:
- CASE determina el precio por materia según el campus
- Multiplica por el número total de materias
- Retorna 0 si el campus no está en la lista
- Operación directa sin descuentos aplicados

2. Descuento por Cantidad de Materias
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
**Descripción**: Calcula descuentos progresivos por cantidad.

**Lógica del código**:
- CASE exterior evalúa número de materias
- Si es 1: no aplica descuento (0)
- Si es 2: calcula 10% sobre una materia
- Si es 3+: calcula 15% por cada materia adicional
- CASE anidado obtiene precio según campus
- Multiplica por factor de descuento correspondiente

[contenido anterior...]

## Meta 3: Configuración de Descuentos y Becas

### Campo de Fórmula para Becas

1. Descuento por Beca
```
CASE(TipoBeca__c,
    'Excelencia', 
    IF(Opportunity.Estudiante__r.Promedio__c >= 9.5,
        SubtotalSinDescuentos__c * 0.10,
        0),
    'Deportiva', 
    SubtotalSinDescuentos__c * 0.10,
    'Familiar Docente', 
    SubtotalSinDescuentos__c * 0.30,
    'Necesidad Económica', 
    SubtotalSinDescuentos__c * 0.30,
    0)
```
**Descripción**: Aplica descuento según tipo de beca.

**Lógica del código**:
- CASE evalúa el tipo de beca seleccionado
- Para beca de excelencia:
  * IF anidado verifica promedio ≥ 9.5
  * Aplica 10% si cumple requisito
- Deportiva: 10% directo
- Familiar/Necesidad: 30% directo
- Default: 0 si no hay beca

2. Validación de Límite de Descuentos
```
(Descuento_por_Cantidad_de_Materias__c + 
 Descuento_por_Pago_de_Contado__c + 
 Descuento_por_Beca__c) / SubtotalSinDescuentos__c > 0.60
```
**Descripción**: Previene exceder 60% en descuentos totales.

**Lógica del código**:
- Suma todos los descuentos aplicados
- Divide entre subtotal para obtener porcentaje
- Compara contra límite máximo (0.60)
- Falla si resultado > 60%

## Meta 4: Automatización del Proceso de Cotización

### Trigger para Envío de Email

1. QuoteEmailHandler
```
public class QuoteEmailHandler {
    public static void sendQuoteEmail(List<Quote> newQuotes, 
                                    Map<Id, Quote> oldQuotes) {
        Id templateId = '00Xak000008qMnp';
        
        List<Quote> quotesWithRelations = [
            SELECT Id, Opportunity.Estudiante__c 
            FROM Quote 
            WHERE Id IN :newQuotes
        ];
        
        for(Quote q : quotesWithRelations) {
            Quote newQuote = (Quote)newQuotes.get(0);
            if(oldQuotes == null || 
               (newQuote.EstadoCotizacion__c == 'Aprobada' && 
                oldQuotes.get(newQuote.Id).EstadoCotizacion__c != 'Aprobada')) {
                
                if(q.Opportunity.Estudiante__c != null) {
                    Messaging.SingleEmailMessage mail = 
                        new Messaging.SingleEmailMessage();
                    mail.setTargetObjectId(q.Opportunity.Estudiante__c);
                    mail.setTemplateId(templateId);
                    mail.setWhatId(q.Id);
                    mail.setSaveAsActivity(false);
                    
                    Messaging.sendEmail(new Messaging.SingleEmailMessage[] 
                        { mail });
                }
            }
        }
    }
}
```
**Descripción**: Envía email automático cuando se aprueba una cotización.

**Lógica del código**:
- Recibe listas de registros nuevos y antiguos
- Consulta relaciones necesarias en una sola query
- Verifica cambio de estado a "Aprobada"
- Configura email con template y destinatario
- Envía email usando Messaging API
- Maneja procesamiento en bulk

2. QuoteEmailTrigger
```
trigger QuoteEmailTrigger on Quote (after update) {
    QuoteEmailHandler.sendQuoteEmail(Trigger.new, Trigger.oldMap);
}
```
**Descripción**: Activa el handler cuando se actualiza una Quote.

**Lógica del código**:
- Se ejecuta después de update
- Pasa nuevos valores y mapa de anteriores
- Delega lógica al handler

## Meta 5: Validación y Reglas de Aprobación

### Proceso de Aprobación

1. Criterios de Entrada
```
EstadoCotizacion__c = 'En Revisión' &&
Descuento_por_Beca__c > 0
```
**Descripción**: Define cuándo una cotización requiere aprobación.

**Lógica del código**:
- Verifica estado específico
- Valida si hay descuentos por beca
- Ambas condiciones deben cumplirse

2. Initial Submission Actions
```
Field Update: EstadoCotizacion__c
Value: 'En Revisión'
Lock Record: True
```
**Descripción**: Acciones automáticas al enviar a aprobación.

**Lógica del código**:
- Actualiza estado automáticamente
- Bloquea registro para edición
- Se ejecuta antes de enviar al aprobador

3. Final Approval Actions
```
Field Update: EstadoCotizacion__c
Value: 'Aprobada'

Email Alert:
Template: Plantilla_Cotizacion
Recipients: Quote.Opportunity.Estudiante__r.Email
```
**Descripción**: Acciones al aprobar cotización.

**Lógica del código**:
- Actualiza estado a aprobado
- Desbloquea registro
- Dispara trigger de email
- Notifica al estudiante

4. Final Rejection Actions
```
Field Update: EstadoCotizacion__c
Value: 'Rechazada'

Email Alert:
Template: Notificacion_Rechazo
Recipients: Quote.Owner.Email, Quote.Opportunity.Estudiante__r.Email
```
**Descripción**: Acciones al rechazar cotización.

**Lógica del código**:
- Cambia estado a rechazado
- Notifica a propietario y estudiante
- Permite edición nuevamente
- Incluye comentarios del rechazo

### Reglas de Reasignación

1. Auto-Assign to Coordinator
```
AssignTo: Quote.Opportunity.Owner.Manager
If_No_Manager: Default_Coordinator_Queue
```
**Descripción**: Asigna automáticamente al coordinador correspondiente.

**Lógica del código**:
- Busca manager del propietario
- Si no existe, usa cola por defecto
- Mantiene jerarquía de aprobación
- Asegura que siempre haya un aprobador
