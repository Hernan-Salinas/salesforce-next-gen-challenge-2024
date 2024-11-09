# Código Fuente
## Meta 1: Configuración de Objetos y Campos

### Reglas de Validación

#### 1. Límite de Materias por Periodo (Opportunity / Inscripción)
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

**Descripción:** Valida que no se excedan los límites de materias según el periodo.
- Periodo Semestral: Máximo 7
- Periodo Cuatrimestral: Máximo 4

**Mensaje de Error:** "El número de materias excede el límite permitido para el periodo seleccionado. Semestral: máximo 7 materias. Cuatrimestral: máximo 4 materias."
